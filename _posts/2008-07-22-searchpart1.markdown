---
layout: post
title: Adventures In Search - Part 1 - What is Search?
categories: [webcenter]
---

If you'll recall, a few posts ago I promised to start fleshing out the presentation I gave at Participate in this blog. It's a somewhat boring task, since I already came up with a presentation, but since I gave the presentation and posted it to my blog, there has been a lot more interest in it than I anticipated.

Apparently, everybody else is just as confused about what search is and how it works as I was. So how about we break out the flashlight and provide a point of reference for the folks who weren't at Participate, or who prefer reference material to a presentation (I know I am in that camp).

### What is Search?

As illustrated by the diagram below, when I talk about search, I simply mean a repository of information. On one side, information about the stuff we want to search is added to the repository and on the other side users or programs query that repository with requests for that information:

![what_is_search][1]

However, this is a somewhat simplified version of Search, since it interfaces with our portal in more ways than just the search box in the header. 

Here is a comprehensive listing of search uses (that I know of):

*   **Portal Search Box** - When you search for things in the portal as a non-administrator. 
*   **Administrative Search** - When you search for things as an administrator (folders, objects, etc) 
*   **Knowledge Directory** - All of the folder/document browsing screens are built from the search index, not the database. You can change this in Portal Admin Options, but it's not recommended, for performance reasons. 
*   **Content Crawlers** - Every time a new document is submitted, metadata is updated, etc (basically every time a crawler is run) 
*   **Publisher** - Used only when publishing/saving content. Publisher search is actually just a database query) 
*   **Collaboration File Upload** - Used when uploading/indexing Collaboration documents. 
*   **Collaboration Search** - Collaboration search actually does use Search. 
*   **IDK Search Factory** - When you use the IDK to perform search requests. 

That's really it. In its basic operation, search is extremely simple. Next time I'll start to delve into search architecture, specifically Nodes and Partitions.

 [1]: /images/what_is_search_basic.jpg  