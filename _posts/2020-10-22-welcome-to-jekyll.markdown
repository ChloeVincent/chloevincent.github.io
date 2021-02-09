---
layout: post
title:  "Welcome to Jekyll!"
date:   2020-10-22 16:17:49 +0100
categories: jekyll update
---
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

# Edit for tips and tricks

#### Building site locally for testing
To build your site locally for testing, you will need to run `bundle exec jekyll serve`. 
The blog will be accessible at [localhost:4000][local-address].

#### Collapsible code snippets
It is supposedly very easy to add collapsible sections in markdown according to [this page][collapsible-markdown]. 
However, in my case, it did not render the markdown inside of the html tags correctly, as in [here][collapse-render].
To resolve, I installed the CommonMarkGhPages by adding `gem 'jekyll-commonmark-ghpages'` to the `jekyll_plugins group` in my `Gemfile`:
```
# jekyll-commonmark-ghpages added to collapse code snippet
group :jekyll_plugins do
  ...
  gem 'jekyll-commonmark-ghpages'
end
```

I then run `bundle install` to install the dependencies.


The collapsible section are written as follow
{% highlight html %}

<details><summary>

`code snippet need newline before` (click to expand)
</summary>
<p>

```golang
func main() {
    code snippet in go
}
```
</p>
</details>
{% endhighlight %}


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[local-address]: localhost:4000

[collapsible-markdown]: https://gist.github.com/joyrexus/16041f2426450e73f5df9391f7f7ae5f
[collapse-render]: https://stackoverflow.com/questions/52944720/content-of-collapsible-sections-detailssummary-renders-markdown-in-gith