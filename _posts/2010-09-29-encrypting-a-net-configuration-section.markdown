---
layout: post
title:  "Encrypting a .NET Configuration Section"
date:   2010-09-29 12:39:00
---

If you don't want your database connection string information hanging around in the configuration file as plain text, you can encrypt it with the aspnet_regiis utility.  Here's how:

Unfortunately the utility is hard coded to modify web.config, so you'll have to rename your file first.

{% highlight bat %}
move MyApplication.exe.config web.config
{% endhighlight %}

Run the aspnet_regiis utility and tell it you want to encrypt the connectionStrings configuration section:

{% highlight bat %}
aspnet_regiis -pef connectionStrings . -prov DataProtectionConfigurationProvider
{% endhighlight %}

Restore your original filename:

{% highlight bat %}
move web.config MyApplication.exe.config
{% endhighlight %}

If you're getting a "command not found" error, you'll have to add the framework binaries to your path.  The aspnet_regiis utility is usually located here:

    C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727
