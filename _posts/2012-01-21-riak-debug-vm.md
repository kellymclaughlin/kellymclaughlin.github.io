---
layout: post
title: Running Riak with a debug version of the Erlang VM
---

# Running Riak with a debug version of the Erlang VM

In this post I describe how the easiest way to run
[Riak](http://www.basho.com) with a debug build of the Erlang
VM. While I focus specifically on Riak, this procedure should hold
true for most Erlang projects that are set up for release builds using
rebar.

First, make sure the debug build of Erlang is the first in your
path. If you have not built a debug-enabled Erlang VM, I covered how
to accomplish that in a prior post and you can find it
[here](http://kelly-mclaughlin.com/2012/01/03/erlang-debug-vm.html).

Next navigate to your Riak repository and do a clean and release build
of Riak.

{% highlight bash %}
make clean rel
{% endhighlight %}

Move to the Riak release directory and open the riak script in an editor.

{% highlight bash %}
cd rel/riak
emacs bin/riak
{% endhighlight %}

Look for the line `EMU=beam`. It should be located somewhere around
line 228 in the section for the `console` command. Change that line to
`EMU=beam.debug` and save the file.

Now you are ready to start up Riak. For this example I just start it
with *console* instead of *start*.

{% highlight bash %}
./bin/riak console
{% endhighlight %}

You should notice the Erlang banner is slightly different than
normal. Using the normal Erlang VM the banner looks something like
this:

{% highlight erlang %}
Erlang R15B (erts-5.9) [source] [64-bit] [smp:4:4] [async-threads:64] [kernel-poll:true]
{% endhighlight %}

However using the debug-enabled VM, it should look something like the
following:

{% highlight erlang %}
Erlang R15B (erts-5.9) [source] [64-bit] [smp:4:4] [async-threads:64] [kernel-poll:true] [type-assertions] [debug-compiled] [lock-checking]
{% endhighlight %}

And that's all there is to it. Once you have the debug-enabled Erlang
VM built, it's very easy to get Riak up and running with it. It may go
without saying, but Riak will run noticeably slower using the
debug-enabled VM so please do not run in this manner in
production. Happy debugging!
