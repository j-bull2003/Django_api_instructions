
make a directory (folder)

`mkdir colours`

go into colours directory

`cd colours`

check python is installed

`python3 --version`
or
`python --version`

create virtual environment

`python -m venv .venv`
or 
`python3 -m venv .venv`

activate it

`. .venv/bin/activate`

to deactivate

`deactivate`

install the requirements

`pip install django`
or 
`pip3 install django`

`pip install djangorestframework`
or 
`pip3 install djangorestframework`

see the commands possible to run on django

`django-admin`

to start the project (this will create a new folder inside the project folder with a manage.py file just outside of it)

`django-admin startproject colours .`

makemigrations packages up your model changes into migration files

`python manage.py makemigrations`
or 
`python3 manage.py makemigrations`

migrate applies those changes to your db

`python manage.py migrate`
or 
`python3 manage.py migrate`

start running basic app

`python manage.py runserver`
or
`python3 manage.py runserver`

create a superuser to access the admin page

`python manage.py createsuperuser`
or
`python3 manage.py createsuperuser`

add app to the settings.py:

```
INSTALLED_APPS = [
    'colours',
    '...',
    '...',
    '...',
]
```

create a model:
inside of the colours folder add another file called "models.py" (same directory as settings.py)

To tell django that this is a model we need to inherit from the model class:

``` 
from django.db import models

class Colour(models.Model):
    name = models.Charfield(max_length=200)
    description = models.CharField(max_length=500)

    # to make the name look better on django admin site(instead of object 1, object 2)
    def __str__(self):
    return self.name + ' ' + self.description
```

add model to admin page:
in our colours folder add a new file called "admin.py":
This is where we register the different models that we want to show in the admin panel

```
from django.contrib import admin
from.models import Colour

admin.site.register(Colour)
```

Then you can go onto the admin gjango site and add data to your model

#### we created a model, a few instances and stored thm in the db, now we are going to work on getting this data through the api using djangorestframework

add djangorestframework to the installed apps list in settings.py

```
INSTALLED_APPS = [
    'colours',
    'rest_framework',
    '...',
    '...',
    '...',
]
```

Inside of the colours folder create another new file called "serializers.py" - to convert complex data types, such as Django model instances, into Python data types that can be easily rendered into JSON

serializers.py

```
from rest_framework import serializers
from .models import Colour

class ColourSerializer(serializer.ModelSerializer):
    class Meta:
            model = Colour
            fields =['id', 'name', 'description']
```

now that we have our model, and we have our serializer, now we need to create an endpoint
endpoint = a certain url that you can access data from
create a new file for our endpoint in the colours folder called views.py

views.py:
```
from django.http import JsonResponse
from .models import Colour
from .serializers import ColourSerializer

def colour_list(request):
    # 1. get all the colours
    # 2. serialize them
    # 3. return json
    colours = Colour.Objects.all() # 1
    serializer = ColourSerializer(colours, many=True) # 2 (many is set to true because we need to serialize multiple colours)
    return JsonResponse('colours': serializer.data)
```
above is a function that will take a GET request

now we need to define what url this endpoint is going to hit, which is all done in urls.py

create a new path:

```
from colours import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('colours/', views.colours_list)
]
```

now we have created a simple api - where you can get a list of drinks


```
from django.http import JsonResponse
from .models import Colour
from .serializers import ColourSerializer

def colour_list(request):
    # 1. get all the colours
    # 2. serialize them
    # 3. return json

    # 1
    colours = Colour.Objects.all() 
    # 2 (many is set to true because we need to serialize multiple colours)
    serializer = ColourSerializer(colours, many=True) 
    # 3
    return JsonResponse('colours': serializer.data)
```


change the views.py file from that to this. 
making this change ensures that you can make multiple different requests (GET and POST)to this endpoint (we do this by adding a decorator)
GET - read data from the db
POST - create (add) data to the db
for reference:
PUT - update (replace)



```
from django.http import JsonResponse
from .models import Colour
from .serializers import ColourSerializer
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

# add a decorator
# pass in arguments as list
@api_view(['GET', 'POST'])
def colour_list(request):

    if request.method == 'GET':
        colours = Colour.Objects.all() 
        serializer = ColourSerializer(colours, many=True) 
        return JsonResponse('colours': serializer.data)
    
    if request.method == 'POST':

        # actually get the data to serialize
        serializer = ColourSerializer(data=request.data)

        # check that the data is valid
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
```

To check adding data to the database, go to postman, and make a POST request to the http://localhost:3000/colours endpoint (or whatever it is).


add a new path to get the details of a sepcific colour ID

```
from colours import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('colours/', views.colours_list),
    path('drinks/<int:id>', views.drink_detail)
]
```

since you've added a new path, you need to create another api endpoint in the views.py to return the details of a speicifc colour ID.
So in your views.py:

```
from django.http import JsonResponse
from .models import Colour
from .serializers import ColourSerializer
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

# add a decorator
# pass in arguments as list
@api_view(['GET', 'POST'])
def colour_list(request):

    if request.method == 'GET':
        colours = Colour.Objects.all() 
        serializer = ColourSerializer(colours, many=True) 

        # use JsonResponse to return data 
        return JsonResponse('colours': serializer.data)
    
    if request.method == 'POST':

        # actually get the data to serialize
        serializer = ColourSerializer(data=request.data)

        # check that the data is valid
        if serializer.is_valid():
            serializer.save()

            # use "Response" to add data 
            return Response(serializer.data, status=status.HTTP_201_CREATED)

# PUT is to update data (change existing data) 
@api_view(['GET', 'PUT', 'DELETE'])
def drink_detail(request, id):

    #checking to see if the id of the colour even exists in the first place
    try:
        colour = Colour.objects.get(pk=id)
    except Drink.DoesNorExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method =='GET':
        serializer = DrinkSerializer(colour)
        return Response(serialize.data)

    elif request.method == 'PUT':
        # you need to pass in the data (so then it knows which data it will need to update)
        serializer = ColourSerializer(colour, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=stauts.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        colour.delete
        return Response(status.status.HTTP_204_NO_CONTENT)
    
```
 

