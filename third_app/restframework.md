# RESTFRAMEWORK
djangorestframework是一款非常简单好用的restframework，通过它你可以很方便地创造出一套API。
这里我们先介绍一下0.4.0的使用方法，再介绍最新版2.0的使用方法

#第一步：安装

###源码安装
从http://pypi.python.org/pypi/djangorestframework/0.4.0 下载，解压后

	python setup.py install
###使用pip安装

	pip install djangorestframework==0.4.0

#第二步：配置
在settings.py的INSTALLED_APPS中加入：

     'djangorestframework',

#第三步：使用

Django REST framework有很多种“用法”，最常见的用法是：

* 1. 定义资源。资源将python对象（比如model对象）进行隔离、组装，生成需要序列化（serialize)的数据。除了基本的Resource类型外，框架还提供了FormResource和ModelResource，以便于对Form或Model的处理。Resource有助于View中的处理，当然你也可以不使用Resource,而在View中去指定要序列化的数据。
* 2. 定义url，将正则表达式匹配的View类的as_view方法，该方法会返回django的view函数。
* 3. 创建视图。视图是对django View的封装，并定义序列化、反系列化等方法，同时通过Mixin的支持来实现get,post,put,delete等操作。框架内置了ModelView，与ModelResource配合使用非常简单方便。


###1. 定义资源
创建chart/resources.py


	from django.core.urlresolvers import reverse
	from djangorestframework.views import View
	from djangorestframework.resources import ModelResource
	from models import *

	class weather_data_Resource( ModelResource ):
	    model = weather_data
	    fields = ( 'id', 'group_number', 'time', 'pm', 'humidity', 'temperature', )

如果其中出现了manytomany或者foreignkey对象，需要重新定义关联的对象。

###2. 定义url
然后使用ModelView定义url：在chart/urls.py的urlpatterns中增加url映射，当然首选要引入相关的模块：



	from django.conf.urls.defaults import *
	from models import *
	from views import *
	from djangorestframework.views import ListOrCreateModelView, InstanceModelView
	from resources import *
	urlpatterns += patterns('',

	    ( r'API/weather_data/items', ListOrCreateModelView.as_view( resource = weather_data_Resource ) ),
	)
此时访问

	http://localhost:8000/chart/API/cart/items/
就可以看到生成的RESTful API了：


如图所示，可以渲染（render）成json,html, xhtml,txt,xml等格式，当然你也可以增加自己的渲染，比如YAML
###3. 创建视图
对于一般的情况来说，这样做就已经足够了。
但是如果你需要扩展一下功能的话（比如权限控制、自定义内容、自定义方式等），还需要进行一些改造。
chart/views.py中自定义一个View类：

	from djangorestframework.views import View

	class RESTfordata( View ):
	    def get( self, request, *args, **kwargs ):
        data = weather_data.objects.order_by( '-id' )[0]
        return data
然后将url改为：

	( r'API/weather_data/items', RESTfordata.as_view( resource = weather_data_Resource ) ),


这时再访问

	http://localhost:8000/chart/API/cart/items/

就可以显示item了。默认的是html 渲染，你可以通过

	http://localhost:8000/chart/API/cart/items/?format=json
访问json渲染:

	[{'id':'0', 'group_number':'1', 'time':'2014-8-9 14:08:08', 'pm':'47', 'humidity':'55', 'temperature':'22.5'},{'id':'1', 'group_number':'1', 'time':'2014-8-9 14:08:08', 'pm':'47', 'humidity':'55', 'temperature':'22.5'}]

用Django REST framework实现RESTful web service，可以说即简单，又灵活。

如果你需要创建数据的话，可以使用post方法，要特别注意post过来的数据格式，一般来说都是form形式，如果是json需要用simplejson处理


	import simplejson

	class RESTfordata( View ):
	    def get( self, request, *args, **kwargs ):

        data = weather_data.objects.order_by( '-id' )[0]
        return data

    def post( self, request, *args, **kwargs ):
        req = simplejson.loads( request.raw_post_data )

        post_group_number = req['records'][0]['group number']
        post_time = datetime.datetime.today().strftime( '%Y-%m-%d %H:%M:%S' )
        post_pm = req['records'][0]['PM2.5']
        post_humidity = req['records'][0]['humidity']
        post_temperature = req['records'][0]['temperature']

        weather_data.objects.create( group_number = post_group_number, time = post_time, pm = post_pm, humidity = post_humidity, temperature = post_temperature )


djangorestframework/0.4.0介绍就到这里了，2.0改进了，稍后介绍
<img src="http://peqiu.com/blog/public/images/posts/restframework/1.png" >
<img src="http://peqiu.com/blog/public/images/posts/restframework/2.png" >

参考资料


http://www.django-rest-framework.org/

http://www.django-rest-framework.org/tutorial/quickstart

http://blog.csdn.net/thinkinside/article/details/7236807
