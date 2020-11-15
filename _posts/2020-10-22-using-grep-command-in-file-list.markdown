---
layout: post
title:  "Using grep command in list of files"
date:   2020-10-22 19:15:00 +0100
categories: bash corpus
---

The grep command look inside a file and returns the line that contains the word
searched. Here, it will return all lines containing *genre* that are found in a CHA file.

{% highlight bash %}
grep -i ’genre’ *.cha
{% endhighlight %}

Adding -C 3 > output.txt to the command will output three lines of context
before and after, and will save the result in a file called output.txt. More complex
request can be done. For instance, the following will output the number of time
‘SP11’ was found in each CHA file in the current folder.

{% highlight bash %}
for i in *.cha; do grep -i ’SP11’ $i | echo $i ‘wc -l‘; done

{% endhighlight %}


