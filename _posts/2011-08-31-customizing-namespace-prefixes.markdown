---
layout: post
title:  "Customizing Namespace Prefixes"
date:   2011-08-31 15:14:00
---

Changing the namespace prefix on XML elements sent to a web service seemed like a simple task. I wasn’t intimately familiar with the granular customization details of JAXB or JAX-WS; but how hard could it be? After spending over a week with XSL, JAX-WS filters, endorsed packages, classpath ordering, and other fun hacks; I’ve developed a bit of hatred for JAXB. I thought I’d document my findings and hopefully save some other poor soul from its torment.

Start by enabling logging:
{% highlight sh %}
-Dcom.sun.xml.internal.ws.transport.http.client.HttpTransportPipe.dump=true
{% endhighlight %}

Flipping this property on will print all of the sent and received SOAP messages to standard output.

Next, annotate the namespace’s generated package with a prefix directive:
{% highlight java %}
@javax.xml.bind.annotation.XmlSchema(
    namespace = "http://tempuri.org/address",
    elementFormDefault = javax.xml.bind.annotation.XmlNsForm.QUALIFIED,
    xmlns = {
        @javax.xml.bind.annotation.XmlNs(
            prefix = "addr",
            namespaceURI = "http://tempuri.org/address")
    })
package org.tempuri.address;
{% endhighlight %}

This is the package-info.java file generated by wsimport. Ideally, a bindings file should have wsimport create the XmlNs annotation but I couldn’t find any documentation on how to do that. It’ll be overwritten each time but there’s a hack for that. (More details below.)

Strangely, the prefix would not work for me at this point and I had to upgrade to JAX-WS 2.2.1. This, in turn, forced me to move the JAX-WS 2.2.1 libraries higher up in the order of my Eclipse project classpath. My deployment package also now required the java.endorsed.dirs JVM property to be set and pointed at the 2.2.1 jars.

With the namespace prefix now working and no documentation on generating an XmlNs annotation, the package-info changes need to be automated. For my hacked solution, I moved package-info.java into my static source tree and added an Ant task to remove the file after generation:
{% highlight xml %}
<wsimport ... />
<delete file="${gen-src}/package-info.java" />
{% endhighlight %}

This produces a new issue. The wsimport task will generate and compile the package-info. When a full build executes later, the class file is already up to date and will be skipped. There are two solutions to this:

1. Add a delete task to remove the class file as well.
1. Add the -npa option to the wsimport task to suppress the generation of package-info files:

{% highlight xml %}
<wsimport ...>
  <xjcarg value="-npa" />
</wsimport>
<delete file="${gen-src}/package-info.java" />
{% endhighlight %}

It’s awful, it’s hideous, it keeps me awake at night, but it works. If you’re reading this article looking for help and now you’re just shaking your head in disgust, try [Harald Wellmann’s blog post][1]. He does a great job of presenting two solutions I wish I had found much earlier in my research.

[1]: http://hwellmann.blogspot.com/2011/03/jaxb-marshalling-with-custom-namespace.html
