---
layout: post
title:  "Deploy Django blog in Google Cloud - Part-2"
date:   2020-11-15 19:30:00 +0100
categories: django python Gcloud Google Cloud
---

After migrating my database to Google Cloud SQL, I now want to deploy my local Django app to the Google App Engine.

I am still following the blog post of [Bennett Garner][bennett-garner]. 


Before using `gscloud app deploy`, I need to modify a few files. 

The following is just copy-pasted from [Bennett Garner][bennett-garner] medium page.

## /app.yaml
This is the basic config file for App Engine. You can change a lot of settings here, but the most basic configuration will get your app up and running for now. Just use this:

```yaml
# [START django_app]
runtime: python37

handlers:
# This configures Google App Engine to serve the files in the app's
# static directory.
- url: /static
  static_dir: static/

# This handler routes all requests not caught above to the main app. 
# It is required when static routes are defined, but can be omitted 
# (along with the entire handlers section) when there are no static 
# files defined.
- url: /.*
  script: auto

# [END django_app]
```

## /main.py
This is a file App Engine looks for by default. In it, we import the WSGI information so GAE knows how to start our app.

```python
from mysite.wsgi import application

# App Engine by default looks for a main.py file at the root of the app
# directory with a WSGI-compatible object called app.
# This file imports the WSGI-compatible object of the Django app,
# application from mysite/wsgi.py and renames it app so it is
# discoverable by App Engine without additional configuration.
# Alternatively, you can add a custom entrypoint field in your app.yaml:
# entrypoint: gunicorn -b :$PORT mysite.wsgi
app = application
```

## /requirements.txt
App Engine needs to know what dependencies to install in order to get your app running. If you’ve got your app running locally with no problems (In an IDE or on Ubuntu), you can use pip to freeze your dependencies. Ideally, you used a virtual environment to separate your app’s dependencies from the rest of your machine’s installations. If not, that’s something to read up on and implement, but it’s way outside the scope of this post.

Freeze your dependencies like so:
```sh
pip freeze > requirements.txt
```

## /mysite/settings.py
We already changed the database settings in `settings.py` but we need to add one more thing. A `STATIC_ROOT` to tell the App Engine where to look for CSS, Javascript, Images, etc.

The `STATIC_URL` field should already be in your `settings.py`, but if it's not or if it's not configured as below, update it.

```python
# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/2.1/howto/static-files/

# Google App Engine: set static root for local static files
# https://cloud.google.com/appengine/docs/flexible/python/serving-static-files
STATIC_URL = '/static/'
STATIC_ROOT = 'static'
```

NOTE: Given that my `STATIC_URL` and `STATIC_ROOT` were already defined with the following values, I keep them, instead of following Bennett Garner:
```python
STATIC_URL = '/static/'

STATIC_ROOT = os.path.join(BASE_DIR,'static')
```

## /mysite/wsgi.py

This file should already exist and be correctly implemented. But if you’ve made changes to it, or if there are problems, here is what it should look like. Take care to change all references to mysite to whatever your Django app's naming scheme is.

```
"""
WSGI config for mysite project.

It exposes the WSGI callable as a module-level variable named ``application``.

For more information on this file, see
https://docs.djangoproject.com/en/2.1/howto/deployment/wsgi/
"""

import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings')
application = get_wsgi_application()
```

NOTE: not sure if I'm supposed to change anything - we'll see how it goes when I try to deploy

## Gather Static Files

The final step before you deploy your app is to gather all your app’s static content in a single folder that the App Engine knows it won’t have to update and can always access.

Django does this for you pretty easily:
```sh
python manage.py collectstatic
```

## Deploy the app

Hopefully after you’ve frozen the requirements, added necessary files, and collected static, your app should be ready for deployment. So try:

```sh
gcloud app deploy
```

THANK YOU BENNETT !

Since I haven't downloaded the Google Cloud SDK (event when I was told I should - but I don't like to do unecessary stuff - keep it simple !), I do that now. 

### Install Google Cloud SDK
I followed the [Google Cloud SDK documentation][gcloud-sdk-install], didn't run into any errors, yay!

`gcloud app deploy` did not work right away, I first needed to `gcloud auth login` which opened a webpage so that I could authenticate. I then needed to run `gcloud config set project [my project id]`. And finally `gcloud app deploy`. I killed it my mistake while it was uploading the files to Google Cloud Storage, but rerunning seems to have picked up where it left (hopefully).

## Errors in deployment

First error I got was due to `pip_download_wheels` not finding `pkg-resources==0.0.0` from `requirements.txt`. This is apparently due to a bug (see [stack overflow][so-pkg-resource-version-error]) and removing this line resolved the issue.


Deployment finished with success! But when I tried to open the app using `gcloud app browse` I got a `disallowed host exception`, so I added `chloe-django-blog.nw.r.appspot.com` to my `ALLOWED_HOSTS` in settings: 
```python
ALLOWED_HOSTS = ['127.0.0.1','.pythonanywhere.com', 'chloe-django-blog.nw.r.appspot.com']
```

When deploying, I get a "clean" version of Django, which results in the "decode" error I got before due to the older version of Django.. I therefore updated the version with: 
```sh
pip install Django==3.1.3
```

After this got an error related to the version of mysqlclient:
```
django.core.exceptions.ImproperlyConfigured: mysqlclient 1.4.0 or newer is required; you have 0.10.1.
```

So I tried adding this as a requirement in the `requirements.txt` file and use the following command to update: 
```sh
pip install -r requirements.txt
```
But it fails (as when I tried to install by itself - probably need mysql)

To sum up: Django version 2.1.17 result in the `decode` error, while the latest version (3.1.3) result in an incompatibility error with mysqlclient. I tried installing a few different Django versions in between, and version 3.0 works (at least locally).



### MY WEBSITE IS UP AND RUNNING !

[https://chloe-django-blog.nw.r.appspot.com/][mywebsite]


[bennett-garner]: https://medium.com/@BennettGarner/deploying-a-django-application-to-google-app-engine-f9c91a30bd35
[gcloud-sdk-install]:https://cloud.google.com/sdk/docs/install#deb
[so-pkg-resource-version-error]:https://stackoverflow.com/questions/39577984/what-is-pkg-resources-0-0-0-in-output-of-pip-freeze-command
[mywebsite]:https://chloe-django-blog.nw.r.appspot.com/