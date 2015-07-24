---
layout: post
title: "基于py-oauth2的jaccount接口开发"
tags: [Django]
---

# oauth2介绍
关于oauth2及认证流程的介绍，可以看我前一篇博文[豆瓣python版api开发小记](http://peqiu.com/blog/douban-python-api.html)，这里我直接拿来用了。

# py-oauth2模块
py-oauth2是用python写的oauth2客户端，作者提供了丰富的应用实例，包括但不仅限于google、douban、github、weibo、QQ、taobao等等。github地址[https://github.com/liluo/py-oauth2](https://github.com/liluo/py-oauth2)。
###安装

	pip install py-oauth2

#django移植oauth2接口

django移植，可以参考项目[https://github.com/simplegeo/python-oauth2](https://github.com/simplegeo/python-oauth2)，为什么我不用这个呢？因为这个是oauth1的，但是流程可以借鉴，这个已经写的很详细了。

#jaccount接入

关于jaccount开发可以参考[官方文档](http://developer.sjtu.edu.cn/wiki/Main_Page)

接口地址：

	http://developer.sjtu.edu.cn/wiki/JAccount
API：

	http://developer.sjtu.edu.cn/wiki/APIs

官方提供了很多版本的，但是没有python。另外需要注意的是我们申请的是oauth2的接口，授权还是可以的，但是api不行。
#详细开发流程

###申请API
###使用py-oauth2s授权
jaccount的OAuth2 地址

	baseURL: https://jaccount.sjtu.edu.cn/oauth2/
	authorizationURL: https://jaccount.sjtu.edu.cn/oauth2/authorize
	tokenURL: https://jaccount.sjtu.edu.cn/oauth2/token

使用liluo提供的douban认证demo

基本认证流程



+ 用户点击jaccounts/login/链接，调用jaccount_login_view函数，构造授权客户端，转到授权页。
+ 用户授权完后，返回到我们提供的url里jaccounts/login/authenticated/，调用jaccount_authenticated_view函数提取code，和服务器换取token，
+ 然后去构造url去请求数据

urls.py添加

		urlpatterns = patterns( '',

                          url( r'^jaccounts/login/?$', jaccount_login_view ),
                          url( r'^jaccounts/login/authenticated/?$', jaccount_authenticated_view ),
							）

views.py中添加

	from pyoauth2 import Client

	KEY = '填自己的'
	SECRET = '填自己的'
	#认证后返回的地址
	CALLBACK = 'http://127.0.0.1:8090/jaccounts/login/authenticated/'
	SCOPE = ['basic', 'essential', 'profile', ]
	SCOPE = ' '.join( SCOPE )

	def jaccount_login_view( request ):

	    client = Client( KEY, SECRET,
	                site = 'https://jaccount.sjtu.edu.cn/oauth2/',
	                authorize_url = 'https://jaccount.sjtu.edu.cn/oauth2/authorize',
	                token_url = 'https://jaccount.sjtu.edu.cn/oauth2/token' )
	    authorize_url = client.auth_code.authorize_url( redirect_uri = REDIRECT_URL,scope = SCOPE )
		#转到授权页
	    return HttpResponseRedirect( authorize_url )
		#

	def jaccount_authenticated_view( request ):
		#获取code
	    code = request.GET.get( 'code' )
	    client = Client( KEY, SECRET,
	                site = 'https://jaccount.sjtu.edu.cn/oauth2/',
	                authorize_url = 'https://jaccount.sjtu.edu.cn/oauth2/authorize',
	                token_url = 'https://jaccount.sjtu.edu.cn/oauth2/token' )
		#用code换取token
	    access_token = client.auth_code.get_token( code, redirect_uri = REDIRECT_URL )
		#提取token
	    token_tmp = access_token.headers['Authorization'].split( ' ' )[1]
	    #用token去请求数据
		ret = access_token.get( 'https://api.sjtu.edu.cn/v1/me/profile?access_token=' + token_tmp, )
		#解析返回数据
	    result = ret.parsed
	    print ret.parsed




