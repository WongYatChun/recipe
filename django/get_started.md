# Meet Django

Django is a high-level Python Web framework that encourages rapid development and clean, pragmatic design. It takes care of much of the hassle of Web development, so you can focus on writing your app without needing to reinvent the wheel.


## Install Django

Being a Python Web framework, Django requires Python. So you need get the latest version of Python (e.g. using Anaconda?)

Then they are two options:

1. Install by Anaconda

2. `$ pip install Django`

```python
import django
print(django.get_version())
```

or bash command
```bash
$ python -m django --version
```

## Demo project: writing your 1st Django app

### Creating a project

First, you need to auto-generate some code that establishes a Django project – a collection of settings for an instance of Django, including database configuration, Django-specific options and application-specific settings.

```bash
$ mkdir demo_project
$ cd demo_project
$ django-admin startproject mysite
```
This will create a mysite directory in your current directory. 

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```

```
Note:
Avoid naming projects after built-in Python or Django components. e.g. django (which will conflict with Django itself) or test (which conflicts with a built-in Python package).
```

These files are:

* The outer mysite/: root directory, container for the project, can be renamed.
* manage.py: A command-line utility to interact with this Django project.
* The inner mysite/: the actual Python package for the project. Its name is the Python package name you’ll need to use to import anything inside it (e.g. mysite.urls).
* mysite/__init__.py: An empty file that tells Python that this directory should be considered a Python package.
* mysite/settings.py: Settings/configuration for this project.
* mysite/urls.py: The URL declarations for this Django project; a “table of contents” of your Django-powered site.
* mysite/wsgi.py: An entry-point for WSGI-compatible web servers to serve your project.

### The development server

Verify your Django project works. Change into the outer mysite directory.
```
$ cd mysite
$ python manage.py runserver
```

You’ll see the following output on the command line:

```
Performing system checks...

System check identified no issues (0 silenced).

You have unapplied migrations; your app may not work properly until they are applied.
Run 'python manage.py migrate' to apply them.

January 11, 2019 - 15:50:53
Django version 2.1, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

```
Note:
Ignore the warning about unapplied database migrations for now; we’ll deal with the database shortly.
```

You’ve started the Django development server.

Now’s a good time to note: don’t use this server in anything resembling a production environment. It’s intended only for use while developing. (We’re in the business of making Web frameworks, not Web servers.)


```
Changing the port

By default, the runserver command starts the development server on the internal IP at port 8000.

If you want to change the server’s port, pass it as a command-line argument. For instance, this command starts the server on port 8080

$ python manage.py runserver 8080

If you want to change the server’s IP, pass it along with the port. For example, to listen on all available public IPs (which is useful if you are running Vagrant or want to show off your work on other computers on the network), use:

$ python manage.py runserver 0:8000

0 is a shortcut for 0.0.0.0.
```

### Creating the Polls app

Each application you write in Django consists of a Python package that follows a certain convention. Django comes with a utility that automatically generates the basic directory structure of an app, so you can focus on writing code rather than creating directories.

Your apps can live anywhere on your Python path. In this tutorial, we’ll create our poll app right next to your manage.py file so that it can be imported as its own top-level module, rather than a submodule of mysite.

To create your app, make sure you’re in the same directory as manage.py and type this command:

```
python manage.py startapp polls
```
That’ll create a directory polls, which is laid out like this:

```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```

This directory structure will house the poll application.

### Write your first view

Open the file polls/views.py and put the following Python code in it:

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
This is the simplest view possible in Django. To call the view, we need to map it to a URL - and for this we need a URLconf.

To create a URLconf in the polls directory, create a file called urls.py. 
```
$ cd polls
$ touch urls.py
```
Your app directory should now look like:
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    urls.py
    views.py
```

In the `polls/urls.py` file include the following code:
```py
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```


The next step is to point the root URLconf at the `polls.urls` module. In `mysite/urls.py`, add an import `for django.urls.include` and insert an `include()` in the `urlpatterns` list, so you have:
```py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
The `include()` function allows referencing other URLconfs. Whenever Django encounters `include()`, it chops off whatever part of the URL matched up to that point and sends the remaining string to the included URLconf for further processing.

The idea behind `include()` is to make it easy to plug-and-play URLs. Since polls are in their own URLconf (`polls/urls.py`), they can be placed under “/polls/”, or under “/fun_polls/”, or under “/content/polls/”, or any other path root, and the app will still work.

```
When to use include()

You should always use include() when you include other URL patterns. admin.site.urls is the only exception to this.
```

You have now wired an index view into the URLconf. Lets verify it’s working, run the following command:
```
$ python manage.py runserver
```
Go to http://localhost:8000/polls/ in your browser, and you should see the text “Hello, world. You’re at the polls index.”, which you defined in the index view.

The `path()` function is passed four arguments, two required: `route` and `view`, and two optional: `kwargs`, and `name`. 

**path() argument: route**
```
route is a string that contains a URL pattern. When processing a request, Django starts at the first pattern in urlpatterns and makes its way down the list, comparing the requested URL against each pattern until it finds one that matches.

Patterns don’t search GET and POST parameters, or the domain name. For example, in a request to https://www.example.com/myapp/, the URLconf will look for myapp/. In a request to https://www.example.com/myapp/?page=3, the URLconf will also look for myapp/.
```
**path() argument: view**
```
When Django finds a matching pattern, it calls the specified view function with an HttpRequest object as the first argument and any “captured” values from the route as keyword arguments. 
```
**path() argument: kwargs**
```
Arbitrary keyword arguments can be passed in a dictionary to the target view. 
```
**path() argument: name**
```
Naming your URL lets you refer to it unambiguously from elsewhere in Django, especially from within templates. This powerful feature allows you to make global changes to the URL patterns of your project while only touching a single file.
```
