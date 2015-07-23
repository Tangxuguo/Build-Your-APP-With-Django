# weixin2py
###[weixin2py](https://github.com/winkidney/weixin2py)
weixin2py是另一个非常强大的框架，最近也进行了一次更新，他的插件机制可以很方便进行二次开发，这是我打算使用的原因，另外框架写好了你在开发过程中所需要的库和函数，非常方便，如果你不需要太多功能的话，匹配回复事件处理直接可以拿来用，github有很详细的教程，就不罗嗦了


#####安装过程

基本环境

	yum install python-devel libxml2  libxml2-devel  python-setuptools
	zlib-devel wget openssl-devel pcre pcre-devel sudo
	gcc make autoconf automake vim git
安装
	git clone https://github.com/winkidney/weixin2py.git
	pip install django==1.5
	cd weixin2py
	python setup.py install
	cd weixin2py
	python rebuild_db.py
如果报错django.setup(),注释掉

然后在你的settings.py中编辑TOKEN，改为你自己的TOKEN

	sh runserver.sh

微信公众平台的接口URL为

	http://yourhost:port/weichat/
后台帐号:admin . 密码：admin 地址：http://youhost:port/admin/
网站后台
<img src="http://peqiu.com/blog/public/images/posts/weixin/weixin5.png" >

管理添加消息
<img src="http://peqiu.com/blog/public/images/posts/weixin/weixin6.png" >

后台事件处理
<img src="http://peqiu.com/blog/public/images/posts/weixin/weixin7.png" >

后台文本处理，支持正则匹配，插件
<img src="http://peqiu.com/blog/public/images/posts/weixin/weixin8.png" >
