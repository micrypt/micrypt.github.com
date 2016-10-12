--- 
layout: post
title: "Notes on the Amazon Dynamo Paper"
postdate: 2011-01-28 21:04:09
summary: Caveat emptor - My incomplete notes on the Amazon Dynamo paper. These are probably not suitable for public consumption.  
---

The following are some notes from reading the Dynamo paper. If anyone're reading this... it's not even a weak substitute for reading the paper. "Man up and go read the [paper](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf!"). Instead, these are my notes on the aspects of the paper that I found particularly interesting. Being what it is, the points are somewhat disjointed and probably wouldn't make sense to anyone. It's a quick reference I can pull up sometime in the future when I'm "old, frail and weak of mind". Enough of caveats, here go goes nothing.

Apache Dynamo is a highly scalable data storage technology. In order to explain what this means, it's useful to expand on two elements of the description.

"highly scalable" - this is achieved via replication with tunable eventual consistency using a quorum system
"data storage" - it's a key/value store

Consistency
-----------

In cases where high consistency is a requirement, it is impossible to also provide high availability as the instances have to come to an agreement before the data is made available. In Apache Dynamo, the quorum system allows the admin to set the number of nodes that need to agree on a condition before an operation is carried out.

Conflict resolution
-------------------

In order to have a good customer experience, Amazon pushes the resolution of conflicts to the read step. This reduces latency from attempting to implement conflict resolution at the data store level. The node that receives the request can just push out all the results from other nodes with data and let the application layer handle resolution. In this case that the application developer chooses to let the data store handle conflict resolution, the default approach is that "the last write wins" (this is similar to conflict resolution in Apache CouchDB.) As would be expected, this is done with object versioning.

Heterogeneity
-------------

Being able to add more powerful nodes without needing to upgrade the whole bunch. Rather than dealing with the issues from non-uniform data & load distribution, Dynamo uses the concept of virtual nodes where actual nodes can be assigned more than one virtual nodes, meaning that there is always more than one instance to handle an operation.

Notes
-----

[Pastry](http://en.wikipedia.org/wiki/Pastry_(DHT)) and [Chord](http://en.wikipedia.org/wiki/Chord_(DHT))
They use routing mechanisms to make sure that queries can be answered (satisfied) within a bounded number of hops.

[Oceanstore](http://oceanstore.cs.berkeley.edu/info/overview.html)
Resolves conflicts by creating an order of updates and consistently applying those updates in that order.

[Antiquity](http://oceanstore.cs.berkeley.edu/publications/papers/pdf/antiquity06.pdf)
Uses a secure log to preserve data integrity and replicates the log to multiple servers for durability. Uses Byzantine fault tolerant protocols to ensure data consistency.

Vector clock
(Node, Counter)
e.g (Sx, 3), (Sy, 1), (Sz, 1)

(Sleepy now... this will be an evolving document anyway.) - Till next time.
