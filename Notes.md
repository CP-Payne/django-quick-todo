# Django notes here...

1. Install Django
   `pip install django`

2. Setup django project
   `django-admin startproject <project_name>`

Structure Notes:

- quick_todo
  - quick_todo
    - **init**.py --> Treat directory like python package
    - asgi.py --> Special configuration files
    - settings.py --> Install different django application, plugins, middleware, databases, etc.
    - urls.py --> Configure Routes to direct to django applications
    - wsgi.py --> Special configuration files
  - manange.py --> Acts as a command line tool: Make database migrations, run server, create admin users, etc.
