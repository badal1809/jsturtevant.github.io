---
layout: fwtpost
title: ".NET DNS Class"
date: "2015-03-01"
categories:
- csharp
- .net framework tour
---

Continuing with the theme of the last post, which covered the [IPAddress class](/posts/net-ipaddress-class/), this week we will take a look at the [DNS class](https://msdn.microsoft.com/en-us/library/System.Net.Dns(v=vs.110).aspx).  The ```DNS``` class also lives in the ```System.Net``` namespace  and helps simplify working with [DNS servers](http://en.wikipedia.org/wiki/Name_server) to resolve IP addresses and [hostnames](http://en.wikipedia.org/wiki/Hostname).  Whether you need to know the IP Address from the hostname, or you have a hostname and you need to know the IP Address, the ```DNS``` class will help make DNS resolution simple.

To retrieve IP addresses via the hostname:

{% highlight csharp %}
IPAddress[] ipAddresses = Dns.GetHostAddresses("jamessturtevant.com");

Assert.AreEqual("162.255.119.254", ipAddresses.First().ToString());
{% endhighlight %}

To get the host namevia the IP address:
{% highlight csharp %}
// May fail if IP changes
IPHostEntry hostEntry = Dns.GetHostEntry("192.30.252.131");

Assert.AreEqual("github.com", hostEntry.HostName);
{% endhighlight %}

## More Information
The above functions show how to make synchronous calls but the ```DNS``` class has recently been given asynchronous support.  You can find the asynchronous version and more information about the ```DNS``` class on the [MSDN page](https://msdn.microsoft.com/en-us/library/System.Net.Dns(v=vs.110).aspx).  You can also find runnable examples on my [.NET Framework Tour](https://github.com/jsturtevant/DotNetTour) GitHub repository.
