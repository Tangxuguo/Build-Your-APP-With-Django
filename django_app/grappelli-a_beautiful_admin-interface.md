---
django-grappelli，非常漂亮的admin界面
---
想必你一定觉得django的admin界面丑爆了吧，没关系，我们有很好的app可以完美替代原来的界面，grappelli这样一个非常漂亮的admin管理界面

    **A jazzy skin for the Django admin interface**.

    Grappelli is a grid-based alternative/extension to the `Django <http://www.djangoproject.com>`_ administration interface.

    Code
    ----

    https://github.com/sehmaschine/django-grappelli

    Website
    -------

    http://www.grappelliproject.com

    Documentation
    -------------

    http://readthedocs.org/docs/django-grappelli/

    Releases
    --------

    **Grappelli is always developed against the latest stable Django release and is NOT tested with Djangos trunk.**

    * Grappelli 2.6.2 (October 17th, 2014): Compatible with Django 1.7
    * Grappelli 2.5.5 (October 17th, 2014): Compatible with Django 1.6
    * Grappelli 2.4.11 (October 17th, 2014): Compatible with Django 1.4/1.5

    Current development branches:

    * Grappelli 2.6.3 (Development version for Django 1.7, see branch dev/2.6.x)
    * Grappelli 2.5.6 (Development version for Django 1.6, see branch Stable/2.5.x)
    * Grappelli 2.4.12 (Development version for Django 1.4/1.5, see branch Stable/2.4.x)

    Older versions are availabe at GitHub, but are not supported anymore.
    Support for 2.4.x and 2.5.x is limited to security issues and very important bugfixes.

从这上面的介绍，我们可以知道，对于django 1.4，我们应该选择 Grappelli 2.4.12

#安装
###安装方法一：

把版本切换到2.4，直接从github上clone，下载到本地目录后直接运行

    python setup.py install

<img src="http://peqiu.com/blog/public/images/posts/grappelli/grappelli.png" >
###安装方法二：

使用pip 安装，后面指定安装的版本

    pip install django-grappelli==2.4

#配置
然后，在settings.py里把grappelli添加到INSTALLED_APPS，并且要在django.contrib.admin之前,这样grappelli会接管admin

    INSTALLED_APPS = (
    'grappelli',
    'django.contrib.admin',
)

再然后在你的站点url.py里增加，不是app的url.py

    urlpatterns = patterns('',
    (r'^grappelli/', include('grappelli.urls')), # grappelli URLS
    (r'^admin/',  include(admin.site.urls)), # admin site
)

确保STATICFILES_FINDERS里面AppDirectoriesFinder是第一个

    STATICFILES_FINDERS = (
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
    'django.contrib.staticfiles.finders.FileSystemFinder',
)

在TEMPLATE_CONTEXT_PROCESSORS增加request context processor，这样就可以找到你定义好的模版

	TEMPLATE_CONTEXT_PROCESSORS = (
    ...
    "django.core.context_processors.request",
)

然后收集媒体数据文件

    python manage.py collectstatic

#测试
运行

    python manage.py runserver <IP-address>:8000

测试地址，这样Grappelli会替代你默认的管理界面

    <IP-address>:8000/admin/

几个界面截图

<img src="http://peqiu.com/blog/public/images/posts/grappelli/grappelli1.png" >
<img src="http://peqiu.com/blog/public/images/posts/grappelli/grappelli2.png" >
<img src="http://peqiu.com/blog/public/images/posts/grappelli/grappelli3.png" >
<img src="http://peqiu.com/blog/public/images/posts/grappelli/grappelli4.png" >


#参考网址

https://github.com/sehmaschine/django-grappelli/

http://django-grappelli.readthedocs.org/

http://grappelliproject.com/
