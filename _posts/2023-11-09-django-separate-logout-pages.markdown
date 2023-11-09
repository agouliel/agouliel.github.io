---
layout: post
title:  "Django: How to create separate logout pages for the Admin app and your own apps"
date:   2023-11-09 15:21:59 +0200
categories: django
---
Create your Django project:

`mkdir logout_sample`   
`cd logout_sample`   
`python -m venv .venv`   
`source .venv/bin/activate`   
`pip install django`   
`django-admin startproject config .`   

----------------------------------------------------   

Create the `accounts` app:   

`python manage.py startapp accounts`   

and add a custom user model:

    # accounts/models.py
    from django.contrib.auth.models import AbstractUser
    from django.db import models

    class CustomUser(AbstractUser):
      pass

Modify the settings to include your new app and specify the user model:

    # config/settings.py
    INSTALLED_APPS = [
      ...
      'django.contrib.staticfiles',

      # Our apps
      'accounts',
    ]
    ...
    AUTH_USER_MODEL = 'accounts.CustomUser'

Perform the final steps:   

`python manage.py makemigrations`   
`python manage.py migrate`   
`python manage.py createsuperuser`   

----------------------------------------------------

Create the `pages` app:   

`python manage.py startapp pages`   

and include it in the settings:

    # config/settings.py
    INSTALLED_APPS = [
        ...
        'django.contrib.staticfiles',

        # Our apps
        'accounts',
        'pages',
    ]

----------------------------------------------------

Modify settings to enable a project-level templates folder:

    TEMPLATES = [
      {
      ...
      #'DIRS': [],
      'DIRS': [BASE_DIR / 'templates'],

Create the templates folder:   

`mkdir templates`   

and add your base template:

    <!-- templates/_base.html -->   
    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>{% raw %}{% block title %}{% endraw %}Our Site{% raw %}{% endblock title %}{% endraw %}</title>
    </head>
    <body>
        <div class="container">
        {% raw %}{% block content %}{% endraw %}
        {% raw %}{% endblock content %}{% endraw %}
        </div>
    </body>
    </html>

Add the app's URLs:

    # pages/urls.py
    from django.urls import path
    from .views import HomePageView

    urlpatterns = [
        path('', HomePageView.as_view(), name='home'),
    ]

Modify the views:

    # pages/views.py
    from django.views.generic import TemplateView

    class HomePageView(TemplateView):
      template_name = 'home.html'

Finally modify the project's URLs:

    # config/urls.py
    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        # from django.contrib.auth import views
        # class LoginView(): template_name = "registration/login.html"
        # class LogoutView(): template_name = "registration/logged_out.html"
        # path("login/", views.LoginView.as_view(), name="login"),
        # path("logout/", views.LogoutView.as_view(), name="logout"),
        path('accounts/', include('django.contrib.auth.urls')),
        path('', include('pages.urls')),
    ]

----------------------------------------------------

Add your homepage template:

    <!-- templates/home.html -->
    {% raw %}{% extends "_base.html" %}{% endraw %}
    {% raw %}{% block title %}{% endraw %}Home{% raw %}{% endblock title %}{% endraw %}
    {% raw %}{% block content %}{% endraw %}
    <h1>This is our home page.</h1>
    {% raw %}{% if user.is_authenticated %}{% endraw %}
      <p>Hi {{ user.email }}!</p>
    {% raw %}{% else %}{% endraw %}
      <p>You are not logged in</p>
      <a href="{% raw %}{% url 'login' %}{% endraw %}">Log In</a>
    {% raw %}{% endif %}{% endraw %}
    {% raw %}{% endblock content %}{% endraw %}

Create the folder that Django expects for login and logout templates:
  
`mkdir templates/registration`    

and add the login template there:

    <!-- templates/registration/login.html -->
    {% raw %}{% extends "_base.html" %}{% endraw %}
    {% raw %}{% block title %}{% endraw %}Log In{% raw %}{% endblock title %}{% endraw %}
    {% raw %}{% block content %}{% endraw %}
    <h2>Log In</h2>
    <form method="post">
    {% raw %}{% csrf_token %}{% endraw %}
    {{ form.as_p }}
    <button type="submit">Log In</button>
    </form>
    {% raw %}{% endblock content %}{% endraw %}

`LOGIN_REDIRECT_URL = 'home'`   

----------------------------------------------------

Simple logout with redirect (no logout screen):

    <!-- templates/home.html -->
    <p>Hi {{ user.email }}!</p>
    <p><a href="{% raw %}{% url 'logout' %}{% endraw %}">Log Out</a></p>

The above just redirects to the admin logout. Let's try adding the line:

`LOGOUT_REDIRECT_URL = 'home'`   

Now we have also lost the admin logout.

----------------------------------------------------

First attempt for logout screen:

Remove `LOGOUT_REDIRECT_URL = 'home'` and add the below:

    <!-- templates/registration/logged_out.html -->
    {% raw %}{% extends "_base.html" %}{% endraw %}
    {% raw %}{% block title %}{% endraw %}Log Out{% raw %}{% endblock title %}{% endraw %}
    {% raw %}{% block content %}{% endraw %}
    <h2>You are logged out</h2>
    {% raw %}{% endblock content %}{% endraw %}

While this works, it again replaces (overrides) the admin logout screen, because the admin template has the same name: `contrib/admin/templates/registration/logged_out.html`

(The admin login was not replaced, because it exists in a special location:   
`contrib/admin/templates/admin/login.html`   
To find the above location, you can use the information here:
https://stackoverflow.com/questions/19919547/where-can-i-find-the-source-file-of-admin-site-urls)

----------------------------------------------------

Solution:

Rename `registration/logged_out.html` to `registration/logout.html`
and change `config/urls.py` according to the following article:
https://stackoverflow.com/questions/59692899/using-loginview-and-logoutview-with-custom-templates

    # config/urls.py
    from django.contrib.auth import views

    urlpatterns = [
        path('admin/', admin.site.urls),
        path(
          'accounts/logout/',
          views.LogoutView.as_view(
            template_name='registration/logout.html',
            next_page=None),
          name='logout'),
        path('accounts/', include('django.contrib.auth.urls')),
        path('', include('pages.urls')),
    ]
