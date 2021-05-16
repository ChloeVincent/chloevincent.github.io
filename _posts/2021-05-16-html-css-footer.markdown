---
layout: post
title:  "HTML, CSS: Keep footer at the bottom of a page"
date:   2021-05-16 12:00:00 +0100
categories: html css footer
---


In this post I write about how to keep the footer of a website at the bottom of a page.


I followed [Matthew James Taylor website][mjt-footer] to keep the footer of my website at the bottom of the page, even when there was not a lot of content.
I modified the margin and padding for the purpose of my website.

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
        One line simple string.<br />
        <!-- The following line allows to check if this works properly with a page with longer content -->
        <!--<br /><br /><br /><br /><br /><br /><br /><br /><br /><br />v<br /><br /><br /><br /><br /><br /><br /><br /><br /><br />v<br /><br /><br /><br /><br /><br /><br /><br /><br /><br />v<br /><br /><br /><br /><br /><br /><br /><br /><br /><br />v<br /> -->
        Two line simple string example<br />
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

The corresponding CSS is the following:
```css
body, html
{
    height: 100%;
}

#bloc_body{
    min-height:100%;
    position:relative;
    width: 900px;
    margin-left: auto;
    margin-right: auto;
}

#bloc_content {
    padding:10px;
    padding-bottom:90px;   /* Height of the footer */
}

#bloc_footer {
   position:absolute;
   bottom:0;
   width:100%;
   height:60px;   /* Height of the footer */
}
```



[mjt-footer]:https://matthewjamestaylor.com/bottom-footer