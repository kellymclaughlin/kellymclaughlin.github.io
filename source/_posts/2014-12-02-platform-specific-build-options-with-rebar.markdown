---
layout: post
title: "Platform-Specific Build Options With Rebar"
date: 2014-12-02 10:48:46 -0700
comments: true
categories: Erlang, Rebar
---

Several months back someone reported an
[issue](https://github.com/kellymclaughlin/druuid/issues/3) when
attempting to use
[`druuid`](https://github.com/kellymclaughlin/druuid) as a dependency
in an Elixir project.  At the time I was not able to look into it, but
recently I circled back to dig deeper into the problem (described in
more detail [here](https://github.com/elixir-lang/elixir/issues/2189))
and in the process I stumbled across some useful rebar configuration
options for projects that have platform-specific build requirements. I
use rebar quite a bit, but I was not aware that these options
existed so I wrote this in case other rebar users find themselves in a
similar state of ignorance.

`Druuid` needs to differentiate between BSD and non-BSD systems during
the `compile` and `clean` operations. This is due to a conflict with
the OS-provided UUID library on BSD systems.  The original method used
to achieve this was rebar's dynamic configuration capability. A
rebar.config.script file was used to check the system architecture
information using `rebar_utils:is_arch/1` and modify the rebar
configuration appropriately based on the results ([relevant github issue](https://github.com/kellymclaughlin/druuid/pull/2)). [Here](https://github.com/kellymclaughlin/druuid/blob/0.2/rebar.config.script)
is the full listing of the `rebar.config.script` file for reference.

This solution certainly works, but it would be great if there was a
way to use standard rebar configuration options to solve the problem
and even better if anyone who wants to use the library, be it in
an Erlang project or an Elixir project, could easily do so.

Fortunately such configuration capabilities already exist in
rebar. `Druuid` uses four rebar configuration directives to tailor the
build environment and steps based on the platform.  These are as
follows: `port_specs`, `port_env`, `pre_hooks`, `post_hooks`. Each
option is specified in the `rebar.config` file as a pair where the
first element is the option name and the second element is a list of
tuples.

For example, here is what a `port_env` entry might look like:

```
{port_env, [{"CFLAGS", "$CFLAGS -fPIC"},
            {"DRV_CFLAGS", "$DRV_CFLAGS -Werror -I c_src/uuid-1.6.2"},
            {"DRV_LDFLAGS", "$DRV_LDFLAGS c_src/uuid-1.6.2/.libs/libuuid.a"}
]}.
```

It turns out that the value tuples can be a pair as shown in the above
example **or** they can be triples where the first element is a
regular expression that is applied as a filter to the platform
information. Let's see an example of that:

```
{port_env, [{"CFLAGS", "$CFLAGS -fPIC"},
            {"(bsd)", "DRV_CFLAGS", "$DRV_CFLAGS -Werror"},
            {"(bsd)", "DRV_LDFLAGS", "$DRV_LDFLAGS"},
            {"^(?s)((?!bsd).)*$", "DRV_CFLAGS", "$DRV_CFLAGS -Werror -I c_src/uuid-1.6.2"},
            {"^(?s)((?!bsd).)*$", "DRV_LDFLAGS", "$DRV_LDFLAGS c_src/uuid-1.6.2/.libs/libuuid.a"}
]}.
```

The `CFLAGS` value is applied to all platforms, but `DRV_CFLAGS` and
`DRV_LDFLAGS` are selected based on whether the platform information
does or does not contain the string *bsd*. This is very handy and can
also be used for all of the other configuration options used by
`druuid`. Let's see the final version of the `rebar.config` file that
translates all of the behavior previously contained in the
`rebar.config.script` file:

```
{port_specs, [{"priv/druuid.so", ["c_src/*.c"]}]}.
{port_env, [{"CFLAGS", "$CFLAGS -fPIC"},
            {"(bsd)", "DRV_CFLAGS", "$DRV_CFLAGS -Werror"},
            {"(bsd)", "DRV_LDFLAGS", "$DRV_LDFLAGS"},
            {"^(?s)((?!bsd).)*$", "DRV_CFLAGS", "$DRV_CFLAGS -Werror -I c_src/uuid-1.6.2"},
            {"^(?s)((?!bsd).)*$", "DRV_LDFLAGS", "$DRV_LDFLAGS c_src/uuid-1.6.2/.libs/libuuid.a"}
]}.
{pre_hooks, [{"^(?s)((?!bsd).)*$", compile, "c_src/build_deps.sh"}]}.
{post_hooks, [{"^(?s)((?!bsd).)*$", clean, "c_src/build_deps.sh clean"}]}.
```

This is much nicer than resorting to using rebar's dynamic
configuration and as a bonus it works much better with Elixir's build
tool, [Mix](http://elixir-lang.org/getting_started/mix_otp/1.html). If
you want to explore how the regular expressions are treated inside
rebar, look
[here](https://github.com/rebar/rebar/blob/2.5.1/src/rebar_core.erl#L507)
for the hooks,
[here](https://github.com/rebar/rebar/blob/2.5.1/src/rebar_port_compiler.erl#L308)
for the `port_specs`, and
[here](https://github.com/rebar/rebar/blob/2.5.1/src/rebar_port_compiler.erl#L484)
for the `port_env` for the port options.

I did not find any mention of these options in the rebar wiki. I did
find some examples of the port options in the output of `rebar help
compile`, but hopefully this post sheds some further light on their
intended purpose. Also note that the information in this post is based
on the `2.5.1` tag of rebar. Later tags or the work on
[rebar3](https://github.com/rebar/rebar3) may make some of this
information obsolete.