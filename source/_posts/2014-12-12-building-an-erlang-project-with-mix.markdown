---
layout: post
title: "Building An Erlang Project With Mix"
date: 2014-12-12 10:58:57 -0700
comments: true
categories: Erlang
tags: erlang, elixir, mix, rebar
---

### *What in the world is Mix? Why would anyone want to do this?*

These are the sorts of questions that any readers of this post may be
asking themselves based on the title. The answer to the former is that
[Mix](http://elixir-lang.org/getting_started/mix_otp/1.html) is the
build tool of the Elixir ecosystem.  For me the answer to the second
question is simply curiosity and this is a brief summary of my
experience following that curiosity.

As an Erlang developer for the last five years I admit to having paid
very little attention to Elixir until very recently. The inspiration
to actually take a closer look came from listening to an interview
with José Valim on an
[episode](http://www.functionalgeekery.com/episode-17-jose-valim/) of
the [Functional Geekery](http://www.functionalgeekery.com/)
podcast. There were several aspects of Elixir discussed that piqued my
interest and one of them was the build tool known as Mix.

Mix provides several features that are appealing having experienced a
lot of frustration with [rebar](https://github.com/rebar/rebar), the
most popular build tool for Erlang projects. Most of the frustration
with rebar comes from the dependency handling and so it was very
interesting to see what Mix offered in this area in particular. Some
of the features I found notable were:

* Built-in dependency locking behavior to aid in repeatable builds.
* Ability to resolve a dependency conflict in the top level
  configuration by explicitly *overriding* any other version of the
  same dependency encountered in the dependency tree.
* The concept of environments (*dev*, *test*, and *prod* are the defaults)
  to facilitate things like omitting test dependencies from the
  development or production builds.
* An umbrella mode for creating new projects that do not directly
  contain source code, but are compositions of other applications and
  dependencies.
* Ability to specify binary packages from the
  [Hex package manager](https://hex.pm) as dependencies in addition to
  git repositories.

In Erlang projects that use rebar some of these things can be
accomplished with plugins or various other workarounds, but I like
that they are baked into Mix. I suspect many of these features will be
part of [rebar3](https://github.com/rebar/rebar3), but it is still in
the development stages whereas Mix can be used today. Another benefit
from using Mix is the possibility for people to contribute to a
project using either Erlang or Elixir. This certainly will not be
applicable to all projects or desired by some, but in general I do
count greater flexibility as a benefit.

### Specifying a Mix configuration

After hearing the description of Mix on the podcast as mentioned
previously I decided to play around with it a bit. I was very
interested in the fact that Mix could be used to build regular
Erlang projects so I decided to test out what it would take to
actually make this happen for a real project. I have quite a bit of
familiarity with Basho's [riak_cs](https://github.com/basho/riak_cs)
project so I decided to use it as my test project.

First here is the `rebar.config` file from `riak_cs`:

```
{sub_dirs, ["rel"]}.

{require_otp_vsn, "R15|R16"}.

{cover_enabled, true}.

%% EDoc options
{edoc_opts, [preprocess]}.

{lib_dirs, ["deps", "apps"]}.

{erl_opts, [debug_info, warnings_as_errors, {parse_transform, lager_transform}]}.

{xref_checks, []}.
{xref_queries,
 [{"(XC - UC) || (XU - X - B - \"(^riak$|^riak_cs_dummy_reader$|^riak_core_bucket$|^app_helper$|^riakc_pb_socket_fake$|^riak_object$|^riak_repl_pb_api$|^riak_cs_multibag$)\" : Mod)", []}]}.
{xref_queries_ee,
 [{"(XC - UC) || (XU - X - B - \"(^riak$|^riak_cs_dummy_reader$|^riak_core_bucket$|^app_helper$|^riakc_pb_socket_fake$|^riak_object$)\" : Mod)", []}]}.

{reset_after_eunit, true}.

{plugin_dir, ".plugins"}.
{plugins, [rebar_test_plugin, rebar_lock_deps_plugin]}.

{client_test, [
               {test_paths, ["client_tests/erlang"]},
               {test_output, ".client_test"}
              ]}.
{int_test, [
            {test_paths, ["int_test"]},
            {test_output, ".int_test"}
           ]}.
{riak_test, [
             {test_paths, ["riak_test/tests", "riak_test/src",
                           "deps/riak_cs_multibag/riak_test/tests",
                           "deps/riak_cs_multibag/riak_test/src"]},
             {test_output, "riak_test/ebin"}
            ]}.

{deps, [
        {node_package, "1.3.8", {git, "git://github.com/basho/node_package", {tag, "1.3.8"}}},
        {getopt, ".*", {git, "git://github.com/jcomellas/getopt.git", {tag, "v0.4.3"}}},
        {webmachine, ".*", {git, "git://github.com/basho/webmachine", {tag, "1.10.3"}}},
        {riakc, ".*", {git, "git://github.com/basho/riak-erlang-client", {tag, "1.4.2"}}},
        {lager, ".*", {git, "git://github.com/basho/lager", {tag, "2.0.3"}}},
        {lager_syslog, ".*", {git, "git://github.com/basho/lager_syslog", {tag, "2.0.3"}}},
        {eper, ".*", {git, "git://github.com/basho/eper.git", "0.78"}},
        {druuid, ".*", {git, "git://github.com/kellymclaughlin/druuid.git", {tag, "0.2"}}},
        {velvet, "1.3.*", {git, "git://github.com/basho/velvet", "4bb0fd664ff065c4082ca8dd2e0683e920537d15"}},
        {poolboy, "0.8.*", {git, "git://github.com/basho/poolboy", "0.8.1p2"}},
        {folsom, ".*", {git, "git://github.com/boundary/folsom", {tag, "0.8.1"}}},
        {cluster_info, ".*", {git, "git://github.com/basho/cluster_info", {tag, "1.2.4"}}},
        {xmerl, ".*", {git, "git://github.com/shino/xmerl", "b35bcb05abaf27f183cfc3d85d8bffdde0f59325"}},
        {erlcloud, ".*", {git, "git://github.com/basho/erlcloud.git", {tag, "0.4.4"}}},
        {rebar_lock_deps_plugin, ".*", {git, "git://github.com/seth/rebar_lock_deps_plugin.git", {tag, "3.1.0"}}}
       ]}.

{deps_ee, [
           {riak_repl_pb_api,".*",{git,"git@github.com:basho/riak_repl_pb_api.git", {tag, "0.2.5"}}},
           {riak_cs_multibag,".*",{git,"git@github.com:basho/riak_cs_multibag.git", {branch, "master"}}}
          ]}.
```

Now here is the mostly equivalent translation to a `mix.exs` file:

```
defmodule CS.Mixfile do
  use Mix.Project

  def project do
    [app: :riak_cs,
     version: "1.5.2",
     compilers: [:erlang, :app],
     erlc_options: [{:parse_transform, :lager_transform}],
     deps: deps]
  end

  def application do
    [applications: [:kernel,
                    :stdlib,
                    :inets,
                    :crypto,
                    :mochiweb,
                    :webmachine,
                    :poolboy,
                    :lager,
                    :cluster_info,
                    :folsom]]
  end

  defp deps do
    [{:node_package, github: "basho/node_package", tag: "1.3.8", compile: "../../rebar compile"},
     {:getopt, github: "jcomellas/getopt", tag: "v0.4.3"},
     {:ibrowse, github: "cmullaparthi/ibrowse", tag: "v4.0.1", override: true},
     {:webmachine, github: "basho/webmachine", tag: "1.10.6"},
     {:riakc, github: "basho/riak-erlang-client", tag: "1.4.2"},
     {:lager, github: "basho/lager", tag: "2.0.3", override: true},
     {:lager_syslog, github: "basho/lager_syslog", tag: "2.0.3"},
     {:eper, gippthub: "basho/eper", tag: "0.78"},
     {:meck, github: "eproxus/meck", tag: "0.8.2", override: true},
     {:druuid, github: "kellymclaughlin/druuid", tag: "0.3"},
     {:velvet, github: "basho/velvet", ref: "4bb0fd664ff065c4082ca8dd2e0683e920537d15"},
     {:poolboy, github: "devinus/poolboy", tag: "1.4.1"},
     {:folsom, github: "boundary/folsom", tag: "0.8.1"},
     {:cluster_info, github: "basho/cluster_info", tag: "1.2.4"},
     {:erlcloud, github: "basho/erlcloud", tag: "0.4.4", only: :test}
    ]
  end
end
```

The two files are very similar, but the thing that sticks out most is the
cleaniness of the dependency specification in the Mix file versus the
rebar configuration. The `github` shorthand is very nice since it is the
most common home for Erlang projects. Also, notice that Mix gathers the
information to generate the project's `.app` file from information in
the `mix.exs` file rather than an `app.src` file.

The dependencies that specify `override: true` are ones that had
version conflicts and I was able to select the correct version
explicitly at the top level. This is also the reason that the set of
dependencies specified differs slightly between the rebar and Mix
configurations.

In order to build the project I also had to specify a basic `mix.exs`
file in the apps/riak_cs directory. Here is the listing of that file:

```
Code.require_file "mix.exs", "../.."

defmodule CS.App.Mixfile do
  use Mix.Project

  def project do
    []
  end
end
```

### Testing it out

Elixir and Mix require at least Erlang 17 so I did have to alter the
versions some of the dependencies in order to get a working build due
to some version-related build errors and for some of the Basho
dependencies I had to manually edit the rebar.config file to allow
using version 17 or disable `warnings_as_errors` (also related to
building with Erlang 17). Beyond that everything built as expected with
very minimal overall effort.

Here are the steps to follow to build the project:
<ol>
<li><code>mix deps.get</code></li>
<li>
Edit <code>deps/velvet/rebar.config</code> in the following manner:
```
diff --git a/rebar.config b/rebar.config
index 9a03432..56d7170 100644
--- a/rebar.config
+++ b/rebar.config
@@ -1,10 +1,10 @@
-{require_otp_vsn, "R14B0[234]|R15|R16"}.
+{require_otp_vsn, "R14B0[234]|R15|R16|17"}.

 {cover_enabled, true}.

 {lib_dirs, ["apps"]}.

-{erl_opts, [debug_info, warnings_as_errors, {parse_transform, lager_transform}]}.
+{erl_opts, [debug_info, {parse_transform, lager_transform}]}.

{reset_after_eunit, true}.
```
</li>
<li>
Edit <code>deps/riakc/rebar.config</code> in the following manner:
```
diff --git a/rebar.config b/rebar.config
index f09a602..9765dc3 100644
--- a/rebar.config
+++ b/rebar.config
@@ -1,6 +1,6 @@
 {cover_enabled, true}.
 {eunit_opts, [verbose]}.
-{erl_opts, [warnings_as_errors, debug_info]}.
+{erl_opts, [debug_info]}.
 {deps, [
         {riak_pb, ".*", {git, "git://github.com/basho/riak_pb", {tag, "1.4.4.0"}}}
        ]}.
```
</li>
 <li><code>mix deps.compile && mix compile</code></li>
</ol>

That is all there is to it. The results should be in the local `_build`
directory.

### Conclusion and further explorations

Mix appears to be a well-implemented, flexible build tool for building
Elixir or Erlang projects. From an Erlang developer's perspective, it
definitely addresses some pain points that frequently arise with
dependency handling with rebar.

I also like that the build tool is a core part of the Elixir
ecosystem. Many Erlang projects include a copy of the rebar executable
as part of the git repository because rebar is not the official Erlang
build tool and is not distributed as part of the language
installation. This, in my view, is a problematic and sloppy solution so
Elixir definitely improves upon that with Mix.

I have not explored how Elixir/Mix handle interaction with
[reltool](http://www.erlang.org/doc/man/reltool.html) for creating
releases yet. An interesting next step will be to explore this more
and attempt to get an actual release of the project created. One missing
thing from the `rebar.config` in the example of `riak_cs` is the
handling of the rebar plugin for executing customized testing
functions. Translating the plugin actions into Mix tasks would be
another interesting learning experience.
