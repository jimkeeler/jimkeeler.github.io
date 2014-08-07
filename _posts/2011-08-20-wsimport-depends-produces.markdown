---
layout: post
title:  "wsimport depends/produces"
date:   2011-08-20 18:13:00
---

Working with the `wsimport` Ant task, you may come across this warning:

>Consider using / so that wsimport won't do unnecessary compilation

For whatever reason these nested elements were left out of the [documentation][1].  I borrowed some wording from the [JAXB documentation][2] and wrote this amendment:

__depends__  
Files specified with this nested element will be taken into account when the task does a modification date check.  For proper syntax, see [`<fileset>`][3].

__produces__  
Files specified with this nested element will be taken into account when the task does a modification date check.  For proper syntax, see [`<fileset>`][3].
  
Example:
{% highlight xml %}
<wsimport
    wsdl="${local-wsdl-path}"
    destdir="${build.dir}"
    sourcedestdir="${src.generated.dir}">
    <depends file="${local-wsdl-path}"/>
    <produces dir="${src.generated.dir}"/>
</wsimport>
{% endhighlight %}

This wsimport example specifies a depends/produces relationship between a local WSDL file and the generated source directory. If the WSDL file has a modification date more recent than the generated directory, the wsimport task will regenerate and recompile the source code.

[1]: http://jax-ws.java.net/2.2.1/docs/wsimportant.html
[2]: http://download.oracle.com/docs/cd/E17802_01/webservices/webservices/docs/1.6/jaxb/ant.html
[3]: http://ant.apache.org/manual/Types/fileset.html
