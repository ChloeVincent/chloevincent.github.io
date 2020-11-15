---
layout: post
title:  "Clear microsoft teams credentials"
date:   2020-10-22 19:15:00 +0100
categories: linux debian microsoft teams
---

I recently downloaded Microsoft teams for Linux to accomodate job interviewers. 
I first logged in with my university credential to check if everything was working fine. 
However, when I tried to log with my "normal" address, the one I was supposed to use for the job interview I could not log out of the university profile. 
Even after removing the package, I would still get the university credential login page. 
In this article I quickly explain the process I followed

# Install Microsoft Teams
This is pretty straightforward: I downloaded the debian package and used the following commands: 

{% highlight bash %}
cd Downloads/
sudo apt install ./teams_1.3.00.25560_amd64.deb
{% endhighlight %}

# Uninstall using remove and purge

The classic way to uninstall a package is remove: 

{% highlight bash %}
sudo apt remove teams
{% endhighlight %}

However since this was not enough to get rid of the university credential, I reinstalled and tried purge: 

{% highlight bash %}
sudo apt purge teams
{% endhighlight %}

# Solution: remove the cache files
I found out, by searching for all the occurences of "teams", that there are folders that are not cleared when uninstalling: in home, look for the ".config" folder which contains one "Microsoft" folder and one "teams" folder. 
Getting rid of both resolved the issue and I was able to connect with other credentials!