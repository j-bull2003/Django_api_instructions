
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
    return JsonResponse(serializer.data)
```

no we need to define what url this endpoint is going to hit, which is all done in urls.py

create a new path:

```
from colours import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('colours/', views.colours_list)
]









