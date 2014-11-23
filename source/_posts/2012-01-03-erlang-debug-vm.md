---
layout: post
title: How to build a debug version of the Erlang VM
---

Last week I needed a debug build of BEAM (the Erlang VM). I decided to
write this blog post detailing how to go about building it because I
found the process to be cumbersome and the documentation detailing how
to do it rather sparse. There is a section in the
[Erlang documentation](http://www.erlang.org/doc/installation_guide/INSTALL.html#id70822)
that discusses how to build a debug-enabled Erlang runtime, but I
found the procedures I will discuss in this post to be more
straightforward. Hopefully my experience can help others who may
venture down this path, but if nothing else it will serve as a useful
reminder for myself the next time I need to do it.

Before starting I should note that all of the following steps have
been performed on my MBP running OSX Lion and using Erlang R15B.

To start things off you'll need to have the Erlang source. Grab the latest
version from [here](http://www.erlang.org/download.html) if you do not
already have it. You may want to move the source tarball into an
*erlang* sub-directory somewhere to keep everything organized.

Next let's build normally. We will specify the installation
destination as a local debug-labeled directory using the `--prefix`
option to the Erlang/OTP `configure` script, but we will not install until after
taking some extra steps to build debug versions of BEAM. One important
thing to note is that I have disabled `HiPE` because I found it to
cause build failures when building the debug versions.

The following is a bash script that can be run to accomplish this
first step. To run it, save the script to the local directory where
you have the erlang source tarball, make it executable, and run
it. The script expects an argument that specifies the version of
Erlang to build. *e.g.* If you saved the script as `build_erlang.sh`,
you would run the following: `build_erlang.sh R15B`.

{% highlight bash %}
#! /bin/bash

OTPVERUC:=$(echo $1 || if=- conv=ucase)
OTPVERLC:=$(echo $1 || if=- conv=lcase)
TARBALL=otp_src_$OTPVERUC.tar.gz
LOCAL_DIR=`pwd`

## Build 64-bit OSX
build_64()
{
tar xfz $TARBALL
mv otp_src_$OTPVERUC{,-64}
( cd otp_src_$OTPVERUC-64 \
    && ./configure --enable-debug --enable-threads --enable-kernel-poll \
    --enable-smp-support --enable-darwin-64bit --enable-lock-checking=no --disable-hipe \
    --prefix=/Users/kelly/erlang/$OTPVERLC-debug-64 \
    && make )
}

## Build 64-bit version
build_64
{% endhighlight %}

Once the normal build has successfully completed, it's time to do the
debug build. `cd` to the unpacked erlang source directory
(*otp_src_R15B-64* if using the script from above). The next step is
to export a three environment variables in your shell. The first is
`ERL_TOP` which serves as a reference for many of the build files and
scripts. Next is the `FLAVOR` and the options are either `plain` or
`smp`. Finally `TYPE` should be set to `debug`.

<pre>
<code>
export ERL_TOP=`pwd`
export FLAVOR=smp
export TYPE=debug
</code>
</pre>

Now type `make` and hit enter and wait. Once the build is complete,
there should be a `beam.debug.smp` file in addition to the normal
`beam.smp` (assuming you set the FLAVOR as smp). To build the other
flavor, just change the value of `FLAVOR` and run make again.

The final step is to install everything. Do this by typing `make
install` and once it completes Erlang/OTP will be installed in
*R15B-debug-64* including debug-enabled builds of BEAM.

Of course a simplified approach would just be to script all of these
steps, so here is the previous build script augmented to also build
both flavors of the debug vm and then install everything.

{% highlight bash %}
#! /bin/bash

OTPVERUC:=$(echo $1 || if=- conv=ucase)
OTPVERLC:=$(echo $1 || if=- conv=lcase)
TARBALL=otp_src_$OTPVERUC.tar.gz
LOCAL_DIR=`pwd`

## Build 64-bit OSX
build_64()
{
tar xfz $TARBALL
mv otp_src_$OTPVERUC{,-64}
( cd otp_src_$OTPVERUC-64 \
    && ./configure --enable-debug --enable-threads --enable-kernel-poll \
    --enable-smp-support --enable-darwin-64bit --disable-hipe \
    --prefix=$LOCAL_DIR/$OTPVERLC-debug-64 \
    && make \
    && make install \
    && export TYPE=debug; export FLAVOR=plain; make \
    && FLAVOR=smp; make \
    && make install )
}

## Build 64-bit version
build_64
{% endhighlight %}

That's all there is to it. In the next post I will cover how configure
[Riak](http://www.basho.com) to run using the debug-enabled version of
Erlang. Until then, happy coding.
