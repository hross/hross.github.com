---
layout: post
title: Adventures in Search - Part 3 - Search Administration
categories: [webcenter]
---

Here we are, back again for another installment in my new blog "mini-series" about search. When I first started researching these posts (er... presentation, actually) the mini-series might have been more aptly titled "Lost" (not to be confused with ABC's hit series, except for the mass confusion and never ending storyline).

Last time I promised some hard-hitting dirt on Search Administration, and as always, I deliver on my blog promises. Okay, maybe hard hitting is a bit of a stretch... let's talk about Search Administration. Most of you are probably familiar with the *Search Cluster Manager* and *Search Service Manager* in the Administrative Utilities drop down, but what are they and how do they work?

Let's start tackling this with a diagram:

![search_admin_1][1]

This diagram represents the end-all be-all of the search administration process. There are two parts:

1.  Portal communication with a search node directly. This is the *Search Service Manager *(left side of the diagram). It is basically the portal asking the node about the health and topology of the search server and the node replying with this information. This node is extremely important, since it tells the portal front end how and which search nodes to query. The query is performed over the same port as any other search request, using the same mechanisms, and will show up in your search logs if you have them at a high enough verbosity. 
2.  Portal communication with the search topology indirectly. This is done via the *Search Cluster Manager *(right side of the diagram). I have heard much rumor and hearsay regarding the Search Cluster Manager, so let me clear up any misconceptions you might have with a properly bolded and formatted statement:

***The Search Cluster Manager is a Java web application that reads and writes files on the Cluster File System.***

What this really means is that the Search Cluster Manager is totally unnecessary. All administration can be done with the **cadmin** tool (in your search server's bin directory) or via direct changes to specific initialization files (this is what the Search Cluster Manager does, anyway). So basically, the diagram above actually looks like this:

![search_admin_2][2]

### Wrap Up

So that's it. Basically, the take-away's here are:

1.  Search Cluster Manager is simply a prettied up version of the command line utility and does not need to run for search to function in the portal.

2.  Search Service Manager controls the contact node and determines search topology for the portal front end.

Pretty simple, eh? Next up... some more interesting details on node operation.

 [1]: /images/search_admin_1.jpg
 [2]: /images/search_admin_2.jpg  