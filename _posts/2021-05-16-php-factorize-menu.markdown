---
layout: post
title:  "PHP: How to factorize menu"
date:   2021-05-16 13:00:00 +0100
categories: html php factorize menu
---


In this post I write about how to factorize parts of my website so that I avoid copy-paste errors and inconsistencies. 
The header and footer will not need to be changed, only the content part will be different.

The PHP file will simply call the different HTML parts:
```php
<?php 
include 'header.html';
include 'content.html';
include 'footer.html';
?>
```

And the html files will be as follow:

`header.html`:

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <link rel="stylesheet" href="mystyle.css" />
    <title>My website title</title>
  </head>

  <body>
    <div id=bloc_body>
    
      <div id="bloc_header">
        <header>
          <h1>My website</h1>
        </header>
      </div>

      <div id="bloc_page">
       
```

`content.html`:
```html
One line simple string.<br />
Two line simple string example<br />
```

`footer.html`:
```html
      </div>
	
      <div id="bloc_footer">	
        <footer>
          Si vous avez des questions n'hésitez pas à me contacter. 
        </footer>
      </div>
    </div>
  </body>
</html>
```
