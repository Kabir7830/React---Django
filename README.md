# React---Django
This is to explain how you can build backend in Django and front-end in React JS and connect them both together

# Creating Back-end in Django

install python and add it to the path variable
install the following dependencies for your backend in Django

    pip install django==4.1
    pip install mysqlclient
    pip install django-mysql
    pip install pillow
    pip install django-restframework
    pip install restframeworksimple_jwt


## After installing the dependencies create your project in a repo

  django-admin startproject project_name

Now move to your project directory and create a new app named backend

    cd project_name
    python manage.py startapp backend

Now open your __settings.py__ file and add 'backend' in the __INSTALLED_APPS__

    INSTALLED_APPS = [
      ...
      'backend',
    ]

Now create your models in the __models.py__ file

__models.py__

    class Product(models.Model):
      product_name = models.CharField(max_length=255)
      regular_price = models.DecimalField(max_digits=10, decimal_places=2)
      sale_price = models.DecimalField(max_digits=10, decimal_places=2,null=True,blank=True)
      description = models.TextField(null=True,blank=True)
      short_description = models.TextField(null=True,blank=True)
      product_image = models.ImageField(upload_to='product_images/',null=True,blank=True)

Now create your serializer_classes for the models you have made
let us create a __serializer.py__ file in backend app

__serializer.py__

    class ProductSerializer(serializers.ModelSerializer):
      class Meta:
          model = Product
          fields = "__all__" # or give the fields you want fields = ['field1','field2','...']

Now create a view for this serializer to accept and give data. Let us create the views in the __views.py__ file

__views.py__

    class ProductAPI(APIView):
    
      def get(self,request):
          
          products = Product.objects.all()
          serializer_class = ProductSerializer(products,many=True)
          return Response({"data":serializer_class.data})
      
      def post(self,request):
          
          serializer_class = ProductSerializer(data=request.data)
          if serializer_class.is_valid():
              serializer_class.save()
              return Response([serializer_class.data])
          return Response([serializer_class.errors])

      def put(self,request,product_id):
        product = Product.objects.filter(id = product_id).first()
        serializer_class = ProductSerializer(instance=product,data=request.data)
        if serializer_class.is_valid():
            serializer_class.save()
            context = {
                "data":serializer_class.data,
                "error":None,
            }
        else:
            context = {
                "data":[],
                "error":serializer_class.errors,
            }
        
        return Response(context)

      def delete(self,request,product_id):
        
        product = Product.objects.filter(id = product_id).first()
        try:
            product.delete()
            return Response({"data":[],"message":"deleted","status":"success"})
        except Exception as e:
            return Response({"data":[],"message":f"couldn't delete! {e}","status":"error"})

Now let us create the URL for the following view in our app's __urls.py__

__urls.py__

    from django.urls import path
    from .views import *
    from django.conf import settings
    from django.conf.urls.static import static
    
    urlpatterns = [
        path('products/',ProductAPI.as_view()),
        path('products/<int:product_id>/',ProductAPI.as_view()),
    ]+ static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)


Let us now configure the __urls.py__ file of our project

__urls.py__

    from django.contrib import admin
    from django.urls import path,include,re_path
    from django.conf import settings
    from django.urls import path, include 
    
    
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('',include('backend.urls')),
        re_path(r'^media/(?P<path>.*)$',serve,{'document_root':settings.MEDIA_ROOT}),
        re_path(r'^static/(?P<path>.*)$',serve,{'document_root':settings.STATIC_ROOT}),
    ]

Now open the terminal and run the following commands

    python manage.py makemigrations
    python manage.py migrate

And we are done with our backend. Now we can view, create, edit, and delete our products using API's
