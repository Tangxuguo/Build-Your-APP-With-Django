# django_weixin_portal
###[django\_weixin\_portal](https://github.com/wwj718/django_weixin_portal)
django\_weixin\_portal功能非常强大，具体你可以去官网上看，他采用了亿米CH开源的第三方微信开发者账号管理平台，在上面做了一些应用，按照它的安装说明是能跑通的，我的是在digitalocean上跑的centos7x64。github有很教程

#####安装过程

基本环境

	yum install python-devel libxml2  libxml2-devel  python-setuptools
	zlib-devel wget openssl-devel pcre pcre-devel sudo
	gcc make autoconf automake vim git
安装

	git clone https://github.com/wwj718/django_weixin_portal.git
	cd django_weixin_portal
	pip install -r requlirements.txt
后台帐号:admin . 密码：admin

地址：

 * http://youhost:port/admin/
 * http://youhost:port/yimi-admin/

如果装了其他版本的换回来

	pip install django==1.6

这里需要注意的是：

 * 它要求django版本是1.6，我在1.5和1.7版本上跑是有问题的
 * 这个开源框架要求你必须有app_id,也就是说你必须是服务号，或者早期的订阅号
 * 这个框架有两个后台，一个网站的，一个亿米的后台

贴几张图

网站后台
<img src="http://peqiu.com/blog/public/images/posts/weixin/weixin1.png" >

添加app_id
<img src="http://peqiu.com/blog/public/images/posts/weixin/weixin2.png" >

亿米后台
<img src="http://peqiu.com/blog/public/images/posts/weixin/weixin3.png" >

管理添加文章
<img src="http://peqiu.com/blog/public/images/posts/weixin/weixin4.png" >
由于我没有服务号，作罢
