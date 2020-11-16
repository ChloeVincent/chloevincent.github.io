---
layout: post
title:  "Update Django blog using pykew"
date:   2020-11-16 18:00:00 +0100
categories: django python Gcloud Google Cloud
---


I want to display more interesting results than the number of occurences found in the database for a given query. 

How to understand what attributes are accessible:

`result.__dict__` gives me a dictionnary with the attributes of result and their value. I loop in the result to print all the attributes: 
```python
result = ipni.search(self.query)
print("THE RESULT IS ")
print(result.__dict__)
print(result._query)

print("START LOOP")
for r in result:
	print(r)
	print("+++")
	print(r['name'])
	print("***")

	print("keys")
	for k in r.keys():
		print("{}: {}".format(k,r[k]))
```

I will add 'name', 'family', 'genus', 'species' and 'distribution' to the information displayed. 
This means I have to upload the database using (after adding new class in `models.py`):
```sh
python manage.py makemigrations
```
and
```sh
python manage.py migrate
```

Note: I added a tiny bit of css to frame the occurences in the response. In order to upload the change to Google Cloud I need run: 
```sh
python manage.py collectstatic
```
The regional version of my website was updated immediately ([the one with nw.r][myregional-website]) but I needed to wait a few minutes for the global one to update ([chloe-django-blog.appspot.com][mywebsite]).



[mywebsite]:https://chloe-django-blog.appspot.com/
[myregional-website]:https://chloe-django-blog.nw.r.appspot.com/