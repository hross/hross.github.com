---
layout: post
title: Using Content Expiration to Improve Portal Performance
categories: [webcenter]
---

Recently I was asked by a colleague what kind of tips I might have for portal administrators in order to compile a "top 10" list of portal tips and tricks. I hate to possibly ruin the surprise, in case it makes it to the [Participate][1] presentation, but one of the tips I often give people is to enable content expiration on their image server. 

The portal has a ton of images and javascript that get provided to a user's browser on each request, and this is not necessarily a good thing (check out [Yahoo's YSlow][2] analysis of it). Luckily, those images/javascript don't change very often. Thus, we can normally tell the user's browser to cache those images so it doesn't have to ask for them every time.

Sometimes, especially on intranet portals, we can cache those images for days at a time. This is called configuring content expiration. Although it doesn't improve "real" performance, it sure reduces the amount of round trips someone's browser has to make, thereby improving perceived performance. Here's a few links to give you specifics on configuring it in your image server of choice (courtesy of Google, of course):

[How to configure content expiration in IIS][3]

[How to configure content expiration in Apache][4]

And if you're looking for more details on caching/performance improvement, [this is an interesting article][5].

Speaking of [Participate][1], I'll be giving a (fairly technical) presentation on Search Server, so be sure to give me a heads up if you'll be attending.


 [1]: http://www.bea.com/participate/
 [2]: http://developer.yahoo.com/yslow/
 [3]: http://technet2.microsoft.com/windowsserver/en/library/1beefc3b-5117-4812-81a5-f7cf7b1997b71033.mspx
 [4]: http://httpd.apache.org/docs/2.0/mod/mod_expires.html
 [5]: http://www.mnot.net/cache_docs/#EXPIRES