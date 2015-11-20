---
layout: post
title: DoS using Hash Collisions in ASP .net
uname: dos-using-hash-collisions-in-asp.net
date: 2012-01-02 05:29
author: Sergio Garcia
comments: true
categories: en, ASP .net, asp .net, C#, C#, Desenvolvimento, DoS, hash collision attack, performance, Sem categoria, Tecnologia, vulnerability, zero day flow, zero day vulnerability
---

This weekend in the [28C3][] Alexander "alech" Klink and Julian "zeri" Wälde presented [Effective Denial of Service attacks against web application platforms][paper] which can make a server with 99% CPU usage using very low resources.

In my tests, a simple HTTP post request, with about 100k collided hashes (about 1,2MB) can take the one server thread for hours using 99% of CPU, as show in the image below.

![Hash Collision](http://blog.ginx.com.br/wp-content/uploads/2012/01/hash-collision-300x176.png)

If we use more threads for the attack, the server will be easily be overloaded. In the study, alech and zeri concluded that a 30 kbits/s connection will be sufficient to keep a Core2 core using 99% CPU forever in a server running ASP .net. And a 1 Gbit/s connection will can take down about ~30k Core2 cores. A attacker can easily take down a small datacenter using a broadband connection.

### The Attack

The vulnerability occurs due to how ASP .net handles Forms, Cookies, Session and Query Strings. All of these collections are stored in a [NameValueCollection][], which internally use the [StringComparer.OrdinalIgnoreCase][StringComparer_OrdinalIgnoreCase] to get the hash of the strings.

The [GetHashCode][StringComparer_GetHashCode] function can return the same value to diferent strings and as the method used to calculate the hash is know and can be predicted. A attacker can generate a big collection of collided hashes to post in any ASP .net page. The server will try to build the [Request][HttpContext_Request] object, and because the keys in the Forms property will collide, the  [NameValueCollection][] will take very long to be created, and the server will keep running using a lot of CPU to compare all the values.

In my test, using IIS 7.5 and ASP .net 4.0, the server took more than a hour to stop using 99% in the affected thread. The server can't stop the processing in the default time (90s) because the request was affected at the start of the ASP .net Pipeline.

### The Solution

A HttpModule can inspect the request before it was passed to ASP .net and it can be be stop in case of the attack is detected.

I started creating this module, which is funcional but not intended for production use. For now, it is for study only, as this flaw is very recent.

The HttpModule and its source can be download from [GitHub][HashCollisionDetector].

Any comments to help improved and ensure production quality to this module will be appreciated.


[28C3]: http://events.ccc.de/congress/2011/Fahrplan/ "28th Chaos Communication Congress"
[paper]: http://events.ccc.de/congress/2011/Fahrplan/events/4680.en.html "Effective Denial of Service attacks against web application platforms"
[NameValueCollection]: http://msdn.microsoft.com/en-us/library/system.collections.specialized.namevaluecollection.aspx
[StringComparer_OrdinalIgnoreCase]: http://msdn.microsoft.com/en-us/library/system.stringcomparer.ordinalignorecase.aspx
[StringComparer_GetHashCode]: http://msdn.microsoft.com/en-us/library/a04941bd.aspx "StringComparer.GetHashCode Method"
[HttpContext_Request]: http://msdn.microsoft.com/en-us/library/system.web.httpcontext.request.aspx "HttpContext.Request Property"
[HashCollisionDetector]: https://github.com/sergio-garcia/HashCollisionDetector
