---
layout: post
title: "Dynamic Java Classpath in UNIX"
date:  2011-02-01 10:12:00
---

I've been playing "guess the dependency" with some Axis2 Java code all morning.  After encountering three or four NoClassDefFound exceptions, I'm rewriting my java command to dynamically list all of the jar libraries on the classpath.  Here's what I compiled from browsing some forums and blogs:

{% highlight sh %}
find /opt/axis2/lib -type f -name *.jar | sed ':a;N;$!ba;s/\n/:/g'
{% endhighlight %}

Sources:  
<http://www.unix.com/unix-dummies-questions-answers/41019-listing-files-full-path.html>
<http://bashrules.blogspot.com/2005/08/removing-newlines-from-file-using-sed.html>
