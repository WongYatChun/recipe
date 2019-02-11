# Django Tutorial 2: Create the public interface

We’re continuing the Web-poll application and will focus on creating the public interface – “views.”

## Overview¶

A view is a “type” of Web page in your Django application that generally serves a specific function and has a specific template. 

In our poll application, we’ll have the following four views:

- Question “index” page – displays the latest few questions.
- Question “detail” page – displays a question text, with no results but with a form to vote.
- Question “results” page – displays results for a particular question.
- Vote action – handles voting for a particular choice in a particular question.

In Django, web pages and other content are delivered by views. Each view is represented by a simple Python function (or method, in the case of class-based views). Django will choose a view by examining the URL that’s requested (to be precise, the part of the URL after the domain name).

A URL pattern is simply the general form of a URL - for example: **/newsarchive/\<year>/\<month>/**.

To get from a URL to a view, Django uses what are known as ‘URLconfs’. A URLconf maps URL patterns to views.

This tutorial provides basic instruction in the use of URLconfs, and you can refer to URL dispatcher for more information.

## Writing more views

Now let’s add a few more views to `polls/views.py`. These views are slightly different, because they take an argument:

In `polls/views.py`:

```python
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```


Wire these new views into the polls.urls module by adding the following path() calls:

In `polls/urls.py`

```python
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]

```

Take a look in your browser, at **“/polls/34/”**. It’ll run the `detail()` method and display whatever ID you provide in the URL. Try **“/polls/34/results/”** and **“/polls/34/vote/”** too – these will display the placeholder results and voting pages.

When somebody requests a page from your website – say, **“/polls/34/”**, Django will load the `mysite.urls` Python module because it’s pointed to by the `ROOT_URLCONF` setting. It finds the variable named `urlpatterns` and traverses the patterns in order. After finding the match at **'polls/'**, it strips off the matching text **("polls/")** and sends the remaining text – **"34/"** – to the ‘`polls.urls`’ URLconf for further processing. There it matches **'\<int:question_id>/'**, resulting in a call to the `detail()` view like so:

```python
detail(request=<HttpRequest object>, question_id=34)
```
The **question_id=34** part comes from **\<int:question_id>**. Using angle brackets “captures” part of the URL and sends it as a keyword argument to the view function. The **:question_id>** part of the string defines the name that will be used to identify the matched pattern, and the **\<int:** part is a converter that determines what patterns should match this part of the URL path.

## Write views that actually do something

Each view is responsible for doing one of two things: returning an `HttpResponse` object containing the content for the requested page, or raising an exception such as `Http404`. The rest is up to you.

Your view can read records from a database, or not. It can use a template system such as Django’s – or a third-party Python template system – or not. It can generate a PDF file, output XML, create a ZIP file on the fly, anything you want, using whatever Python libraries you want.

All Django wants is that `HttpResponse`. Or an exception.

Let’s use Django’s own database API. Here’s one stab at a new `index()` view, which displays the latest 5 poll questions in the system, separated by commas, according to publication date:

in `polls/views.py`

```python

from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# Leave the rest of the views (detail, results, vote) unchanged

```

There’s a problem here, though: the page’s design is hard-coded in the view. If you want to change the way the page looks, you’ll have to edit this Python code. So let’s use Django’s template system to separate the design from Python by creating a template that the view can use.

First, create a directory called **templates** in your **polls** directory. Django will look for templates in there.

Your project’s `TEMPLATES` setting describes how Django will load and render templates. The default settings file configures a `DjangoTemplates` backend whose `APP_DIRS` option is set to True. By convention `DjangoTemplates` looks for a “templates” subdirectory in each of the `INSTALLED_APPS`.

Within the **templates** directory you have just created, create another directory called **polls**, and within that create a file called **index.html**. In other words, your template should be at **polls/templates/polls/index.html**. Because of how the app_directories template loader works as described above, you can refer to this template within Django simply as **polls/index.html**.

```sh
[Rex@localhost polls]$ mkdir templates
[Rex@localhost polls]$ cd templates/
[Rex@localhost templates]$ mkdir polls
[Rex@localhost templates]$ cd polls/
[Rex@localhost polls]$ touch index.html
```

Put the following code in that template:
In `polls/templates/polls/index.html`

```html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

Now let’s update our index view in polls/views.py to use the template:
In `polls/views.py`

```python
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```
### A shortcut: render()

It’s a very common idiom to load a template, fill a context and return an `HttpResponse` object with the result of the rendered template. Django provides a shortcut. Here’s the full `index()` view, rewritten:

In `polls/views.py`

```python
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```


Note that once we’ve done this in all these views, we no longer need to import **loader** and **HttpResponse** (you’ll want to keep **HttpResponse** if you still have the stub methods for **detail**, **results**, and **vote**).

The `render()` function takes the request object as its first argument, a template name as its second argument and a dictionary as its optional third argument. It returns an **HttpResponse** object of the given template rendered with the given context.


## Raising a 404 error

Now, let’s tackle the question detail view – the page that displays the question text for a given poll. Here’s the view:

In `polls/views.py`

```python
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})


```
The new concept here: The view raises the `Http404` exception if a question with the requested ID doesn’t exist.

We’ll discuss what you could put in that **polls/detail.html** template a bit later, but if you’d like to quickly get the above example working, a file containing just:


### A shortcut: get_object_or_404()

It’s a very common idiom to use **get()** and raise **Http404** if the object doesn’t exist. Django provides a shortcut. Here’s the **detail()** view, rewritten:

In `polls/views.py`
```python
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})

```
There’s also a `get_list_or_404()` function, which works just as `get_object_or_404()` – except using `filter()` instead of `get()`. It raises `Http404` if the list is empty.


## Use the template system

Back to the **detail()** view for our poll application. Given the context variable **question**, here’s what the **polls/detail.html** template might look like:

In `polls/templates/polls/detail.html`

```html

<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>

```

The template system uses dot-lookup syntax to access variable attributes. In the example of **{{ question.question_text }}**, first Django does a dictionary lookup on the object `question`. Failing that, it tries an attribute lookup – which works, in this case. If attribute lookup had failed, it would’ve tried a list-index lookup.

Method-calling happens in the `{% for %}` loop: `question.choice_set.all` is interpreted as the Python code `question.choice_set.all()`, which returns an iterable of `Choice` objects and is suitable for use in the `{% for %}` tag.

## Removing hardcoded URLs in templates

Remember, when we wrote the link to a question in the **polls/index.html** template, the link was partially hardcoded like this:

```html
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

The problem with this hardcoded, tightly-coupled approach is that it becomes challenging to change URLs on projects with a lot of templates. However, since you defined the name argument in the **path()** functions in the **polls.urls** module, you can remove a reliance on specific URL paths defined in your url configurations by using the **{% url %}** template tag:


```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

The way this works is by looking up the URL definition as specified in the **polls.urls** module. 

## Namespacing URL names

The tutorial project has just one app, **polls**. In real Django projects, there might be five, ten, twenty apps or more. How does Django differentiate the URL names between them? For example, the **polls** app has a **detail** view, and so might an app on the same project that is for a blog. How does one make it so that Django knows which app view to create for a url when using the **{% url %}** template tag?

The answer is to add namespaces to your URLconf. In the **polls/urls.py** file, go ahead and add an **app_name** to set the application namespace:

In `polls/urls.py`

```python
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

Now change your `polls/index.html` template from:
In `polls/templates/polls/index.html`
```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

to point at the namespaced detail view:
```html
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li
```
