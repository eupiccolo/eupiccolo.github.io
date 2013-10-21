---
layout: page
title: Welcome 
tagline: 做一名有态度的逗比
---
{% include JB/setup %}

#欢迎

我是`Eupiccolo`，是一名喜爱编程的少年，崇尚算法，梦想是成为一名伟大的`逗比`。

{% highlight c++ %}
#include <stdio.h>

int main() {
	printf("Welcome to my blog!");
	return 0;
}
{% endhighlight %}


#最新文章
<pre>
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
</pre>

