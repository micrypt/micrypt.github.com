--- 
layout: post
title: "Squeryl testing with specs2: gotchas"
postdate: 2011-07-4 12:04:09
summary: A quick start guide to using squeryl with specs2 for unit testing.
---

In the absence of a detailed quick start guide to testing [Squeryl](http://squeryl.org) projects with [specs2](http://etorreborre.github.com/specs2), I've ended writing up some information for anyone else getting started with combining both projects. After quite a bit of hair pulling, I got some guidance on doing so from [@etorreborre](http://twitter.com/etorreborre).

Avoid implicit clashes
----------------------

As it turns out Squeryl and specs2 have a number of name clashes. You can avoid the implicits clash by using the \`&gt;&gt;\` operator and other similar abbreviations provided by specs2 when you run into them. I initially had no idea they existed and spent a fair bit of time trying to fix this implicit resolution issue whilst grumbling loudly on Twitter:

<script src="https://gist.github.com/1256663.js?file=schema.scala">
</script>
Sequential execution or concurrent execution
--------------------------------------------

If you have issues with examples being executed concurrently you can try to use the \`sequential\` argument at the beginning of the specification:

<script src="https://gist.github.com/1256663.js?file=database.scala">
</script>
Other bits of advice I got included pointing out that sometimes it is smarter to attach the database session to the current thread, so that you can still have your examples being executed concurrently. Also, if you create a \`Before\` context object, you need to \`apply\` it to your example body, but the easiest thing to do is to use the BeforeExample trait in that case.

Here's an example unit test to get you started:

<script src="https://gist.github.com/1256663.js?file=sample.scala">
</script>
