# DOUBAN
---
layout: post
title: "豆瓣python版api开发小记"
tags: [python]
---
由于打算把douban的api接口移植到django，所以采用的是python版sdk

[github地址](https://github.com/douban/douban-client)

这是一份简单的新手指南

###一、安装douban-client

	pip install douban-client
或

	easy_install douban-client

###二、下载源码及例子

地址

	https://github.com/douban/douban-client.git

里面有一些测试代码，可以直接添加到自己的django工程中

###三、运行demo

1.去豆瓣创建一个新应用
应用名称，简介随便填，
应用地址填你希望的地址www.xxxx.com，
回调地址是douban授权通过后返回的地址，可以填你希望的地址www.xxxx.com，
如果你要本地调试的话，可以自己见一个轻量级的http服务器（这里我使用django），
然后在hosts文件里修改

	127.0.0.1 www.xxxx.com
添加测试用户

添加api权限

这个跟后面的报错也有很大关系，测试的话，可以全选上，然后提交审核

2.测试

从上面注册的应用中拿到API key和Secret，以及回调地址然后填到framework.py中
<img src="http://peqiu.com/blog/public/images/posts/douban/doubanapi1.png" >
运行测试可能不通过，报错

	DoubanAPIError: ***400 (Bad Request)*** {u'msg': u'invalid_credencial1', u'code': 108, u'request': u'GET /v2/movie/celebrity/1053585'}
去[豆瓣开发平台网站](http://developers.douban.com/wiki/?title=api_v2)找到自己需要的权限，加到scope里面，如果添加成功，会在授权页出现

<img src="http://peqiu.com/blog/public/images/posts/douban/doubanapi2.png" >
一些权限仅供参考

	douban_basic_r,douban_basic_w 公共API读写
	shuo_basic_r,shuo_basic_w 广播读写
	book_basic_r,book_basic_w 读书读写
	movie_basic_r,movie_basic_w 电影读写
	music_basic_r,music_basic_w 音乐读写
	community_basic_r,community_basic_w 社区读写（经测试无效果）
	event_basic_r,event_basic_w 同城读写

点击运行，会给你一长串地址，在浏览器打开，点击授权，会返回如下一串地址
http://listview.sinaapp.com:8000/depotapp/store/?code=43695fd50fea85db

复制code后面的数字，输入到终端，回车就会开始测试

###四、douban oauth2解释
关于douban oauth2大家可以去网站上看
[地址]( http://developers.douban.com/wiki/?title=oauth2)

基本流程是

#####1.获取authorization_code，

根据client_id，client_secret，redirect_uri，grant_type，code
构造类似如下地址

	https://www.douban.com/service/auth2/auth?
	  client_id=0b5405e19c58e4cc21fc11a4d50aae64&
	  redirect_uri=https://www.example.com/back&
	  response_type=code&
	  scope=shuo_basic_r,shuo_basic_w,douban_basic_common
HTTP GET方式请求

返回结果：
当用户同意授权时，浏览器会重定向到redirect_uri，并附加autorization_code

	https://www.example.com/back?code=9b73a4248

#####2.获取access_token
根据上面获取的code
构造类似如下地址

	https://www.douban.com/service/auth2/token?
	  client_id=0b5405e19c58e4cc21fc11a4d50aae64&
	  client_secret=edfc4e395ef93375&
	  redirect_uri=https://www.example.com/back&
	  grant_type=authorization_code&
	  code=9b73a4248
HTTP POST方式请求

返回结果：

	{
	  "access_token":"a14afef0f66fcffce3e0fcd2e34f6ff4",
	  "expires_in":3920,
	  "refresh_token":"5d633d136b6d56a41829b73a424803ec",
	  "douban_user_id":"1221"
	}

#####3，使用access_token

利用上一步给的access_token，以GET的方式提交。
这里注意一点，提交的时候要把access_token加入Headers提交，而不是加到url里以参数方式提交，貌似新浪支持url，豆瓣不支持。

	Headers.Set("Authorization","Bearer "+ access_token)
这里也要注意一下，Bearer后面是有空格的，不加这个空格的话，授权就不会成功

另外需要注意一下，是https还是http。

#####4,更新access_token

此请求必须是HTTP POST方式，refresh_token只有在access_token过期时才能使用，并且只能使用一次。refresh_token在我们上面获取到的access_token下面。当换取到的access_token再次过期时，使用新的refresh_token来换取access_token

构造

	https://www.douban.com/service/auth2/token?
	  client_id=0b5405e19c58e4cc21fc11a4d50aae64&
	  client_secret=edfc4e395ef93375&
	  redirect_uri=https://www.example.com/back&
	  grant_type=refresh_token&
	  refresh_token=5d633d136b6d56a41829b73a424803ec

###参考资料：

[豆瓣API汇总](http://developers.douban.com/wiki/?title=api_v2)

[使用OAuth2.0访问豆瓣API](http://developers.douban.com/wiki/?title=oauth2)

[总结一下豆瓣OAuth2.0的经验](http://www.douban.com/note/231310967/)
