---
layout: post
title:  "Django: How to create separate logout pages for the Admin app and your own apps"
date:   2023-11-09 15:21:59 +0200
categories: django
---
mkdir logout_sample
cd logout_sample
python -m venv .venv
source .venv/bin/activate
pip install django

django-admin startproject config .

----------------------------------------------------

python manage.py startapp accounts

# accounts/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
  pass

# config/settings.py
INSTALLED_APPS = [
    ...
    'django.contrib.staticfiles',

    # Apps
    'accounts',
]
...
AUTH_USER_MODEL = 'accounts.CustomUser'

python manage.py makemigrations
python manage.py migrate

python manage.py createsuperuser

----------------------------------------------------

python manage.py startapp pages

# config/settings.py
INSTALLED_APPS = [
    ...
    'django.contrib.staticfiles',

    # Apps
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

mkdir templates

<!-- templates/_base.html -->
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>{% block title %}Our Site{% endblock title %}</title>
</head>
<body>
    <div class="container">
    {% block content %}
    {% endblock content %}
    </div>
</body>
</html>


# pages/urls.py
from django.urls import path
from .views import HomePageView

urlpatterns = [
    path('', HomePageView.as_view(), name='home'),
]

# pages/views.py
from django.views.generic import TemplateView

class HomePageView(TemplateView):
  template_name = 'home.html'

# config/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('django.contrib.auth.urls')),
    path('', include('pages.urls')),
]

# from django.contrib.auth import views
# class LoginView(FormView): template_name = "registration/login.html"
# class LogoutView(TemplateView): template_name = "registration/logged_out.html"
# path("login/", views.LoginView.as_view(), name="login"),
# path("logout/", views.LogoutView.as_view(), name="logout"),

----------------------------------------------------

<!-- templates/home.html -->
{% extends "_base.html" %}
{% block title %}Home{% endblock title %}
{% block content %}
<h1>This is our home page.</h1>
{% if user.is_authenticated %}
  <p>Hi {{ user.email }}!</p>
{% else %}
  <p>You are not logged in</p>
  <a href="{% url 'login' %}">Log In</a>
{% endif %}
{% endblock content %}

mkdir templates/registration

<!-- templates/registration/login.html -->
{% extends "_base.html" %}
{% block title %}Log In{% endblock title %}
{% block content %}
<h2>Log In</h2>
<form method="post">
{% csrf_token %}
{{ form.as_p }}
<button type="submit">Log In</button>
</form>
{% endblock content %}

LOGIN_REDIRECT_URL = 'home'

----------------------------------------------------

Simple logout with redirect (no logout screen):

<!-- templates/home.html -->
<p>Hi {{ user.email }}!</p>
<p><a href="{% url 'logout' %}">Log Out</a></p>

The above just redirects to the admin logout. Let's try adding the line:

LOGOUT_REDIRECT_URL = 'home'

Now we have also lost the admin logout.

----------------------------------------------------

First attempt for logout screen:

Remove LOGOUT_REDIRECT_URL = 'home' and add the below:

<!-- templates/registration/logged_out.html -->
{% extends "_base.html" %}
{% block title %}Log Out{% endblock title %}
{% block content %}
<h2>You are logged out</h2>
{% endblock content %}

While this works, it again replaces (overrides) the admin logout screen, because the template has the same name.
contrib/admin/templates/registration/logged_out.html

(The admin login was not replaced, because it exists in a special location:
contrib/admin/templates/admin/login.html
To find the above location, you can use the information here:
https://stackoverflow.com/questions/19919547/where-can-i-find-the-source-file-of-admin-site-urls)

----------------------------------------------------

Solution:

Rename registration/logged_out.html to registration/logout.html
and change config/urls.py according to the following article:
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
