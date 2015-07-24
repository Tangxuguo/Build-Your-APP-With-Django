# UWSGI
---
layout: post
title: "centos部署django+uwsgi+nginx+sqlite3"
tags: [python]
---

前段时间按照thinkinside的专栏写了个应用，打算把它放到vps上

#环境选择

以前自己用的一直是ubuntu，后来自己的vps转到debian，然后看了[文章](http://www.zhihu.com/question/19599986)打算转到centos。这里选用的是centos7_x86。
http上使用了执行效率更高的nginx+uwsgi。
#安装
###安装编译环境
	yum install python-devel libxml2  libxml2-devel  python-setuptools
	zlib-devel wget openssl-devel pcre pcre-devel sudo
	gcc make autoconf automake
###安装python
这里我使用了python2.7.5，也是系统默认版本。本来想使用virtualenv，但是装好django一直报错，作罢
###安装pip

	easy_install pip
###安装django
这选用django1.4.1毕竟1.4这个版本用得比较多

	wget https://www.djangoproject.com/download/1.4.1/tarball/
	#注意文件名，可能为index.html
	tar -zxvf Django-1.4.1.tar.gz
	cd Django-1.4.1
	python setup.py install
创建项目

	cd /srv/www/
	django-admin.py startproject depot

###安装djangorestframework
由于我要使用restframework，所以装了这个包

	wget https://pypi.python.org/packages/source/d/djangorestframework/djangorestframework-0.4.0.tar.gz
	tar -zxvf djangorestframework-0.4.0.tar.gz
	cd djangorestframework-0.4.0
	python setup.py install
###安装uwsgi

	pip install uwsgi
安装完后在你的机器上写一个test.py

	cd /srv/www/depot
	# test.py
	def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return "Hello World"

然后执行shell命令：

	uwsgi --http :8001 --wsgi-file test.py

访问网页：

	http://127.0.0.1:8001/
看在网页上是否有Hello World

###安装nginx
nginx默认源里没有
创建一个文件

	vim /etc/yum.repos.d/nginx.repo
复制下面

	[nginx]
	name=nginx repo
	baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
	gpgcheck=0
	enabled=1
安装

	yum install nginx

###连接django和uwsgi
假设你的项目地址为/srv/www/depot


	uwsgi --http :8000 --chdir /srv/www/depot --module django_wsgi
这样，你就可以在浏览器中访问你的Django程序了。所有的请求都是经过uwsgi传递给Django程序

###连接nginx和uwsgi
#####配置uwsgi
在实际部署环境中，我们常用的是配置文件的方式，而非命令行的方式。我的一般做法是用命令行来测试是否uWSGI安装成功，然后用配置文件来真正部署。

另外，为了实现Nginx与uWSGI的连接，两者之间将采用soket来通讯方式。
我们将要让Nginx采用8077端口与uWSGI通讯，请确保此端口没有被其它程序采用
新建xml配置文件，将它放在 /srv/www/depot 目录下：

	#django_socket.xml
	<uwsgi>
    <socket>:8077</socket>
    <chdir>/srv/www/depot</chdir>
    <module>django_wsgi</module>
    <processes>2</processes> <!-- 进程数 -->
    <daemonize>uwsgi.log</daemonize>
	</uwsgi>

在上面的配置中，我们使用 uwsgi.log 来记录日志，开启2个进程来处理请求。

这样，我们就配置好uWSGI了。
#####配置nginx

我们假设你将会把Nginx程序日志放到你的目录/srv/www/depot/下，请确保该目录存在。

我们假设你的Django的static目录是/srv/www/depot/static/ ， media目录是/srv/www/depot/public/media/，请确保这些目录存在。

我们假设你的域名是 book.peqiu.com （在调试时你可以设置成你的机器IP）

我们假设你的域名端口是 80（在调试时你可以设置一些特殊端口如 8070）

基于上面的假设，我们为新建/etc/nginx/conf.d/depot.conf添加以下配置

	server {

        listen   80;
        server_name book.peqiu.com;
        access_log /srv/www/depot/access.log;
        error_log /srv/www/depot/error.log;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
         include        uwsgi_params;
         uwsgi_pass     127.0.0.1:8077;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        location /static/ {
            alias  /srv/www/depot/static;
            index  index.html index.htm;
        }

        location /media/ {
            alias  /srv/www/depot/public/media/;
        }
    }
如果需要做多网站配置，再新建如/etc/nginx/conf.d/list.conf添加以下配置，修改server_name，及文件路径

	server {

        listen   80;
        server_name list.peqiu.com;
        access_log /srv/www/list/access.log;
        error_log /srv/www/list/error.log;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
         include        uwsgi_params;
         uwsgi_pass     127.0.0.1:8077;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        location /static/ {
            alias  /srv/www/list/static;
            index  index.html index.htm;
        }

        location /media/ {
            alias  /srv/www/list/public/media/;
        }
    }
###启动服务

1.重启Nginx服务器，以使Nginx的配置生效。

	nginx -s  reload
重启后检查Nginx日志是否有异常。

2.启动uWSGI服务器


	cd /srv/www/depot
	uwsgi -x django_socket.xml

3.访问服务

基于上面的假设你的域名是book.peqiu.com

因此，我们访问 book.peqiu.com，如果发现程序与 单独使用Django启动的程序一模一样时，就说明成功啦！

4.关闭服务的方法

将uWSGi进程杀死即可。

整个系统运行内存大概在100M左右
#参考资料：
大部分来自django-china.cn，感谢
http://django-china.cn/topic/124/
