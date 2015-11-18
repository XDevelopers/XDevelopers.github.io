---
layout: post
title: DoS using Hash Collisions in ASP .net
uname: dos-using-hash-collisins-in-asp.net
date: 2012-01-02 05:29
author: Sergio Garcia
comments: true
categories: en, ASP .net, asp .net, C#, C#, Desenvolvimento, DoS, hash collision attack, performance, Sem categoria, Tecnologia, vulnerability, zero day flow, zero day vulnerability
---
<p>This weekend in the <a href="http://events.ccc.de/congress/2011/Fahrplan/" title="28th Chaos Communication Congress" target="_blank">28C3</a> Alexander "alech" Klink and Julian "zeri" Wälde presented <a href="http://events.ccc.de/congress/2011/Fahrplan/events/4680.en.html" title="Effective Denial of Service attacks against web application platforms" target="_blank">Effective Denial of Service attacks against web application platforms</a> which can make a server with 99% CPU usage using very low resources.</p>

<!--more-->

<p>In my tests, a simple HTTP post request, with about 100k collided hashes (about 1,2MB) can take the one server thread for hours using 99% of CPU, as show in the image below.</p>

<a href="http://blog.ginx.com.br/wp-content/uploads/2012/01/hash-collision.png"><img src="http://blog.ginx.com.br/wp-content/uploads/2012/01/hash-collision-300x176.png" alt="" title="hash-collision" width="300" height="176" class="alignnone size-medium wp-image-87" /></a>

<p>If we use more threads for the attack, the server will be easily be overloaded. In the study, alech and zeri concluded that a 30 kbits/s connection will be sufficient to keep a Core2 core using 99% CPU forever in a server running ASP .net. And a 1 Gbit/s connection will can take down about ~30k Core2 cores. A attacker can easily take down a small datacenter using a broadband connection.</p>

<h3>The Attack</h3>

<p>The vulnerability occurs due to how ASP .net handles Forms, Cookies, Session and Query Strings. All of these collections are stored in a <a href="http://msdn.microsoft.com/en-us/library/system.collections.specialized.namevaluecollection.aspx" title="NameValueCollection" target="_blank">NameValueCollection</a>, which internally use the <a href="http://msdn.microsoft.com/en-us/library/system.stringcomparer.ordinalignorecase.aspx" title="StringComparer.OrdinalIgnoreCase" target="_blank">StringComparer.OrdinalIgnoreCase</a> to get the hash of the strings.</p>

<p>The <a href="http://msdn.microsoft.com/en-us/library/a04941bd.aspx" title="StringComparer.GetHashCode Method " target="_blank">GetHashCode</a> function can return the same value to diferent strings and as the method used to calculate the hash is know and can be predicted. A attacker can generate a big collection of collided hashes to post in any ASP .net page. The server will try to build the <a href="http://msdn.microsoft.com/en-us/library/system.web.httpcontext.request.aspx" title="HttpContext.Request Property" target="_blank">Request</a> object, and because the keys in the Forms property will collide, the <a href="http://msdn.microsoft.com/en-us/library/system.collections.specialized.namevaluecollection.aspx" title="NameValueCollection" target="_blank">NameValueCollection</a> will take very long to be created, and the server will keep running using a lot of CPU to compare all the values.</p>

<p>In my test, using IIS 7.5 and ASP .net 4.0, the server took more than a hour to stop using 99% in the affected thread. The server can't stop the processing in the default time (90s) because the request was affected at the start of the ASP .net Pipeline.</p>

<h3>The Solution</h3>

<p>A HttpModule can inspect the request before it was passed to ASP .net and it can be be stop in case of the attack is detected.</p>

<p>A started creating this module, which is funcional but not intended for production use. For now, it is for study only, as this flaw is very recent.</p>

<p>The HttpModule and its source can be download from <a href="https://github.com/ginx/HashCollisionDetector" title="HashCollisionDetector" target="_blank">GitHub</a>.

<p>Any comments to help improved and ensure production quality to this module will be appreciated.</p>
