---
layout: post
categories: web
tags: 
  - python
published: true
title: "iCQ-Web Development note"
---

## Problem set (问题集锦)
----

### ImproperlyConfigured at /path/, The included urlconf app.urls doesn't have any patterns in it
先看看url.py
{% highlight python %}
urlpatterns = patterns('',
	 ....
    url(r'^user/forget/$', SSUserForgetView.as_view(), name='user-forget'),
    ....
)
{% endhighlight %}

再看view.py代码：
{% highlight python %}
class SSUserForgetView(TemplateView):
    success_url = reverse('user-forget')
{% endhighlight %}

当运行python manage.py runserver，就会出现如下错误：
ImproperlyConfigured at /path/, The included urlconf app.urls doesn't have any patterns in it

错误的原因，我想应该是url.py加载的时候，会import SSUserForgetView，而SSUserForgetView的success_url就在此时初始化，而因为url还没初始化完成，无法通过reverse找到对应的url，所以就出现了以上的错误，所以，解决办法是，通过get_success_url方法，返回对应的success_url，修改后的代码为：
{% highlight python %}
class SSUserForgetView(TemplateView):
    def get_success_url(self):
        return reverse('user-forget')
{% endhighlight %}

这样，这个问题就可以完美解决。
