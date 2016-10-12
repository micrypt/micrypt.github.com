--- 
layout: post
title: "Getting started with HyperDex"
postdate: 2012-8-10 2:11:09
summary: Super-charging your NoSQL with a datastore that cares about your data
---

Alright, let's start this off with a fitting [soundtrack](http://youtu.be/6nBR30nIoAg) just because we can. Open it up in a tab and come back?

Greetings, valiant adventurer!
------------------------------

So, I heard you care about data. You aren't storing your precious data in anything that acknowledges PUT requests before being certain it'll be able to return it to you? Well then, you've come to the right place.

Okay, I'm clearly excited, but with good reason. Some time in the past few months, I ran into a paper; "HyperDex: A Distributed, Searchable Key-Value Store"[1] from a team at Cornell. By now the typical reaction to NoSQL news tends to be that your eyes glaze over and you start mouthing "…is Web-Scale™", but this isn't "yet another NoSQL database". So, I've finally gotten round to writing this piece in hopes of sharing it with others.

Before plunging into the deep end, it's probably a good idea to discuss why I've found [HyperDex](http://hyperdex.org) to be particularly exciting. For reasons that will probably be in a different blog post, I've been researching the design of a distributed key/value store with support for strong consistency (for the morbidly curious, it's connected to [Ampify](http://github.com/tav/ampify)). You must realise that the state-of-the-art distributed key/value stores such as Dynamo (and its open-source clone, \*Riak) tend to aim for eventual consistency.

Eventually consistent systems leave it up to the application developer to resolve any conflicts that may occur. Not only does this make app development more complex, but it is also easy to get wrong. And whilst you could do strongly consistent quorum reads/writes in these systems, the performance suffers dramatically as they were not designed for that primary purpose.

There simply aren't many open source options for strongly consistent, horizontally scalable datastores with support for transactions and rich querying capabilities. Whilst big boys like Google have proprietary platforms built on top of [Megastore](http://www.cidrdb.org/cidr2011/Papers/CIDR11_Paper32.pdf), the rest of us have limited options like Scalaris (which doesn't support persisting to disk) and ScalienDB (which lacks rich querying support and requires manually sharding).

That shouldn't be surprising though as it's not a trivial problem to solve. So, when you find out that a team is actually building an open source distributed K/V store with a focus on strong consistency, and not only have a functional build, but actually outperform quite a few of the other NoSQL datastores out there (not just the distributed options), you have my go-ahead to get a little excited too. And when you find out that you even get a few primitive data structures (lists, sets and maps) with atomic operations such as set intersections and map additions like Redis, maybe it's time to start dancing in your seat.

So, what do you get?
--------------------

In the case you're as lazy as I tend to be and are still putting off reading the paper[2], the most interesting features of HyperDex are the following:

\*\# Consistent: linearizable; GET requests will always return the latest PUT.
\*\# High Availability: the system will stay up in the presence of ≤ f failures.
\*\# Partition-Tolerant: for partitions with ≤ f nodes, you can be certain your system is still operational.
\*\# Horizontally Scalable: you can grow your system by adding additional servers.
\*\# Performance: high throughput and low variance.
\*\# Searchable: it provides an expressive high performance API for searching your data.

So, that's a pretty impressive list, right? I bet you're wondering how it manages to support all those. I was. For details you'll have to actually read the paper[3], but it's a good idea to cover two pretty cool techniques involved: hyperspace hashing and value-dependent chaining.

HyperSpace hashing
------------------

With HyperSpace hashing, an object's attributes are mapped to a multi-dimensional Euclidean space (a hyperspace), with each attribute
corresponding to a dimension. For an example, let us imagine attempting to store a calendar entry. In the case of a scenario such as creating a "John's birthday on the 12th of March, in Las Vegas" entry, you might structure the attributes to include a *title*, *date* and *location*. The way hyperspace hashing works is that these attributes are mapped to corresponding planes in the hyperspace. In a simple case like the above, that would be a three-dimensional space with a matching plane for each attribute.

To determine which server handles storage of a particular attribute, the hyperspace is demarcated into non-overlapping regions/subspaces without gaps, which are matched to a corresponding server. HyperDex also stores the object's key in a dedicated one-dimensional subspace for efficient lookups.

Subsequent lookups such as one where the date is the 12th of March, and the location is Las Vegas, would only need to contact the servers which match the regions of the hyperspace assigned for those attributes.

It turns out that hyperspace hashing also comes in handy when you wish to support efficient search. The fact that object attributes are mapped to dimensions means that when you want to retrieve data, you only have to look in a specific region of the space which matches the attributes you're using to look up data. It's quite the stroke of genius.

Value-dependent Chaining
------------------------

This allows the system to keep replicas in sync without incurring a high overhead from coordination. It works by deterministically choosing how an object is replicated based on its hyperspace coordinates. Choosing a *point leader* at the head of the chain, it works out the next links in the chain based on hashing the object's attributes. Updates then flow from the head along the chain, with acknowledgements being sent in reverse from the tail. The paper[4] includes details on how state is cleaned up using this mechanism and performance considerations in its design.

So, the moment you've probably been waiting for is upon us: how do you actually use this cool piece of kit?

Setting up a playground
-----------------------

To get started, you can follow the installation [instructions](https://github.com/rescrv/HyperDex/blob/master/doc/02.installation.rst) included in the HyperDex Git repo. This covers installing one of the precompiled binaries for Debian, Ubuntu or Fedora, or compiling from source depending on your needs. If you're on Debian and run into any trouble, you could take a look at François Dussert's [guide](http://blog.usul.org/compiling-and-installing-hyperdex-from-sources-with-java-and-ycsb-bindings-on-debian/) for Debian installations.

With that done, you can have a play with the [tutorial](http://hyperdex.org/doc/tutorial/), and perhaps attempt modelling the calendar we discussed earlier.

Libraries
---------

HyperDex ships with C, C**, Python, Java and Node.JS bindings, with partial Ruby bindings. There are also the beginnings of a [Go client](https://github.com/tiborvass/hyperclient) by Tibor Vass. He's been a bit naughty and hasn't updated it in a while (5 months). So, I might have to pitch in and help finish it.

Getting a bit more involved
---------------------------

If at some point, you wish to get a bit more involved and help improve HyperDex, you should have a read through the [contributors guide](http://hyperdex.org/contributors/) which includes the expected commit message style, and expectations when sending upstream changes. A good place to start might be the hyperclient [directory](https://github.com/rescrv/HyperDex/tree/master/hyperclient) in the repo which contains the client libraries. Whip up a binding for your favorite language, perhaps?

References
----------

Notes
-----

Many thanks to [Tav](http://tav.espians.com), [Ziyad Basheer](http://ziyadbasheer.com), [Emin Gün Sirer](http://www.cs.cornell.edu/people/egs/) and [Mahipal Raythattha](http://mahipal.org) for reading drafts of this article and helping make it somewhat coherent. As always, any errors are mine. If you want any clarification or just want to troll me, you can find me [here](http://twitter.com/micrypt).

Update
------

-   In late October 2013, Basho announced Riak 2.0, which claims support for strong consistency. I have not independently investigated these claims, but this is a note that the state and expectations of Riak have changed.

[1] Escriva R., Wong B. and Sirer E., 2011. HyperDex: A Distributed, Searchable Key-Value Store for Cloud Computing. Cornell University Technical Report. <http://hyperdex.org/papers/hyperdex.pdf>

[2] Escriva R., Wong B. and Sirer E., 2011. HyperDex: A Distributed, Searchable Key-Value Store for Cloud Computing. Cornell University Technical Report. <http://hyperdex.org/papers/hyperdex.pdf>

[3] Escriva R., Wong B. and Sirer E., 2011. HyperDex: A Distributed, Searchable Key-Value Store for Cloud Computing. Cornell University Technical Report. <http://hyperdex.org/papers/hyperdex.pdf>

[4] Escriva R., Wong B. and Sirer E., 2011. HyperDex: A Distributed, Searchable Key-Value Store for Cloud Computing. Cornell University Technical Report. <http://hyperdex.org/papers/hyperdex.pdf>
