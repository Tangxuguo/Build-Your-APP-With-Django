---
layout: post
title: "使用django-celery实现异步执行"
tags: [django]
---
我们通常可能会碰到一些如发送邮件等后台处理时间比较长的任务，这个时候我们不能等待任务处理完成再返回，而应该马上返回，让任务后台处理。

django-celery就这这样一种实现后台异步执行模块

网上常用的组合：

 django：web框架；

 RabbitMQ：消息队列系统，负责存储消息；

 celery：worker进程，同时提供在webapp中创建任务的功能。

在这里我们并不打算使用RabbitMQ，而是采用DJANGO自带的消息处理

网上写的比较好的教程有:

1. [Django中如何使用django-celery完成异步任务](http://www.weiguda.com/blog/73/)
2. [DJANGO中异步任务CELERY](http://quxl.snbway.net/celery/)
3. [django-celery](http://www.cnblogs.com/tuifeideyouran/p/4191511.html)
4. [ [Celery]Celery 最佳实践](http://blog.csdn.net/orangleliu/article/details/37967433)

但是发现网上很多的教程版本比较低，新的版本发生了一些变化，可以看看，在这里记录一下

国外写的比较好的教程：

1. [FIRST STEPS WITH CELERY: HOW TO NOT TRIP](http://hairycode.org/2013/07/23/first-steps-with-celery-how-to-not-trip/)


可以参考，其他的还是看官方文档吧

1. [First steps with Django](http://docs.celeryproject.org/en/latest/django/first-steps-with-django.html#using-celery-with-django)
2. [Configuration and defaults](http://docs.celeryproject.org/en/latest/configuration.html#std:setting-CELERY_IMPORTS)

有点出入的一个是配置文件，配置文件的路径要注意，根目录是项目所在目录。另外需要首先实例化一个celery，除此之外一定要启动celery，后台如果发现没有调用worker就要查下配置文件了，这里官方推荐的是采用配置文件的形式而不是都写到setting文件里，另外需要注意环境变量，在celery启动的时候需要先加载环境变量

	os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'reservation.settings')

启动celery命令,中间的'celerytask.tasks'根据自己的文件自己改

	$ celery -A celerytask.tasks worker -l info

查看celery进程

	$ ps aux | grep celery

<img src="/blog/public/images/posts/code/celery.png" >

tasks.py

	# encoding:utf-8
	from celery import Celery
	import os
	import time
	from django.conf import settings

	os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'reservation.settings')
	app = Celery('test')
	# Using a string here means the worker will not have to
	# pickle the object when using Windows.
	app.config_from_object('celerytask.celeryconfig')
	app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)

	@app.task()
	def add_task(name):
	    for i in range(1,10):
	        print 'hello:%s %s'%(name,i)
	        time.sleep(1)
	    return 1
	if __name__ == '__main__':
	    app.start()


celeryconfig.py,这里配置有点绕，看官方文档

	BROKER_URL = 'django://'
	#CELERY_RESULT_BACKEND = 'amqp://'

	CELERY_TASK_SERIALIZER = 'json'
	CELERY_RESULT_SERIALIZER = 'json'
	CELERY_ACCEPT_CONTENT=['json']
	CELERY_TIMEZONE = 'Europe/Oslo'
	CELERY_ENABLE_UTC = True

	# CELERY_ROUTES = {
	#     'celerytask.tasks.add_task': 'low-priority',
	# }

	CELERY_ANNOTATIONS = {
	    'celerytask.tasks.add_task': {'rate_limit': '2/m'}
	}

admin.py注册一下管理接口

	from django.contrib import admin

	# Register your models here.
	from django.contrib import admin
	from kombu.transport.django import models as kombu_models

	admin.site.register(kombu_models.Message)

view.py 写了一个简单的视图函数

	from celerytask.tasks import add_task

	# encoding:utf-8
	from django.http import HttpResponse
	from django.views.generic import View


	class Hello(View):

	    def get(self, request, *args, **kwargs):
	        add_task.delay('GreenPine')
	        return HttpResponse('Hello, World!')

主站urls.py里面加一个这个视图就好了

	from celerytask.views import Hello
	urlpatterns += patterns('',url(r'^celery/',Hello.as_view()),)

访问的时候如果终端里面的worker有接受任务，并且浏览器很快返回，管理界面有消息记录就成功了，如果没有对照官方文档检查配置
