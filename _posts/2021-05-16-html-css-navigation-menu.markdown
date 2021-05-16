---
layout: post
title:  "HTML, CSS: Navigation menu"
date:   2021-05-16 14:00:00 +0100
categories: html css manu
---


In this post I write about how to create a nice menu for my website.
This is based on a former website I created following a "site du zero" html tutorial which I don't have the reference for anymore since it was 8 years ago. 



``` html
<nav>
    <ul class="menu">
        <li><a href="index.html">ACCUEIL</a>
            <ul class="sousmenu">
              <li> <a href="sub-menu-page.html"> Sub menu</a></li>
            </ul>
        </li>
        <li><a href="other-page.html">OTHER PAGE</a>
            <ul class="sousmenu">
              <li> <a href="sub-other.html"> First other</a></li>
              <li> <a href="second-other.pdf"> PDF other </a></li>
            </ul>
        </li>
        <li><a >DO&nbspNOT&nbspSPLIT</a>
            <ul class="sousmenu">
              <li> <a href="only-sub-menu.ods"> To download </a></li>
            </ul>
        </li>
     </ul>
</nav>
```

The corresponding CSS is the following:
```css
/* Navigation */
nav
{
  display: inline-block;
  border-bottom: 2px solid rgba(0,0,0,0.5);
  width: 900px;
  margin-bottom: 20px;
  height: 72px;
}
nav ul ul
{
  display: none;
  padding: 0;
  position: absolute; 
  top: 100%;
  z-index:100;
}

nav ul li:hover > ul 
{
  display: block;
}

nav ul
{
  list-style: none;
  position: relative;
  display: inline-table;
}


nav ul li 
{
  float: left;
}


nav ul li a 
{
  display: block; 
  padding: 5px 20px;
  color: #181818; /* 181818 */
  text-decoration: none;
  font-size: 1.5em;
  opacity: 1;
  background: url('images/fond_bleu.png');
}

nav ul li ul li a
{
  background: url('images/fond_bleu.png');
  border: 1px solid rgb(0,92,39); /*#760001;*/
}

nav a:hover
{
    color: rgb(0,92,39);
    border-bottom: 3px solid rgb(0,92,39);
} 
  
nav ul ul li 
{
  float: none; 
  position: relative;
}

```

The background image is necessary so that it covers what is below when the menu opens.

The `z-index` property put the menu on the foreground compared to the website page below. 
It is especially useful when the page of the website contains a div with its position property as relative.



[mjt-footer]:https://matthewjamestaylor.com/bottom-footer