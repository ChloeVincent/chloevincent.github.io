---
layout: post
title:  "Use markdown in posts"
date:   2020-11-16 20:00:00 +0100
categories: django python markdown
---

I want to be able to use markdown in the posts I am writing. 
I follow [this stackoverflow response][so-markdown].

I first install markdown deux and update `requirements.txt`
```sh
pip install django-markdown-deux
pip freeze > requirements.txt
```
(remember to remove the `pkg-resources==0.0.0`)

Add the following in settings.py:
```python
INSTALLED_APPS = [
	...
    'markdown_deux',
]
```

And finally, wherever I use the text from my posts, I need to load the `markdown_deux_tags` after the 'extends' and add the tag `markdown`:
```html
{% extends 'blog/base.html' %}
{% load markdown_deux_tags %}

...

<p>{{ post.text|markdown }}</p>


```



[mywebsite]:https://chloe-django-blog.appspot.com/
[myregional-website]:https://chloe-django-blog.nw.r.appspot.com/
[so-markdown]:https://stackoverflow.com/questions/23031406/how-do-i-implement-markdown-in-django-1-6-app