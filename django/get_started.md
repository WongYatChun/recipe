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

### Database setup

Now, open up `mysite/settings.py`. It’s a normal Python module with module-level variables representing Django settings.

By default, the configuration uses SQLite. If you’re new to databases, or you’re just interested in trying Django, this is the easiest choice. SQLite is included in Python, so you won’t need to install anything else to support your database. 

While you’re editing `mysite/settings.py`, set `TIME_ZONE` to your time zone.

Also, note the `INSTALLED_APPS` setting at the top of the file. That holds the names of all Django applications that are activated in this Django instance. Apps can be used in multiple projects, and you can package and distribute them for use by others in their projects.

By default, `INSTALLED_APPS` contains the following apps, all of which come with Django:
- `django.contrib.admin` – The admin site. You’ll use it shortly.
- `django.contrib.auth` – An authentication system.
- `django.contrib.contenttypes` – A framework for content types.
- `django.contrib.sessions` – A session framework.
- `django.contrib.messages` – A messaging framework.
- `django.contrib.staticfiles` – A framework for managing static files.
These applications are included by default as a convenience for the common case.

Some of these applications make use of at least one database table, though, so we need to create the tables in the database before we can use them. To do that, run the following command:

```sh
$ python manage.py migrate
```

The `migrate` command looks at the `INSTALLED_APPS` setting and creates any necessary database tables according to the database settings in your `mysite/settings.py` file and the database migrations shipped with the app (we’ll cover those later). You’ll see a message for each migration it applies. If you’re interested, run the command-line client for your database and type \dt (PostgreSQL), SHOW TABLES; (MySQL), .schema (SQLite), or SELECT TABLE_NAME FROM USER_TABLES; (Oracle) to display the tables Django created.


### Creating Models

Model is essentially the database layout with additional metadata. Django follows the DRY Principal. The goal is to define the data model in one place and automatically derive things from it. This includes the migrations which are entirely derived from the model file and are essentially just a history that Django can roll through to update the database schema to match the current model.

In our simple poll app, we will create two models : **Questions** and **Choice**. A **Question** has a question and a publication date. A **Choice** has two fields: the text of the choice and a vote tally. Each **Choice** is associated with a **Question**. 

These concepts are represented by simple Python classes. Edit `polls/models.py` file so it looks like this:

```python

from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

```

 Each model is represented by a class that subclasses `django.db.models.Model`. Each model has a number of class variables, each of which represents a database field in the model.

Each field is represented by an instance of a Field class – e.g., `CharField` for character fields and `DateTimeField` for datetimes. This tells Django what type of data each field holds.

The name of each Field instance (e.g. `question_text` or `pub_date`) is the field’s name, in machine-friendly format. You’ll use this value in your Python code, and your database will use it as the column name.

You can use an optional first positional argument to a Field to designate a human-readable name. That’s used in a couple of introspective parts of Django, and it doubles as documentation. If this field isn’t provided, Django will use the machine-readable name. In this example, we’ve only defined a human-readable name for `Question.pub_date`. For all other fields in this model, the field’s machine-readable name will suffice as its human-readable name.

Some Field classes have required arguments. `CharField`, for example, requires that you give it a `max_length`. That’s used not only in the database schema, but in validation, as we’ll soon see.

A Field can also have various optional arguments; in this case, we’ve set the default value of `votes` to 0.

Finally, note a relationship is defined, using `ForeignKey`. That tells Django each Choice is related to a single Question. Django supports all the common database relationships: many-to-one, many-to-many, and one-to-one.

### Activating models

That small bit of model code gives Django a lot of information. With it, Django is able to:
- Create a database schema (`CREATE TABLE` statements) for this app.
- Create a Python database-access API for accessing Question and Choice objects.
But first we need to tell our project that the polls app is installed.

```
Philosophy

Django apps are “pluggable”
- You can use an app in multiple projects
- you can distribute apps

because they don’t have to be tied to a given Django installation.
```

To include the app in our project, we need to add a reference to its configuration class in the `INSTALLED_APPS` setting. The `PollsConfig` class is in the `polls/apps.py` file, so its dotted path is '`polls.apps.PollsConfig`'. Edit the `mysite/settings.py` file and add that dotted path to the `INSTALLED_APPS` setting. It’ll look like this:

```python

INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

```
Now Django knows to include the polls app. Let’s run another command:

```sh
$ python manage.py makemigrations polls
```
By running `makemigrations`, you’re telling Django that you’ve made some changes to your models (in this case, you’ve made new ones) and that you’d like the changes to be stored as a migration.

Migrations are how Django stores changes to your models (and thus your database schema) - they’re just files on disk. You can read the migration for your new model if you like; it’s the file `polls/migrations/0001_initial.py`. Don’t worry, you’re not expected to read them every time Django makes one, but they’re designed to be human-editable in case you want to manually tweak how Django changes things.

There’s a command that will run the migrations for you and manage your database schema automatically - that’s called `migrate`, and we’ll come to it in a moment - but first, let’s see what SQL that migration would run. The `sqlmigrate` command takes migration names and returns their SQL:

```sh
$ python manage.py sqlmigrate polls 0001
```
You should see something similar to the following (we’ve reformatted it for readability):

```sql
BEGIN;
--
-- Create model Choice
--
CREATE TABLE "polls_choice" (
    "id" serial NOT NULL PRIMARY KEY,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL
);
--
-- Create model Question
--
CREATE TABLE "polls_question" (
    "id" serial NOT NULL PRIMARY KEY,
    "question_text" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);
--
-- Add field question to choice
--
ALTER TABLE "polls_choice" ADD COLUMN "question_id" integer NOT NULL;
ALTER TABLE "polls_choice" ALTER COLUMN "question_id" DROP DEFAULT;
CREATE INDEX "polls_choice_7aa0f6ee" ON "polls_choice" ("question_id");
ALTER TABLE "polls_choice"
  ADD CONSTRAINT "polls_choice_question_id_246c99a640fbbd72_fk_polls_question_id"
    FOREIGN KEY ("question_id")
    REFERENCES "polls_question" ("id")
    DEFERRABLE INITIALLY DEFERRED;

COMMIT;

```
Note the following:
- The exact output will vary depending on the database you are using. The example above is generated for PostgreSQL.
- Table names are automatically generated by combining the name of the app (polls) and the lowercase name of the model – `question` and `choice`. (You can override this behavior.)
- Primary keys (IDs) are added automatically. (You can override this, too.)
- By convention, Django appends "_id" to the foreign key field name. (you can override this, as well.)
- The foreign key relationship is made explicit by a `FOREIGN KEY` constraint. Don’t worry about the `DEFERRABLE` parts; that’s just telling PostgreSQL to not enforce the foreign key until the end of the transaction.
- It’s tailored to the database you’re using, so database-specific field types such as `auto_increment` (MySQL), `serial` (PostgreSQL), or `integer primary key autoincrement` (SQLite) are handled for you automatically. Same goes for the quoting of field names – e.g., using double quotes or single quotes.
- The `sqlmigrate` command doesn’t actually run the migration on your database - it just prints it to the screen so that you can see what SQL Django thinks is required. It’s useful for checking what Django is going to do or if you have database administrators who require SQL scripts for changes.

If you’re interested, you can also run `python manage.py check`; this checks for any problems in your project without making migrations or touching the database.

Now, run migrate again to create those model tables in your database:

```sh
$ python manage.py migrate
```

The `migrate` command takes all the migrations that haven’t been applied (Django tracks which ones are applied using a special table in your database called `django_migrations`) and runs them against your database - essentially, synchronizing the changes you made to your models with the schema in the database.

Migrations are very powerful and let you change your models over time, as you develop your project, without the need to delete your database or tables and make new ones - it specializes in upgrading your database live, without losing data. For now, remember the three-step guide to making model changes:
1. Change your models (in models.py).
2. Run python manage.py makemigrations to create migrations for those changes
3. Run python manage.py migrate to apply those changes to the database.

The reason that there are separate commands to make and apply migrations is because you’ll commit migrations to your version control system and ship them with your app; they not only make your development easier, they’re also usable by other developers and in production.

### Play with the API

Let’s hop into the interactive Python shell and play around with the free API Django gives you. To invoke the Python shell, use this command:

```sh
$ python manage.py shell
```

We’re using this instead of simply typing “python”, because `manage.py` sets the `DJANGO_SETTINGS_MODULE` environment variable, which gives Django the Python import path to your `mysite/settings.py` file.

Once you’re in the shell, explore the database API:

```python

>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```

However, `<Question: Question object (1)>` isn’t a helpful representation of this object. Let’s fix that by editing the Question model (in the `polls/models.py` file) and adding a `__str__()` method to both `Question` and `Choice`:

```python
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```
It’s important to add `__str__()` methods to your models: 
1. for your own convenience when dealing with the interactive prompt
2. objects’ representations are used throughout Django’s automatically-generated admin.

Note these are normal Python methods. Let’s add a custom method, just for demonstration:

In `polls/models.py`
```python
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now().date() - datetime.timedelta(days=1)

```

Note the addition of import `datetime` and `from django.utils import timezone`, to reference Python’s standard `datetime` module and Django’s time-zone-related utilities in `django.utils.timezone`, respectively. 

Save these changes and start a new Python interactive shell by running 
```sh
$ python manage.py shell 
```

```python

>>> from polls.models import Choice, Question

# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set to hold the "other side" of a ForeignKey relation
# (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()

```
## Introducing the Django Admin

Generating admin sites for your staff or clients to add, change, and delete content is tedious work that doesn’t require much creativity. For that reason, Django entirely automates creation of admin interfaces for models.

Django was written in a newsroom environment, with a very clear separation between “content publishers” and the “public” site. Site managers use the system to add news stories, events, sports scores, etc., and that content is displayed on the public site. Django solves the problem of creating a unified interface for site administrators to edit content.

The admin isn’t intended to be used by site visitors. It’s for site managers.

### Creating an admin user¶

First we’ll need to create a user who can login to the admin site. Run the following command:

```sh
$ python manage.py createsuperuser
```
Enter your desired username and press enter:
```sh
Username: admin
```
You will then be prompted for your desired email address:
```sh
Email address: admin@example.com
```
The final step is to enter your password. You will be asked to enter your password twice, the second time as a confirmation of the first.

```sh
Password: **********
Password (again): *********
Superuser created successfully.
```
### Start the development server¶

The Django admin site is activated by default. Let’s start the development server and explore it.

```sh
$ python manage.py runserver
```

Now, open a Web browser and go to “/admin/” on your local domain – e.g., http://127.0.0.1:8000/admin/. You should see the admin’s login screen:


Since translation is turned on by default, the login screen may be displayed in your own language, depending on your browser’s settings and if Django has a translation for this language.

### Enter the admin site

Now, try logging in with the superuser account you created in the previous step. You should see the Django admin index page:

You should see a few types of editable content: groups and users. They are provided by `django.contrib.auth`, the authentication framework shipped by Django.

### Make the poll app modifiable in the admin

We need to tell the admin that Question objects have an admin interface. To do this, open the polls/admin.py file, and edit it to look like this:

In `polls/admin.py`

```python
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```



### Explore the free admin functionality

Now that we’ve registered Question, Django knows that it should be displayed on the admin index page:

Click “Questions”. Now you’re at the “change list” page for questions. This page displays all the questions in the database and lets you choose one to change it. There’s the “What’s up?” question we created earlier.
