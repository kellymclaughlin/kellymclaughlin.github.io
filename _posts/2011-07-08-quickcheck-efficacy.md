---
layout: post
title: A Quickcheck in time saves nine
---

# A Quickcheck in time saves nine

Today I am going to present some anecdotal evidence on the efficacy
of property-based testing. The specific tool in this case is Quiviq's
[Quickcheck](http://www.quviq.com/), a property-based testing tool for
Erlang. I am not going to spend *much* time discussing the particulars
of property-based testing, but instead focus on a specific real-world
instance where it saved me and my employer,
[Basho](http://www.basho.com/), time and money.

## The Bare Essentials
Very briefly, here is enough about how property-based testing with
Quickcheck works to whet your appetite. When doing property-based
testing you specify a set of properties of the code you are testing,
create a model, and then test if the model holds for all (or a large
number) of permutations of those properties.

My coworker Rusty Klophaus published a very good 3-minute podcast on
property-based testing
[here](http://riak.minutewith.com/pages/20110128) that you _should_
listen to now if you have not already.

The full version of Quickcheck requires a license which I am fortunate
enough to be able to use when testing code for
[Basho](http://www.basho.com/). [Contact](http://www.quviq.com/contact.html)
Quiviq for more information about that or they also have a free
version called QuickCheck Mini with a reduced feature set. There are
also some open source property testing tools for Erlang such as
[PropEr](https://github.com/manopapad/proper), but I do not have any
first-hand experience using any of them.

## Some Relevant Data
Here are a couple of tidbits that will help you follow the
story if you are unfamiliar with how [Riak](http://www.basho.com)
works.
 * In [Riak](http://www.basho.com), the data that is stored is divided among a set of partitions. Understanding the specifics of a partition is not important here, just understand that a partition is a place where data is stored and that to generate a list of all the object keys for a bucket in [Riak](http://www.basho.com) requires accessing a subset of these partitions.
 * The number of partitions is configurable.
   
     
## Story Time
Last week I was working on some changes to how
[Riak](http://www.basho.com) manages key listing. I made the changes I
thought were necessary to accomplish my task and did some preliminary
manual testing. This involved inserting a set of objects into a bucket
and then performing a key listing and verifying that results matched
what I expected. This manual testing closely resembles the type of
test cases I would likely create for this code using traditional unit
tests. Pick a few different sizes of object sets and done. Most
developers probably will not spend all day enumerating hundreds of
test cases and the result is that the unit test suite only gives you
assurance that your code works for a very small number of possible
cases. So what about the rest of them? Well, let's continue the story.

It turned out that there was some intermediate results buffering that
I had not accounted for when I made my changes. Each partition would
accumulate the object keys it had stored for a bucket, but once this
buffer contained 100 keys it would send the keys back to the calling
process, empty the buffer, and continue the accumulation. The default
number of partitions in [Riak](http://www.basho.com) is 64. The size
of the set of partitions required to list all of the keys from a
bucket when there are 64 partitions is 22 (just take my word for
it). Assuming a uniform distribution of keys among the partitions,
there would need to be at least 2179 objects in a bucket to trigger
the intermediate buffering. The largest set of objects I tried in my
manual testing was 2000, but fortunately the object set size is one of
the inputs that the Quickcheck test for this code generates for me and
when I ran the test it quickly found a case where the expected set of
object keys did not match the actual set of keys returned.

Now chances are good that a suite of traditional unit tests would have
tested an object set greater than 2000, but what if the buffer size
that triggered the intermediate buffering was 1000 or 10000. Or what if
the number of partitions was set to 128 or 256 instead of 64. The
point is that as the number of variables that affect the code
under test increases, the number of test case permutations quickly
becomes difficult to manage with traditional unit test suites and
relies on the developer not to overlook any important
cases. 

Instead with Quickcheck the developer describes the characteristics of
the variables that affect the code and Quickcheck explores the
permutations by generating values for the variables and searching for
failing cases.

## In Summary
Catching and fixing bugs in development saves time and
money. Quickcheck helped me find and eliminate this bug early and I
have a higher level of confidence that my changes are correct.  The
case I described here was rather trivial, but it is illustrative of
the power and usefulness of property-based testing. Creating
Quickcheck tests may take more time than churning out a few unit test
cases (especially as you ascend the learning curve), but this is time
well spent and the investment will pay dividends.

If you would like to have a look at the Quickcheck test I discussed in
this post, you can find it
[here](https://github.com/basho/riak_kv/blob/master/test/keys_fsm_eqc.erl). Happy
testing!

