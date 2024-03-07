# Django notes here...

1. Install Django
   `pip install django`

2. Setup django project
   `django-admin startproject <project_name>`

Project Structure:

- quick_todo
  - quick_todo
    - **init**.py --> Treat directory like python package
    - asgi.py --> Special configuration files
    - settings.py --> Install different django application, plugins, middleware, databases, etc.
    - urls.py --> Configure Routes to direct to django applications
    - wsgi.py --> Special configuration files
  - manange.py --> Acts as a command line tool: Make database migrations, run server, create admin users, etc.

The above is a django project, we now need to create a django application if we want some sort of executable code or for our website to appear. A django application is ment to be a standalone application that you can "plug and play", meaning that I can take it out of this django project and add it to another django project. Django apps contains things like:

- Database models
- different views
- Routes
- Templates
- etc..

Create a django app in project:

1. Change directory to project
   `cd <project_name>`

2. Create django app
   `python manage.py startapp <app_name>`

App Structure

- quick_todo
  - quick_todo
    - ...
  - todo_app
    - migrations
    - **init**.py
    - admin.py --> Register database models to view in admin panel
    - apps.py
    - models.py --> Database models are added here
    - tests.py --> Automated test cases
    - views.py --> Add different routes
  - manage.py

3. Link app to project

   1. Go to project_name folder
   2. Go to `settings.py`
   3. Add the app name under `INSTALLED_APPS`. (i.e. "todo_app")

4. Create file `urls.py` in the app folder --> Routes will be added to this file
5. Connect `views.py` with `urls.py`

Creating a View Function:
example:

Note: Import `HttpResponse` to use responses

```python
def home(request):
    return Http.Response("Hello World")

```

The above view needs to be linked with a route in the `urls.py` (in app directory) file created in the previous step.

Import path from django.urls as well as the views then add the the path the `urlpatterns` (list):

```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.home, name="home")
]
```

From the path added in the example above, if a user visits root "/" then the home function will be called.

We have now configured the url in our application. We also need to configure the URL TO our application. The `urls.py` file inside directory with the same name as our project contains the base routes.

Inside the project name directory (`quick_todo` in this case) open the `urls.py` and add the following to the `urlpatterns` list:

```python
urlpatterns = [
    ...
    path("", include("todo_app.urls"))
]
```

The above essentially means that any requests to "/" will be forwarded to the routes inside the `urls.py` inside our app called `todo_app` which will then be handled by the appropriate views (functions).
For example, if we had the following:

```python
path('todo/', include("todo_app.urls"))
```

Any requests which starts with the path `todo/` will be forwarded to `urls.py` in our app called `todo_app`.
Note: `todo/` will not be passed to the `urls.py` file, everything after that will be passed. For example, if we send a request to `todo/home`, only the `home` path will be passed the the app's router.

## Running The Server

You can start the server with `manage.py` using the following command:

```python
python manage.py runserver
```

# Templates

A template is a reusable html file that allows us to display dynamic data.

Create a `templates` directory inside your application directory.
Note: If you don't name it templates it will not work.

Inside `templates` create a `base.html` file. The templates use a templating engine called `Jinja` which allows you to display dynamic data.

Inside the template (`html` files) we can create something known as a `block` which is overridable pieces of content that is overridden by other templates.

Example `base.html`:

```python
<HTML CONTENT HERE>
    {% block title %} Django App {% endblock %}
<HTML CONTENT HERE>
    {% block content %} {% endblock %}
<HTML CONTENT HERE>
```

We can now create another html file (`home.html` for example) which will inherit the code from `base.html` and from which we can add custom content to that html by overridding the blocks:

```html
{% extends "base.html" %}
<!-- Uses html from base.html -->
{% block title %} Home Page {% endblock %}
<!-- Overrides the title block and changes it to 'Home Page' -->
{% block content %}
<!-- Adds the p tag to the content block by overriding the it -->
<p>This is the home page</p>
{% endblock %}
```

Note: It seems like incorrect spacing in template could result in error

To render the template we need to use the `render` function inside our `views.py` file in the approriate function. For example, in our `home` function we can pass through the template:

```python
def home(request):
    return render(request, "home.html")
```

Note: Make sure the template (`home.html` in this case) is inside the `templates` directory.

# Database Models

Django provides an Object Relational Mapping (_ORM_) which means we can create databases in python code using Models.
Migration --> Automated code that takes the models and creates the corresponding model in your database (postgres, sqlite, etc).

You create models in the `models.py` file inside the app directory. Below is an example of table that will be created in a database:

```python
class TodoItem(models.Model):
    title = models.Charfield(max_length=200)
    complete = models.BooleanField(default=False)
```

Further Research:
How to reference and add foreign keys, etc.

We can now register the model with our admin panel in order to modify and view them. Through the admin panel we will be able to add data to the table, delete data, etc.

Inside `admin.py` within the app's directory, add the following to register the model:

```python
from .models import TodoItem

# Register your models here.
admin.site.register(TodoItem)
```

We now need to make a migration which will apply the model to the database, in our case create the table. Any time we make a change to a model we make a migration, so that the database is updated. This can be done with:

The below generates the files that defines the changes to be made to the database.

```bash
python manage.py makemigrations
```

The below will apply the migrations to the database

```bash
python manage.py migrate
```

## Rendering dynamic data

We can execute python code and access variables:

- Variables are accessed using `{{}}` and python code is executed within `{% %}`

```python
{% extends "base.html" %} {% block content %}
<h1>Todo List</h1>
<ul>
  {% for todo in todos %}
  <!-- Placeholder-->
  <li>
    {{ todo.title }}: {% if todo.completed %}Completed{% else %}Not completed{%
    endif %}

    <!-- Placeholder-->
  </li>

  <!-- Placeholder-->
  {% endfor %}

  <!-- Placeholder-->
</ul>

<!-- Placeholder-->
{% endblock %}
```

The `todos` variables will be passed to this template. In the views file, import the model and create a handler:

```python
from .models import TodoItem

def todos(request):
    items = TodoItem.objects.all()
    return render(request, "todos.html", {"todos": items })
```

In the above code we create a todo handler which will render the todos.html and pass a dictionary containing the todos. The todos are obtained from the database by qurying all in the TodoItem model (table in database)

Add a route to the handler in the `urls.py`:

```python
urlpatterns = [
    ...[SNIP]...
    path("todos/", views.todos, name="todos")
]
```

## Django Admin Panel

The django admin panel allows to manage users and models.

To access the admin panel we need to create a superuser:

```bash
python manage.py createsuperuser
```

You can now visit `http://localhost:8000/admin` and log in with the created superuser. From here, we can add data to our database, manage database, manage users, etc.
