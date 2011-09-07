---
layout: post
title: Apache Directory
categories: [ldap]
---
In one of my [previous posts][1] I mentioned [Apache Directory Studio][2] as a great way to view LDAP directories, but the entire [Apache Directory][3] project really deserves its own post (and here it is).

First of all, Apache has implemented a fully featured directory browser ([Apache Directory Studio][2]) and a fully featured LDAP directory ([Apache Directory Server][4]). Both of these projects are entirely Java based. Studio is an eclipse based directory browser with all the bells and whistles. [JXplorer][5] is nice, but not as powerful or as easy to use.

However, the real power of ApacheDS is the server. It's entirely java based and is [available for a variety of platforms, including Windows][6]. I have yet to find an easier platform to install and use on a variety of operating systems (especially Windows). Rather than trying to build a confusing [OpenLDAP][7] implementation, you can simply download, install, and start ApacheDS in 5 to 10 minutes. 

And, oh by the way, you can also [embed and manipulate ApacheDS in your own applications][8], since its written entirely in Java and the source code is freely available.

So, if you are looking for an easy, free, directory implementation for your next proof of concept, demo, or unit test, look no further than ApacheDS.

 [1]: http://hross.net/blog/2009/09/windows-tools.html
 [2]: http://directory.apache.org/studio/
 [3]: http://directory.apache.org/
 [4]: http://directory.apache.org/apacheds/1.5/
 [5]: http://jxplorer.org/
 [6]: http://directory.apache.org/apacheds/1.5/downloads.html
 [7]: http://www.openldap.org/
 [8]: http://cwiki.apache.org/DIRxSRVx10/embedding-apacheds-as-a-web-application.html  