---
layout: post
title: Sample Search Portlets
categories: [search]
---

A while back I wrote [this post][1] detailing my struggle to find a quick and dirty way of displaying paginated table data in the portal (or anywhere, for that matter). I ended up settling on the method I [found at Spartan Java][2].

While I enjoyed the series of articles, I still ended up having to review a [jQuery primer][3], the [DWR documentation][4] and the [iBATIS user guide][5]. I would have liked to have been able to download the finished application. On top of that there were the usual struggles with portletizing the code, and scrambling to jam a bunch of features into the finished web application before launch.

Thus, when I got asked to provide some “development best practices” portal code, I was in a bit of a bind. I wanted to show off the WCI portal's ability to consume and use a variety of frameworks, but my original source was… pretty crappy. In the end I went back and rewrote a very simple sample application from scratch (including documentation) using the methods described above.

Hopefully I can save you some time and effort in similar endeavors. Without further ado here is a link to a [sample database search application][6] using the [Oracle HR sample data][7].

It's nothing fancy (yes, there are even possible SQL injection vulnerabilities), but if you want a simple example of jQuery, iBATIS, Java and the portal, this is definitely something to check out. All you should have to do is import the war file into eclipse and read the index.html documentation.

 [1]: /jquery/dwr/2009/05/11/dwrjquerytables.html
 [2]: http://www.spartanjava.com/2008/paginated-lists-made-really-easy-part-1-of-2-front-end/
 [3]: http://dotnetslackers.com/articles/ajax/JQuery-Primer-Part-1.aspx
 [4]: http://directwebremoting.org/dwr/documentation.html
 [5]: http://svn.apache.org/repos/asf/ibatis/java/ibatis-3/trunk/doc/en/iBATIS-3-User-Guide.pdf
 [6]: /assets/searchsample.war
 [7]: http://www.oracle.com/technology/obe/obe1013jdev/common/OBEConnection.htm  