---
layout: post
title:  "Deploy Django blog in Google Cloud - Part-3"
date:   2020-11-16 11:00:00 +0100
categories: django python Gcloud Google Cloud
---


My website is now up and running on [chloe-django-blog.nw.r.appspot.com][mywebsite].

I would now like to deploy from Github. But first I need to resolve some issues:


## Update .gcloudignore
On a related note, when I deployed to Google Cloud using `gcloud app deploy`, *many* files were uploaded. I need to change the `.gcloudignore` file to include entries that are in the .gitignore:
```
*.pyc
*~
/.vscode
__pycache__
myvenv
db.sqlite3
/static/admin #we need /static/css
.DS_Store
```

When I deploy, the changes are taken into account (you can make sure of that by ignoring `/static/css`, the style is reset).


## Update chloe-django-blog.appspot.com

At the moment only chloe-django-blog.nw.r.appspot.com is updated when I deploy the app:
```
target url:      [https://chloe-django-blog.nw.r.appspot.com]
```
Actually it is well updated, I only needed to add `chloe-django-blog.appspot.com` to the `ALLOWED_HOSTS` in settings.py

## TO BE CONTINUED
Deploying from Github will be the next step and I will change this post accordingly.








[bennett-garner]: https://medium.com/@BennettGarner/deploying-a-django-application-to-google-app-engine-f9c91a30bd35
[gcloud-sdk-install]:https://cloud.google.com/sdk/docs/install#deb
[so-pkg-resource-version-error]:https://stackoverflow.com/questions/39577984/what-is-pkg-resources-0-0-0-in-output-of-pip-freeze-command
[mywebsite]:https://chloe-django-blog.nw.r.appspot.com/