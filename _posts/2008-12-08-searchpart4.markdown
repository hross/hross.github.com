---
layout: post
title: Adventures in Search - Part 4 - Search Node Operation
categories: [webcenter]
---

You know, it's funny how some things can seem extremely complicated and then when you crack them open they turn out to be fairly easy to understand. Remember the mystery behind how a G.I. Joe stayed together, but then you broke one and found it was simply a rubber band holding his guts together? Turns out search server is much like that. A terribly complicated-seeming C program that, fundamentally, is held together by a rubber band.

### What is a Search Node?

From my [previous posts][1], you've probably inferred that search nodes are the fundamental building blocks of ALUI's search capability. In fact, search nodes are actually the \*only\* building blocks of the search capability. Everything you need to set up a clustered or non-clustered search environment is contained in one simple install, a few directories and an executable.

All this seemingly complicated system amounts to is the following breakdown:

1.  An executable running somewhere listening for requests 
2.  An open TCP port that receives text based search queries 
3.  Two directories that contain everything search needs to operate: a cluster directory and a node directory.

Here's a more complicated picture of what I just listed:

![search_node_architecture][2]

**Figure 1 - Search Node Architecture**

### Search Requests

Let's start with the executable. When you start it up using the command line (from the bin directory in a *nix environment, or via a service on Windows), it uses environment variables to find its various configuration files, starts up a process, opens a TCP socket on whatever port you tell it to, and sits around waiting for stuff to happen.

The "stuff that happens" turns out to also be fairly simple. Search server doesn't actually know anything about portals, documents or anything else for that matter. It sits around and waits for one of two things:

*   An index request (put some information into the search index so it can be searched for later) 
*   A search request (search for something in the current index)

These two things are specified in a text-based custom language over a TCP port. What I mean is that you, Joe Six-pack, could open up a telnet session to your search server port and type a search query (index or request) freehand, were you so inclined. You would type something like the following:

    ( FIELDALIAS ptsearch,[2]PT1,[2]PT1_en,[0.1]PT2,[0.1]PT2_en,[0.1]PT50 ) (((ptsearch:a) TAG phraseQ OR (ptsearch:a*) TAG nearQ) AND ((subtype:"PTCARD")[0])) AND ((((@type:"PTPORTAL")[0]) OR ((@type:"PTCONTENTTEMPLATE")[0])) AND (((ptacl:"u2") OR (ptacl:"51"))[0]) AND (((ptfacl:"u2") OR (ptfacl:"51"))[0])) 

    METRIC logtf [1] RESULTS 10 PRINT FIELDS parentids,ptacl,ptfacl,PT51,PT56,@type,subtype,ancestors,PT58,PT7,PT53,abstracttype,

    PT1,PT1_en,PT2,PT2_en,PT3,PT4,PT5,PT6,PT8,collab_properties,collab_project_url,collab_project_name,collab_icon_alttext_index,collab_acl,publisheduser,portletid TERMS 10000 results[1-10] KWIC 15
    

Obviously, this kind of a query isn't very pretty or intuitive, but the point is you could type it via telnet and search server would spit out an XML formatted response to your query. You can see these types of queries in your search node logs if you set your logging levels high enough. Lucky for you, the search API takes care of all of this heavy lifting and converts those XML results into the pretty HTML you see when you perform a search in the portal.

### Building a Search Index

"Okay Ross," you're probably thinking, "I can run search queries over telnet to see what's in my search index. That's all well and good, but how does all that junk get in the index in the first place?"

How indeed. As I mentioned above, that junk gets in there via an index request, which is much like a search request (runs over a TCP port, follows a specific querying language), but allows whoever or whatever to put information into search instead of extract it.

If you look closely at your Publisher *content.properties* file, Collaboration *config.xml* file or even at the portal database (PTSERVERCONFIG table), you will see an "Indexing Search Port" and "Indexing Search Host" specified. What these values really do is tell each product (Portal, Publisher, Collaboration) where to submit their new document data (i.e. when someone publishes something, uploads something to a project, or a crawler runs). That data is submitted over the same TCP port to the same type of node that handles queries.

### How an Indexing Request Works

Here's a brief explanation followed by a couple of pictures:

1.  An index request is submitted to a search node. Since that search node may be part of a multi-node cluster, the request goes straight to the cluster file system (remember, all nodes share this directory). 
2.  The request is assigned a transaction ID and added to a queue on the cluster (you can see this in the form of the requests folder in the cluster folder of your search node). 
3.  Every search node in the cluster independently maintains its own transaction ID, which corresponds to the last index request it processed. These nodes continually poll the shared requests folder. If they find a transaction that has a higher ID than the one they maintain, they pull the information for that transaction and add it to their local search index. They then update their local transaction ID to match the transaction they just processed.

You can actually see this process in real time by amping up your search logs and watching the transaction ID's increment when you upload a collab document, create and admin object, etc.. Here's a few Powerpoint diagrams I created of this process:

![index_request1][3]

**Figure 2 - Adding an index request to the cluster's transaction queue.**

![index_request2][4]

**Figure 3 - Updating a local search index from the transaction queue.**

### Conclusion

As far as node operation goes, that should clear up most of the mystery. At this point, you should understand most of the how's and why's of search operation. The last piece to this puzzle is the "checkpoint" feature, which I'll review in the final exciting chapter of this blog series.

 [1]: http://hross.net/blog/2008/07/adventures-in-search-part-1-wh.html
 [2]: /images/search_node_architecture.jpg
 [3]: /images/index_request1.jpg
 [4]: /images/index_request2.jpg  