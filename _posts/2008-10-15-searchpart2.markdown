---
layout: post
title: Adventures in Search - Part 2 - Search Architecture
categories: [webcenter]
---

### Breaking Down a Search Collection

Last time I listed the various functions of search and reposted my first search slide. It was fairly simple, just an abstract "Search Collection" diagram. This time let's break that diagram down a bit more:

![what_is_search_3][1]

What we see above is a less abstract view of the same diagram. Instead of one giant "Search" lump, we actually have an API, which makes the communication decisions, and a collection of search nodes. These nodes are just processes running somewhere, listening on a specific port. More about them later.

### Partitions

That was pretty simple, right? Let's throw in one more wrinkle before moving on to the complicated bits: *Partitions*. A partition is simply a grouping of search data into a set of nodes. Applying that concept to the above diagram, a partitioning of our search collection might look something like:

![what_is_search_partitions][2]

In other words, some of the data indexed by search (search results) will reside in Partition 1 on Node 1, and some of the data will reside in Partition 2 on Nodes 1 and 2. If we draw out the partitions in a more abstract manner, they look like this:

![what_is_search_partitions_abstract][3]

As you can see, there are two separate "bins" of data. When new information is indexed it goes into one of these two bins. It is important to note that neither partition contains duplicate data, so when you search for something the results from Partition 1 and Partition 2 must be **aggregated** together. Duplicate data will, however, exist on Nodes 1 and 2 in Partition 2 (see above).

### Search Coordination

With all this data moving about, being partitioned, searched, etc, you may be wondering how all of the search nodes communicate with one another. How do they know which partition they belong to, which node they are and what data has already been indexed?

The answer, it turns out, is extremely simple. They all must share at least one common set of files and directories, which I'll call the "Cluster File System". There is no special port-to-port communication, magic pixie dust, or any other way for search nodes to talk to each other. The cluster file system contains configuration information about the entire search topology, as well as a common queue/locking mechanism for incoming search indexing requests (more detail later). In other words, our previous diagram now looks like this:

![what_is_search_cluster_file_system][4]

And that's really all there is to it. I've just covered all of the concepts you'll need for a basic understanding of search.

### Wrap Up

Alright, well we've covered the basics, but as you know, I'm never fully satisfied with the basics. Hopefully you now have a base understanding of search operation and are ready to stick with me for the under-the-covers part. Most of the information I've provided to this point is covered in the docs, just (in my opinion) not very well. Next time look for some more detailed information on how search administration works and under the covers node operation.

 [1]: /images/what_is_search.jpg
 [2]: /images/what_is_search_partitions.jpg
 [3]: /images/what_is_search_partitions_abstract.jpg
 [4]: /images/what_is_search_cluster_file_system.jpg  