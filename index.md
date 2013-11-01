---
layout: page
title: Welcome 
tagline: 在你想要放弃的那一刻，想想为什么当初坚持走到了这里
---
{% include JB/setup %}

#欢迎

我是`Eupiccolo`，是一名喜爱编程的少年，崇尚算法，我是一个`逗比`。

{% highlight c++ linenos %}
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

