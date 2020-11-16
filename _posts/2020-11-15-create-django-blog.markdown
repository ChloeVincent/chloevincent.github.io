---
layout: post
title:  "Create Django blog and deployment on PythonAnywhere"
date:   2020-11-15 16:15:00 +0100
categories: django python pythonanywhere
---

I simply followed [Django girls tutorial][django-girls]. 

It involved creating a very simple blog, using a SQlite database. The basic website display a list of posts, that can be added and edited if the user is authenticated. 

I deployed it on [Python Anywhere][pythonanywhere]. It is accessible at [chloevincent.pythonanywhere.com][cvpa]

I then wanted to customize my website and access Kew collection thanks to the python package [pykew][pykew]. After some CSS changes, I added a button that open a search form that uses the Kew API, and returns informations regarding the species searched (for instance 'monstera deliciosa' or 'poa annua'). 

The website is not secured, nor is it robust. 
The main current issue is that the free account for pythonanywhere does not allow access to external databases. 
The website works on my local environment but the option to look for species information raises an exception when the website is deployed on PythonAnywhere

The next step is deploying the app on Google Cloud, to work around this issue and learn about a new tool. My Google Cloud experience is described in a next article. 

## Edit: my app runs on pythonanywhere now

Giles from the support team at pythonanywhere added pykew to the API whitelist and now the app works fine. 
I have not uploaded the latest version of the app that communicates with Google Cloud SQl, because that would mean pushing passwords in Github.

[pythonanywhere]:https://pythonanywhere.com
[pykew]:https://pypi.org/project/pykew/
[django-girls]:https://tutorial.djangogirls.org/en/
[cvpa]:https://chloevincent.pythonanywhere.com