# django rest framework 3.0

前面说了0.4版本，现在已经3.1.2，更新下使用

[安装](http://www.django-rest-framework.org/)

安装软件

    pip install djangorestframework

加到setting设置里

    INSTALLED_APPS = (
    ...
    'rest_framework',
)

在路由urls.py,添加认证用的路由

    urlpatterns = [
    ...
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
    ]

使用教程

官方给的使用example，在urls.py添加如下内容

    from django.conf.urls import url, include
    from django.contrib.auth.models import User
    from rest_framework import routers, serializers, viewsets

    # Serializers define the API representation.
    class UserSerializer(serializers.HyperlinkedModelSerializer):
        class Meta:
            model = User
            fields = ('url', 'username', 'email', 'is_staff')

    # ViewSets define the view behavior.
    class UserViewSet(viewsets.ModelViewSet):
        queryset = User.objects.all()
        serializer_class = UserSerializer

    # Routers provide an easy way of automatically determining the URL conf.
    router = routers.DefaultRouter()
    router.register(r'users', UserViewSet)

    # Wire up our API using automatic URL routing.
    # Additionally, we include login URLs for the browsable API.
    urlpatterns = [
        url(r'^', include(router.urls)),
        url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
    ]

其实你也可以把上面的内容拆成3部分

serializers.py

    from rest_framework import serializers
    from django.contrib.auth.models import User

    # Serializers define the API representation.
    class UserSerializer(serializers.HyperlinkedModelSerializer):
        class Meta:
            model = User
            fields = ('url', 'username', 'email', 'is_staff')

views.py

    from django.contrib.auth.models import User
    from rest_framework import  viewsets
    from serializers import UserSerializer

    # ViewSets define the view behavior.
    class UserViewSet(viewsets.ModelViewSet):
        queryset = User.objects.all()
        serializer_class = UserSerializer

urls.py

    from django.contrib.auth.models import User
    from rest_framework import routers
    from views import UserViewSet

    # Routers provide an easy way of automatically determining the URL conf.
    router = routers.DefaultRouter()
    router.register(r'users', UserViewSet)

    # Wire up our API using automatic URL routing.
    # Additionally, we include login URLs for the browsable API.
    urlpatterns = [
        url(r'^', include(router.urls)),
        url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
    ]


说下详细使用:

目前官方的6部分教程都有中文翻译,不过不是最新的，要看最新的去看英文的

[django rest framework 入门1－序列化 Serialization](http://blog.csdn.net/zhaoyingm/article/details/8531617)

[django rest framework 入门2——Request and Response](http://blog.csdn.net/zhaoyingm/article/details/8532808)

[Django REST Framework Tutorial 3：基于类的Views（中文版教程）by hillfree](http://www.cnblogs.com/hillfree/archive/2013/03/25/django-rest-framework-tutorial-03.html)

[Django REST Framework Tutorial 4：认证与权限（中文版教程）by hillfree](http://www.cnblogs.com/hillfree/archive/2013/03/26/django-rest-framework-tutorial-04.html)

[Django REST Framework Tutorial 5：关系与超链接API（中文版教程）by hillfree](http://www.cnblogs.com/hillfree/archive/2013/03/29/django-rest-framework-tutorial-05.html)

[教程 6: ViewSets & Routers](http://blog.csdn.net/zouyee/article/details/16994815)

1.序列化

序列号有好几种方式
你可以自己定制序列化，使用serializers.Serializer

    from rest_framework.renderers import JSONRenderer
    from rest_framework.parsers import JSONParser

    class SnippetSerializer(serializers.Serializer):
        pk = serializers.IntegerField(read_only=True)
        def create(self, validated_data):
        """
        Create and return a new `Snippet` instance, given the validated data.
        """
        return Snippet.objects.create(**validated_data)

    serializer = SnippetSerializer(snippet)
    content = JSONRenderer().render(serializer.data)
反序列化
    from django.utils.six import BytesIO

    stream = BytesIO(content)
    data = JSONParser().parse(stream)
也可以使用ModelSerializers

    class SnippetSerializer(serializers.ModelSerializer):
        class Meta:
            model = Snippet
            fields = ('id', 'title', 'code', 'linenos', 'language', 'style')

2.请求响应

@api_view可以限制访问类型

    @api_view(['GET', 'PUT', 'DELETE'])
    def snippet_detail(request, pk):
        """
        Retrieve, update or delete a snippet instance.
        """
        try:
            snippet = Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            return Response(status=status.HTTP_404_NOT_FOUND)

        if request.method == 'GET':
            serializer = SnippetSerializer(snippet)
            return Response(serializer.data)

        elif request.method == 'PUT':
            serializer = SnippetSerializer(snippet, data=request.data)
            if serializer.is_valid():
                serializer.save()
                return Response(serializer.data)
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

        elif request.method == 'DELETE':
            snippet.delete()
            return Response(status=status.HTTP_204_NO_CONTENT)

也可以限制输出格式

    from rest_framework.urlpatterns import format_suffix_patterns
    from blog import views

    urlpatterns = [
        url(r'^/$', views.apt_root),
        url(r'^comments/$', views.comment_list),
        url(r'^comments/(?P<pk>[0-9]+)/$', views.comment_detail)
    ]

    urlpatterns = format_suffix_patterns(urlpatterns, allowed=['json', 'html'])

    使用i18n

    urlpatterns = i18n_patterns(
    format_suffix_patterns(urlpatterns, allowed=['json', 'html'])
)

3.类视图

重点来了，类视图可以节省大量代码




    from snippets.models import Snippet
    from snippets.serializers import SnippetSerializer
    from django.http import Http404
    from rest_framework.views import APIView
    from rest_framework.response import Response
    from rest_framework import status


    class SnippetList(APIView):
        """
        List all snippets, or create a new snippet.
        """
        def get(self, request, format=None):
            snippets = Snippet.objects.all()
            serializer = SnippetSerializer(snippets, many=True)
            return Response(serializer.data)

        def post(self, request, format=None):
            serializer = SnippetSerializer(data=request.data)
            if serializer.is_valid():
                serializer.save()
                return Response(serializer.data, status=status.HTTP_201_CREATED)
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    class SnippetDetail(APIView):
        """
        Retrieve, update or delete a snippet instance.
        """
        def get_object(self, pk):
            try:
                return Snippet.objects.get(pk=pk)
            except Snippet.DoesNotExist:
                raise Http404

        def get(self, request, pk, format=None):
            snippet = self.get_object(pk)
            serializer = SnippetSerializer(snippet)
            return Response(serializer.data)

        def put(self, request, pk, format=None):
            snippet = self.get_object(pk)
            serializer = SnippetSerializer(snippet, data=request.data)
            if serializer.is_valid():
                serializer.save()
                return Response(serializer.data)
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

        def delete(self, request, pk, format=None):
            snippet = self.get_object(pk)
            snippet.delete()
            return Response(status=status.HTTP_204_NO_CONTENT)

使用类视图，需要使用class的as_view方法，

    from django.conf.urls import url
    from rest_framework.urlpatterns import format_suffix_patterns
    from snippets import views

    urlpatterns = [
        url(r'^snippets/$', views.SnippetList.as_view()),
        url(r'^snippets/(?P<pk>[0-9]+)/$', views.SnippetDetail.as_view()),
    ]

    urlpatterns = format_suffix_patterns(urlpatterns)

使用mixin类，可以继承多个mixin类，

    from snippets.models import Snippet
    from snippets.serializers import SnippetSerializer
    from rest_framework import mixins
    from rest_framework import generics

    class SnippetList(mixins.ListModelMixin,
                      mixins.CreateModelMixin,
                      generics.GenericAPIView):
        queryset = Snippet.objects.all()
        serializer_class = SnippetSerializer

        def get(self, request, *args, **kwargs):
            return self.list(request, *args, **kwargs)

        def post(self, request, *args, **kwargs):
            return self.create(request, *args, **kwargs)

上面的定义提供了.list() and .create()方法

    class SnippetDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
        queryset = Snippet.objects.all()
        serializer_class = SnippetSerializer

        def get(self, request, *args, **kwargs):
            return self.retrieve(request, *args, **kwargs)

        def put(self, request, *args, **kwargs):
            return self.update(request, *args, **kwargs)

        def delete(self, request, *args, **kwargs):
            return self.destroy(request, *args, **kwargs)

上面的定义提供了.retrieve(), .update() and .destroy() 方法

再进一步，使用通用视图

    from snippets.models import Snippet
    from snippets.serializers import SnippetSerializer
    from rest_framework import generics


    class SnippetList(generics.ListCreateAPIView):
        queryset = Snippet.objects.all()
        serializer_class = SnippetSerializer


    class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
        queryset = Snippet.objects.all()
        serializer_class = SnippetSerializer

4.认证

5.Relationships & Hyperlinked APIs

6.ViewSets & Routers
