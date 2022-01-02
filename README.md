
### Django REST Framework[](https://realpython.com/api-integration-in-python/#django-rest-framework "Permanent link")

Another popular option for building REST APIs is  [Django REST framework](https://realpython.com/django-rest-framework-quick-start/). Django REST framework is a  [Django](https://realpython.com/get-started-with-django-1/)  plugin that adds REST API functionality on top of an existing Django project.

To use Django REST framework, you need a Django project to work with. If you already have one, then you can apply the patterns in the section to your project. Otherwise, follow along and you’ll build a Django project and add in Django REST framework.

First, install  `Django`  and  `djangorestframework`  with  `pip`:

`$ python -m pip install Django djangorestframework` 

This installs  `Django`  and  `djangorestframework`. You can now use the  `django-admin`  tool to create a new Django project. Run the following command to start your project:

`$ django-admin startproject countryapi` 

This command creates a new folder in your current directory called  `countryapi`. Inside this folder are all the files you need to run your Django project. Next, you’re going to create a new  **Django application**  inside your project. Django breaks up the functionality of a project into applications. Each application manages a distinct part of the project.

**Note:**  You’re only going to scratch the surface of what Django can do in this tutorial. If you’re interested in learning more, then check out the available  [Django tutorials](https://realpython.com/tutorials/django/).

To create the application, change directories to  `countryapi`  and run the following command:

`$ python manage.py startapp countries` 

This creates a new  `countries`  folder inside your project. Inside this folder are the base files for this application.

Now that you’ve created an application to work with, you need to tell Django about it. Alongside the  `countries`  folder that you just created is another folder called  `countryapi`. This folder contains configurations and settings for your project.

**Note:**  This folder has the same name as the root folder that Django created when you ran  `django-admin startproject countryapi`.

Open up the  `settings.py`  file that’s inside the  `countryapi`  folder. Add the following lines to  `INSTALLED_APPS`  to tell Django about the  `countries`  application and Django REST framework:

```
# countryapi/settings.py
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
	"rest_framework", 
	"countries", 
 ] 
```
You’ve added a line for the  `countries`  application and  `rest_framework`.

You may be wondering why you need to add  `rest_framework`  to the applications list. You need to add it because Django REST framework is just another Django application. Django plugins are Django applications that are packaged up and distributed and that anyone can use.

The next step is to create a Django model to define the fields of your data. Inside of the  `countries`  application, update  `models.py`  with the following code:

```
# countries/models.py
from django.db import models

class Country(models.Model):
 name = models.CharField(max_length=100) 
 capital = models.CharField(max_length=100) 
 area = models.IntegerField(help_text="(in square kilometers)")
 ```

This code defines a  `Country`  model. Django will use this model to create the database table and columns for the country data.

Run the following commands to have Django update the database based on this model:

```
$ python manage.py makemigrations
Migrations for 'countries':
 countries/migrations/0001_initial.py
 - Create model Country

$ python manage.py migrate
Operations to perform:
 Apply all migrations: admin, auth, contenttypes, countries, sessions
Running migrations:
 Applying contenttypes.0001_initial... OK
 Applying auth.0001_initial... OK
 ...` 
 ```

These commands use  [Django migrations](https://realpython.com/django-migrations-a-primer/)  to create a new table in the database.

This table starts empty, but it would be nice to have some initial data so you can test Django REST framework. To do this, you’re going to use a  [Django fixture](https://realpython.com/django-pytest-fixtures/)  to load some data in the database.

Copy and save the following JSON data into a file called  `countries.json`  and save it inside the  `countries`  directory:

```
[
    {
        "model": "countries.country",
        "pk": 1,
        "fields": {
            "name": "Thailand",
            "capital": "Bangkok",
            "area": 513120
        }
    },
    {
        "model": "countries.country",
        "pk": 2,
        "fields": {
            "name": "Australia",
            "capital": "Canberra",
            "area": 7617930
        }
    },
    {
        "model": "countries.country",
        "pk": 3,
        "fields": {
            "name": "Egypt",
            "capital": "Cairo",
            "area": 1010408
        }
    }
]
```

This JSON contains database entries for three countries. Call the following command to load this data in the database:

```
$ python manage.py loaddata countries.json
Installed 3 object(s) from 1 fixture(s)
```


This adds three rows to the database.

With that, your Django application is all set up and populated with some data. You can now start adding Django REST framework to the project.

Django REST framework takes an existing Django model and converts it to JSON for a REST API. It does this with  **model serializers**. A model serializer tells Django REST framework how to convert a model instance into JSON and what data to include.

You’ll create your serializer for the  `Country`  model from above. Start by creating a file called  `serializers.py`  inside of the  `countries`  application. Once you’ve done that, add the following code to  `serializers.py`:

```
# countries/serializers.py
from rest_framework import serializers
from .models import Country

class CountrySerializer(serializers.ModelSerializer):
    class Meta:
        model = Country
        fields = ["id", "name", "capital", "area"]
```

This serializer,  `CountrySerializer`, subclasses  `serializers.ModelSerializer`  to automatically generate JSON content based on the model fields of  `Country`. Unless specified, a  `ModelSerializer`  subclass will include all fields from the Django model in the JSON. You can modify this behavior by setting  `fields`  to a list of data you wish to include.

Just like Django, Django REST framework uses  [views](https://docs.djangoproject.com/en/dev/topics/http/views/)  to query data from the database to display to the user. Instead of writing REST API views from scratch, you can subclass Django REST framework’s  [`ModelViewSet`](https://www.django-rest-framework.org/api-guide/viewsets/#modelviewset)  class, which has default views for common REST API operations.

**Note:**  The Django REST framework documentation refers to these views as  [actions](https://www.django-rest-framework.org/api-guide/viewsets/#viewset-actions).

Here’s a list of the actions that  `ModelViewSet`  provides and their equivalent HTTP methods:
|HTTP method | Action | Description |
|--|--|--|
|`GET` | `.list()` |Get a list of countries. |
|`GET` |`.retrieve()` |Get a single country. |
|`POST` |`.create()` | Create a new country.|
|`PUT`|`.update()`|Update a country.|
|`PATCH`|`.partial_update()`|Partially update a country.|
|`DELETE`|`.destroy()`|Delete a country.|

As you can see, these actions map to the standard HTTP methods you’d expect in a REST API. You can  [override these actions](https://www.django-rest-framework.org/api-guide/viewsets/#example)  in your subclass or  [add additional actions](https://www.django-rest-framework.org/api-guide/viewsets/#marking-extra-actions-for-routing)  based on the requirements of your API.

Below is the code for a  `ModelViewSet`  subclass called  `CountryViewSet`. This class will generate the views needed to manage  `Country`  data. Add the following code to  `views.py`  inside the  `countries`  application:

```
# countries/views.py
from rest_framework import viewsets

from .models import Country
from .serializers import CountrySerializer

class CountryViewSet(viewsets.ModelViewSet):
    serializer_class = CountrySerializer
    queryset = Country.objects.all()
   ```
    

In this class,  `serializer_class`  is set to  `CountrySerializer`  and  `queryset`  is set to  `Country.objects.all()`. This tells Django REST framework which serializer to use and how to query the database for this specific set of views.

Once the views are created, they need to be mapped to the appropriate URLs or endpoints. To do this, Django REST framework provides a  `DefaultRouter`  that will automatically generate URLs for a  `ModelViewSet`.

Create a  `urls.py`  file in the  `countries`  application and add the following code to the file:

```
# countries/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter

from .views import CountryViewSet

router = DefaultRouter()
router.register(r"countries", CountryViewSet)

urlpatterns = [
    path("", include(router.urls))
]
```

This code creates a  `DefaultRouter`  and registers  `CountryViewSet`  under the  `countries`  URL. This will place all the URLs for  `CountryViewSet`  under  `/countries/`.

**Note:**  Django REST framework automatically appends a forward slash (`/`) to the end of any endpoints generated by  `DefaultRouter`. You can disable this behavior like so:

`router = DefaultRouter(trailing_slash=False)` 

This will disable the forward slash at the end of endpoints.

Finally, you need to update the project’s base  `urls.py`  file to include all the  `countries`  URLs in the project. Update the  `urls.py`  file inside of the  `countryapi`  folder with the following code:

```
# countryapi/urls.py
from django.contrib import admin
from django.urls import path, include 
urlpatterns = [
    path("admin/", admin.site.urls),
 path("", include("countries.urls")), ]
 ``` 

This puts all the URLs under  `/countries/`. Now you’re ready to try out your Django-backed REST API. Run the following command in the root  `countryapi`  directory to start the Django development server:

`$ python manage.py runserver` 

The development server is now running. Go ahead and send a  `GET`  request to  `/countries/`  to get a list of all the countries in your Django project:

```
$ curl -i http://127.0.0.1:8000/countries/ -w '\n'

HTTP/1.1 200 OK
...

[
 {
 "id": 1,
 "name":"Thailand",
 "capital":"Bangkok",
 "area":513120
 },
 {
 "id": 2,
 "name":"Australia",
 "capital":"Canberra",
 "area":7617930
 },
 {
 "id": 3,
 "name":"Egypt",
 "capital":"Cairo",
 "area":1010408
 }
]
```

Django REST framework sends back a JSON response with the three countries you added earlier. The response above is formatted for readability, so your response will look different.

The  [`DefaultRouter`](https://www.django-rest-framework.org/api-guide/routers/#defaultrouter)  you created in  `countries/urls.py`  provides URLs for requests to all the standard API endpoints:

-   `GET /countries/`
-   `GET /countries/<country_id>/`
-   `POST /countries/`
-   `PUT /countries/<country_id>/`
-   `PATCH /countries/<country_id>/`
-   `DELETE /countries/<country_id>/`

You can try out a few more endpoints below. Send a  `POST`  request to  `/countries/`  to a create a new  `Country`  in your Django project:

```
$ curl -i http://127.0.0.1:8000/countries/ \
-X POST \
-H 'Content-Type: application/json' \
-d '{"name":"Germany", "capital": "Berlin", "area": 357022}' \
-w '\n'

HTTP/1.1 201 Created
...

{
 "id":4,
 "name":"Germany",
 "capital":"Berlin",
 "area":357022
}
```

This creates a new  `Country`  with the JSON you sent in the request. Django REST framework returns a  `201 Created`  status code and the new  `Country`.

**Note:**  By default, the response doesn’t include a new line at the end. This means that the JSON may run into your command prompt. The curl command above includes  `-w '\n'`  to add a newline character after the JSON to fix this issue.

You can view an existing  `Country`  by sending a request to  `GET /countries/<country_id>/`  with an existing  `id`. Run the following command to get the first  `Country`:

```
$ curl -i http://127.0.0.1:8000/countries/1/ -w '\n'

HTTP/1.1 200 OK
...

{
 "id":1,
 "name":"Thailand",
 "capital":"Bangkok",
 "area":513120
}
``` 

The response contains the information for the first  `Country`. These examples only covered  `GET`  and  `POST`  requests. Feel free to try out  `PUT`,  `PATCH`, and  `DELETE`  requests on your own to see how you can fully manage your model from the REST API.

As you’ve seen, Django REST framework is a great option for building REST APIs, especially if you have an existing Django project and you want to add an API.
