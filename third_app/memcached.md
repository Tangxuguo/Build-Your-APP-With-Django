---
layout: post
title: "在Django中使用memcached"
tags: [Django]
---

mac 下安装memcached [教程](http://www.blogways.net/blog/2013/05/01/demo-libmemcached-at-mac.html)

安装服务器

	brew install memcached
安装客户端

	brew install libmemcached
启动服务器

	memcached -d

在virtualenv下安装python binding

建议用[pylibmc](http://sendapatch.se/projects/pylibmc/)，不用[python-memcached](ftp://ftp.tummy.com/pub/python-memcached/),方便移到SAE

	pip install pylibmc

其他配置使用参考官方文档

主要配置：

在settings.py

	# memcahce
	CACHES = {
	    'default': {
	        'BACKEND': 'django.core.cache.backends.memcached.PyLibMCCache',
	        'LOCATION': '127.0.0.1:11211',
	    }
	}

使用详情查看官方文档

[Django’s cache framework](https://docs.djangoproject.com/en/1.8/topics/cache/)

支持站点缓存，页面缓存，模板缓存，分布式缓存，指定服务器缓存等
