# django-mongodb-crud

**Django MongoDB CRUD Rest API overview**

We will build Rest Apis using [Django Rest Framework](https://www.django-rest-framework.org/) that can create, retrieve, update, delete and find Tutorials by title or published status.

First, we setup Django Project with a MongoDB Connector. Next, we create Rest Api app, add it with Django Rest Framework to the project. Next, we define data model and migrate it to the database. Then we write API Views and define Routes for handling all CRUD operations (including custom finder).

The following table shows overview of the Rest APIs that will be exported:

| **Methods** | **Urls** | **Actions** |
| --- | --- | --- |
| GET | api/tutorials | get all Tutorials |
| --- | --- | --- |
| GET | api/tutorials/:id | get Tutorial by id |
| POST | api/tutorials | add new Tutorial |
| PUT | api/tutorials/:id | update Tutorial by id |
| DELETE | api/tutorials/:id | remove Tutorial by id |
| DELETE | api/tutorials | remove all Tutorials |
| GET | api/tutorials/published | find all published Tutorials |
| GET | api/tutorials?title=[kw] | find all Tutorials which title contains 'kw' |

Finally, we're gonna test the Rest Apis using Postman.

**Architecture**

This diagram shows the architecture of our Django CRUD Rest Apis App with MongoDB database:

- HTTP requests will be matched by  **Url Patterns**  and passed to the  **Views**
- **Views**  processes the HTTP requests and returns HTTP responses (with the help of  **Serializer** )
- **Serializer**  serializes/deserializes data model objects
- **Models**  contains essential fields and behaviors for CRUD Operations with MongoDB Database

**Technology**

- Python 3.7
- Django 2.1.15
- Django Rest Framework 3.11.0
- djongo 1.3.1
- django-cors-headers 3.2.1
- MongoDB 3.4 or higher

**Project structure**

This is the directory structure of our Django Project:

Let me explain it briefly.

- _tutorials/apps.py_: declares TutorialsConfig class (subclass of django.apps.AppConfig) that represents Rest CRUD Apis app and its configuration.
- _DjangoRestApiMongoDB/settings.py_: contains settings for our Django project: MongoDB Database engine, INSTALLED\_APPS list with Django REST framework, Tutorials Application, CORS and MIDDLEWARE.
- _tutorials/models.py_: defines Tutorial data model class (subclass of django.db.models.Model).
- _migrations/0001\_initial.py_: is created when we make migrations for the data model, and will be used for generating MongoDB database collection.
- _tutorials/serializers.py_: manages serialization and deserialization with TutorialSerializer class (subclass of rest\_framework.serializers.ModelSerializer).
- _tutorials/views.py_: contains functions to process HTTP requests and produce HTTP responses (using TutorialSerializer).
- _tutorials/urls.py_: defines URL patterns along with request functions in the Views.
- _DjangoRestApiMongoDB/urls.py_: also has URL patterns that includes tutorials.urls, it is the root URL configurations.

**Install Django REST framework**

Django REST framework helps us to build RESTful Web Services flexibly.

To install this package, run command:
pip install djangorestframework

**Setup new Django project**

Let's create a new Django project with command:

django-admin startproject DjangoRestApiMongoDB

You can see the following folder tree when the process is done.

Now open _settings.py_ and add Django REST framework to the INSTALLED\_APPS array here.

INSTALLED\_APPS =[

...

# Django REST framework

'rest\_framework',

]

**Connect Django project to MongoDB database**

We need a Django MongoDb connector to work with MongoDb database.
 In this tutorial, we're gonna use [djongo](https://github.com/nesdis/djongo).

Run the command: pip install djongo.

Then we need to setup MongoDb Database engine.
 So open _settings.py_ and change declaration of DATABASES:

DATABASES ={

'default':{

'ENGINE':'djongo',

'NAME':'djangomongo\_db',

'HOST':'127.0.0.1',

'PORT':27017,

}

}

**Setup new Django app for Rest CRUD Api**

Run following commands to create new Django app  **tutorials** :

cd DjangoRestApiMongoDB

python manage.py startapp tutorials

The project directory now looks like:

Open _tutorials/apps.py_, you can see TutorialsConfig class (subclass of django.apps.AppConfig). This is the Django app and its configuration that we've just created.

from django.apps import AppConfig

class TutorialsConfig(AppConfig):

name ='tutorials'

Don't forget to add this app to INSTALLED\_APPS array in _settings.py_:

INSTALLED\_APPS =[

...

# Tutorials application

'tutorials.apps.TutorialsConfig',

]

**Configure CORS and middleware**

We need to allow requests to our Django application from other origins.
 In this example, we're gonna configure CORS to accept requests from localhost:8081.

First, install the _django-cors-headers_ library:
pip install django-cors-headers

In _settings.py_, add configuration for CORS:

INSTALLED\_APPS =[

...

# CORS

'corsheaders',

]

You also need to add a middleware class to listen in on responses:

MIDDLEWARE =[

...

# CORS

'corsheaders.middleware.CorsMiddleware',

'django.middleware.common.CommonMiddleware',

]

_ **Note:** _ CorsMiddleware should be placed as high as possible, especially before any middleware that can generate responses such as CommonMiddleware.

Next, set _CORS\_ORIGIN\_ALLOW\_ALL_ and add the host to _CORS\_ORIGIN\_WHITELIST_:

CORS\_ORIGIN\_ALLOW\_ALL =False

CORS\_ORIGIN\_WHITELIST =(

'http://localhost:8081',

)

- _CORS\_ORIGIN\_ALLOW\_ALL_: If True, all origins will be accepted (not use the whitelist below). Defaults to False.
- _CORS\_ORIGIN\_WHITELIST_: List of origins that are authorized to make cross-site HTTP requests. Defaults to [].

**Define the Django Model**

Open  **tutorials** /_models.py_, add Tutorial class as subclass of django.db.models.Model.
 There are 3 fields: _title_, _description_, _published_.

from django.db import models

class Tutorial(models.Model):

title = models.CharField(max\_length=70, blank=False, default='')

description = models.CharField(max\_length=200,blank=False, default='')

published = models.BooleanField(default=False)

Each field is specified as a class attribute, and each attribute maps to a database column.
_id_ field is added automatically.

**Migrate Data Model to MongoDB database**

Run the Python script: python manage.py makemigrations tutorials.

The console will show:

Migrations for 'tutorials':

tutorials\migrations\0001\_initial.py

- Create model Tutorial

New file _0001\_initial.py_ has just been generated in  **tutorials** / **migrations**  folder. It includes code to create Tutorial data model:

# Generated by Django 2.1.15

from django.db import migrations, models

class Migration(migrations.Migration):

initial =True

dependencies =[

]

operations =[

migrations.CreateModel(

name='Tutorial',

fields=[

('id', models.AutoField(auto\_created=True, primary\_key=True, serialize=False, verbose\_name='ID')),

('title', models.CharField(default='', max\_length=70)),

('description', models.CharField(default='', max\_length=200)),

('published', models.BooleanField(default=False)),

],

),

]

The generated code defines Migration class (subclass of the django.db.migrations.Migration).
 It has operations array that contains operation for creating collection for Customer model: migrations.CreateModel().

The call to this will create a new model in the project history and a corresponding collection in the MongoDB database to match it.

Now run the following Python script to apply the generated migration above:
python manage.py migrate tutorials

The console will show:

Operations to perform:

Apply all migrations: tutorials

Running migrations:

Applying tutorials.0001\_initial... OK

Check MongoDB database, you can see that a collection for Tutorial model was generated automatically with the name: _ **tutorials\_tutorial** _:

**Create Serializer class for Data Model**

Let's create TutorialSerializer class that will manage serialization and deserialization from JSON.

It inherit from rest\_framework.serializers.ModelSerializer superclass which automatically populates a set of fields and default validators. We need to specify the model class here.

**tutorials** /_serializers.py_

from rest\_framework import serializers

from tutorials.models import Tutorial

class TutorialSerializer(serializers.ModelSerializer):

class Meta:

model = Tutorial

fields =('id',

'title',

'description',

'published')

In the inner class Meta, we declare 2 attributes:

- model: the model for Serializer
- fields: a tuple of field names to be included in the serialization

**Define Routes to Views functions**

Let determine how the server will response for HTTP request (GET, POST, PUT, DELETE) with some endpoints. We're gonna define the routes:

- /api/tutorials: GET, POST, DELETE
- /api/tutorials/:id: GET, PUT, DELETE
- /api/tutorials/published: GET

Inside  **tutorials**  app, create _urls.py_ file with urlpatterns containing urls to be matched with the functions in the _views.py_:

from django.conf.urls import url

from tutorials import views

urlpatterns =[

url(r'^api/tutorials$', views.tutorial\_list),

url(r'^api/tutorials/(?P\<pk\>[0-9]+)$', views.tutorial\_detail),

url(r'^api/tutorials/published$', views.tutorial\_list\_published)

]

Don't forget to include this URL patterns in root URL configurations.
 Open  **DjangoRestApiMongoDB** /_urls.py_ and modify the content with the following code:

from django.conf.urls import url, include

urlpatterns =[

url(r'^', include('tutorials.urls')),

]

**Write API Views**

Now we create the functions that are pointed in Url Patterns above:
 – tutorial\_list(): GET list of tutorials, POST a new tutorial, DELETE all tutorials
 – tutorial\_detail(): GET / PUT / DELETE tutorial by 'id'
 – tutorial\_list\_published(): GET all published tutorials

These functions process HTTP requests and make CRUD Operations to database via Django Model.

Open  **tutorials** /_views.py_ and write following code:

from django.shortcuts import render

from django.http.response import JsonResponse

from rest\_framework.parsers import JSONParser

from rest\_framework import status

from tutorials.models import Tutorial

from tutorials.serializers import TutorialSerializer

from rest\_framework.decorators import api\_view

@api\_view(['GET','POST','DELETE'])

deftutorial\_list(request):

# GET list of tutorials, POST a new tutorial, DELETE all tutorials

@api\_view(['GET','PUT','DELETE'])

deftutorial\_detail(request, pk):

# find tutorial by pk (id)

try:

tutorial = Tutorial.objects.get(pk=pk)

except Tutorial.DoesNotExist:

return JsonResponse({'message':'The tutorial does not exist'}, status=status.HTTP\_404\_NOT\_FOUND)

# GET / PUT / DELETE tutorial

@api\_view(['GET'])

deftutorial\_list\_published(request):

# GET all published tutorials

Let's implement these functions.

**Create a new object**

Create and Save a new Tutorial:

@api\_view(['GET','POST','DELETE'])

deftutorial\_list(request):

...

elif request.method =='POST':

tutorial\_data = JSONParser().parse(request)

tutorial\_serializer = TutorialSerializer(data=tutorial\_data)

if tutorial\_serializer.is\_valid():

tutorial\_serializer.save()

return JsonResponse(tutorial\_serializer.data, status=status.HTTP\_201\_CREATED)

return JsonResponse(tutorial\_serializer.errors, status=status.HTTP\_400\_BAD\_REQUEST)

**Retrieve objects (with condition)**

Retrieve all Tutorials/ find by title from MongoDB database:

@api\_view(['GET','POST','DELETE'])

deftutorial\_list(request):

if request.method =='GET':

tutorials = Tutorial.objects.all()

title = request.GET.get('title',None)

if title isnotNone:

tutorials = tutorials.filter(title\_\_icontains=title)

tutorials\_serializer = TutorialSerializer(tutorials, many=True)

return JsonResponse(tutorials\_serializer.data, safe=False)

# 'safe=False' for objects serialization

**Retrieve a single object**

Find a single Tutorial with an id:

@api\_view(['GET','PUT','DELETE'])

deftutorial\_detail(request, pk):

# ... tutorial = Tutorial.objects.get(pk=pk)

if request.method =='GET':

tutorial\_serializer = TutorialSerializer(tutorial)

return JsonResponse(tutorial\_serializer.data)

**Update an object**

Update a Tutorial by the id in the request:

@api\_view(['GET','PUT','DELETE'])

deftutorial\_detail(request, pk):

# ... tutorial = Tutorial.objects.get(pk=pk)

# ...

elif request.method =='PUT':

tutorial\_data = JSONParser().parse(request)

tutorial\_serializer = TutorialSerializer(tutorial, data=tutorial\_data)

if tutorial\_serializer.is\_valid():

tutorial\_serializer.save()

return JsonResponse(tutorial\_serializer.data)

return JsonResponse(tutorial\_serializer.errors, status=status.HTTP\_400\_BAD\_REQUEST)

**Delete an object**

Delete a Tutorial with the specified id:

@api\_view(['GET','PUT','DELETE'])

deftutorial\_detail(request, pk):

# ... tutorial = Tutorial.objects.get(pk=pk)

# ...

elif request.method =='DELETE':

tutorial.delete()

return JsonResponse({'message':'Tutorial was deleted successfully!'}, status=status.HTTP\_204\_NO\_CONTENT)

**Delete all objects**

Delete all Tutorials from the database:

@api\_view(['GET','POST','DELETE'])

deftutorial\_list(request):

# ...

elif request.method =='DELETE':

count = Tutorial.objects.all().delete()

return JsonResponse({'message':'{} Tutorials were deleted successfully!'.format(count[0])}, status=status.HTTP\_204\_NO\_CONTENT)

**Find all objects by condition**

Find all Tutorials with published = True:

@api\_view(['GET'])

deftutorial\_list\_published(request):

tutorials = Tutorial.objects.filter(published=True)

if request.method =='GET':

tutorials\_serializer = TutorialSerializer(tutorials, many=True)

return JsonResponse(tutorials\_serializer.data, safe=False)

**Test the CRUD with APIs**

Run our Django Project with command: python manage.py runserver 8080.
 The console shows:

python manage.py runserver 8080

Performing system checks...

System check identified no issues (0 silenced).

March 30, 2020 - 16:24:01

Django version 2.1.15, using settings 'DjangoRestApiMongoDB.settings'

Starting development server at http://127.0.0.1:8080/

Quit the server with CTRL-BREAK.

Using Postman, we're gonna test all the Apis above.

1. **Create a new Tutorial using **** POST /tutorials **** Api**

Check MongoDB database after creating some Tutorials:

1. **Retrieve all Tutorials using **** GET /tutorials **** Api**

1. **Retrieve a single Tutorial by id using **** GET /tutorials/:id **** Api**

1. **Update a Tutorial using **** PUT /tutorials/:id **** Api**

Check tutorials\_tutorial collection after some rows were updated:

1. **Find all Tutorials which title contains 'ngo': **** GET /tutorials?title=ngo**

1. **Find all published Tutorials using **** GET /tutorials/published **** Api**

1. **Delete a Tutorial using **** DELETE /tutorials/:id **** Api**

Tutorial with id=4 was removed from tutorials\_tutorial collection:

1. **Delete all Tutorials using **** DELETE /tutorials **** Api**

![Shape1](RackMultipart20240205-1-sb16a1_html_ded80440b535355c.gif)
