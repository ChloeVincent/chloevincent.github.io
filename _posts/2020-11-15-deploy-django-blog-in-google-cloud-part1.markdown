---
layout: post
title:  "Deploy Django blog in Google Cloud - Part-1"
date:   2020-11-15 16:45:00 +0100
categories: django python Gcloud Google Cloud
---

The idea of this step is that: 
- I currently have a local app running on a local database
- ultimately I want my app to run on Google Cloud on a Google Cloud MySQL database
- an intermediate step is to redirect my local app to use the Google Cloud database
- (a further step will be to deploy my app on the cloud)

# Connect my local Django app to the proxy to the MySQL database in Google Cloud

I mainly followed the blog post of [Bennett Garner][bennett-garner]. I also used [Google Cloud documentation][gcloud-proxy]


## Prerequisite on the Google Cloud dashboard

I first started by creating a free account in [Google Cloud][gcloud].
I created a project called `chloe-django-blog`.
I then went on to create a SQL database (I chose MySQL, because that's what Bennett Garner did).

To get informations on my database, I used the command line in Google Cloud Shell :
{% highlight shell %}
gcloud sql instances describe chloe-django-blog-sql
{% endhighlight %}

(replace chloe-django-blog-sql with the name of your SQL instance)

This raised the following question, because I had not enabled APIs for the project: 
{% highlight shell %}
API [sqladmin.googleapis.com] not enabled on project [XXXXXXXXXXX].
Would you like to enable and retry (this will take a few minutes)?
(y/N)? 
{% endhighlight %}

## Install the proxy on my local machine

I then continued to create the proxy that would enable me to connect to the SQL database, by running the following command at the top-level of my Django app directory (where the manage.py file is located):

```sh
wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy
```
Followed by (to make the proxy executable):
```sh
chmod +x cloud_sql_proxy
```

(this will change depending on your operating system [see Google Cloud documentation][gcloud-proxy-env])

## Connecting the proxy

I first tried this: 
```sh
./cloud_sql_proxy -instances=<INSTANCE_CONNECTION_NAME>=tcp:3306
```

With no luck: I had problems related to the authentication "google: could not find default credentials."
I tried creating users everywhere, but finally the solution for me was to create a service account (in IAM & Admin Tab > Service Accounts > Create Service Account) to which I gave many rights regardind the SQL database. I then created a key (json) for this service account which I used to connect from my local machine: 

```sh
./cloud_sql_proxy -instances=<INSTANCE_CONNECTION_NAME>=tcp:3306 -credential_file=<PATH_TO_KEY_FILE> &
```

## Connecting a minimal python app to the proxy
I first used a minimal application that did the following: 

```python
import pymysql

connection = pymysql.connect(host='127.0.0.1',
                             user='DATABASE_USER',
                             password='PASSWORD',
                             db='DATABASE_NAME')
```

I created new database users to connect. 

I could see new requests to the Cloud SQL Admin API (in the dashboard), but I always got an error that the database was "unknown".

It took me some time to realize that I was using the **SQL instance** name instead of the database name: I needed to create a database in Google Cloud Dashboard > SQL > my instance > Databases > Create Database.

One of the reason it took me a long time to figure this out, is that I got stuck trying to "restart" the proxy, as a first step to understand why the database was not recognized (as I said, it was because it didn't exist and I was taking the SQL instance for a database). 
It turned out that the proxy running in the background. I used `ps` to check which process were descendant from the shell session, and `telnet 127.0.0.1 3306` to check what was happening on this port (3306). To get the proxy process back on the foreground (and kill it properly), I used `fg` (as in foreground). `jobs`, as `ps`, shows the processes from the bash, it gives their job id, which allows to `fg` one or the other. To launch an app in the background, add an "&" at the end of the command line, or if it is launch in foreground, use `ctrl+D` to stop the job and `bg` to put it in the background.

Finally my minimal application could connect.

## Connecting my Django app to the proxy

The last step of this part one is to connect my Django application to the proxy.

In mysite/settings.py, I replaced:
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

with: 
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'HOST': '127.0.0.1',
        'PORT': '3306',
        'NAME': 'mydatabasename',
        'USER': 'myusername',
        'PASSWORD': 'mypassword',
    }
}
```

I tried to run `python manage.py makemigrations`, but got an error
```
django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module.
Did you install mysqlclient?
```


After a [quick internet search][so-pymysql-error], I did install `pymysql`: 
```sh
pip install pymysql
```
And I added the following lines to `mysite/__init__.py` (which was empty):

```python
import pymysql

pymysql.install_as_MySQLdb()
```

I then got the following error:

```
AttributeError: 'str' object has no attribute 'decode'
```

A [quick internet search][so-decode-error] told me that this was due to my version of Django being too old, instead of reinstalling a newer version, I replace `query = query.decode(errors='replace')` by `query = errors='replace'` in `django\db\backends\mysql\operations.py`

`makemigrations` did not raise any errors anymore but it said that no changes were detected. So I needed to remove the existing migrations from the `migrations` folder in my app.

Last step was to apply those migrations to the database in Google Cloud: 
```
python manage.py makemigrations
python manage.py migrate
``` 

## Extra final step
Bennett Garner suggest to use the following in settings.py. This allows to manage the production vs development settings:

```python
# [START db_setup]
if os.getenv('GAE_APPLICATION', None):
    # Running on production App Engine, so connect to Google Cloud SQL using
    # the unix socket at /cloudsql/<your-cloudsql-connection string>
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'HOST': '/cloudsql/[YOUR-CONNECTION-NAME]',
            'USER': '[YOUR-USERNAME]',
            'PASSWORD': '[YOUR-PASSWORD]',
            'NAME': '[YOUR-DATABASE]',
        }
    }

else:
    # Running locally so connect to either a local MySQL instance or connect 
    # to Cloud SQL via the proxy.  To start the proxy via command line: 
    #    $ cloud_sql_proxy -instances=[INSTANCE_CONNECTION_NAME]=tcp:3306 
    # See https://cloud.google.com/sql/docs/mysql-connect-proxy
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'HOST': '127.0.0.1',
            'PORT': '3306',
            'NAME': '[YOUR-DATABASE]',
            'USER': '[YOUR-USERNAME]',
            'PASSWORD': '[YOUR-PASSWORD]',
        }
    }
# [END db_setup]
```

## Extra bonus step
Create a Django admin superuser to be able to manage the application, and in my case add new posts.
```sh
python manage.py createsuperuser
```


[bennett-garner]: https://medium.com/@BennettGarner/deploying-a-django-application-to-google-app-engine-f9c91a30bd35
[gcloud-proxy]:https://cloud.google.com/sql/docs/mysql/connect-external-app#proxy
[gcloud]:https://cloud.google.com
[gcloud-proxy-env]:https://cloud.google.com/sql/docs/mysql/connect-external-app#install
[so-decode-error]:https://stackoverflow.com/questions/56820895/migrations-error-in-django-2-attributeerror-str-object-has-no-attribute-dec
[so-pymysql-error]:https://stackoverflow.com/questions/46902357/error-loading-mysqldb-module-did-you-install-mysqlclient-or-mysql-python