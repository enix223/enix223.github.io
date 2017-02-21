---
layout: post
categories: iOS
published: true
title: How to remove the empty spaces of UIScrollView and UIWebView  using Storyboard
---
This is a post about how to remove empty spaces when using Storyboard and auto layout to create UIScrollView and UIWebView.

## The traps

Let's say, we need drag a UIViewController to the storyboard, and add a UIWebView to the storyboard, and we also need the UIWebView taks up all the space of its parent UIView. The design will be liked that:

![storyboard-01.png]({{site.baseurl}}/media/storyboard-01.png)

And, we need to add constraints to the UIWebView, and the constraints will be like that:

![storyboard-02.png]({{site.baseurl}}/media/storyboard-02.png)

That's it, very simple, right? So, let's run it and test the, you may get the following result when you run under navigation controller and tab bar controller:

![storyboard-03.jpg]({{site.baseurl}}/media/storyboard-03.jpg)

Oh, why there is an black bottom space bar under the webview? Did I do something wrong? Yes, this is a trap 
you need to get around when using storyboard and autolayout.

## How to fix it?

It is very simple, before you adding the constraints to the UIWebiView, something you need to be paying more attention:

![storyboard-04.png]({{site.baseurl}}/media/storyboard-04.png)

After adding the contraints relative to the parent view instead of `Top layout Guide.bottom` and `Bottom Layout Guide.top`, you may get what you want when running your app again.

If you are using Webview, you also need to check `Scales page to fit`, or you still get an empty space in the bottom of th webview.

![storyboard-05.jpg]({{site.baseurl}}/media/storyboard-05.jpg)
