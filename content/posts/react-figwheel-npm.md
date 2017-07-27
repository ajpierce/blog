---
title: "React in Figwheel from Scratch"
date: 2017-07-26T15:58:36-04:00
draft: false
---

The Clojurescript team has been doing some [amazing work](https://clojurescript.org/news/2017-07-12-clojurescript-is-not-an-island-integrating-node-modules)
recently with regard to interop with the Javascript ecosystem. I was surprised to
discover that it's easier to set up a Clojurescript project now than a [modern Javascript web app](https://hackernoon.com/how-it-feels-to-learn-javascript-in-2016-d3a717dd577f).

Our Web UI stack at [BlackSky](https://blacksky.com) is React paired with Redux,
ImmutableJS, and a workflow powered by Webpack (transpilation with Babel,
Hot Module reloading, code splitting, ESLint, Prettier -- the works).

The total developer time we invested in setting everything up as it evolved over
the months was probably on the order of weeks &ndash; an expensive endeavor.
But the cost of setting up our Javascript development environment was cheap
when you consider the benefits we derive from:

+ Tight feedback loops (enabled by HMR and React/Redux)
+ Elimination of a whole class of bugs (enabled by Redux's uni-directional data flow and Immutable objects)
+ The rich ecosystem of React components published as open source

Clojurescript has given its developers the first two points for "free" since
2014 thanks to Figwheel and Clojure's built-in immutable objects, but last
bullet point has always eluded us.

Until now!

Here's how you can set up a brand new Clojurescript web app to consume the
popular React framework (and, presumably, almost any other project published on
NPM), and all in around 5 minutes!

# Step 1: Install Clojurescript
At the time of writing, the version of Clojurescript supporting these features
has not been officially released yet, but it's on the master branch of the
[Clojurescript repository](https://github.com/clojure/clojurescript).

```
$ cd /tmp
$ git clone git@github.com:clojure/clojurescript.git
$ cd clojurescript
$ ./script/build
```

This will compile the latest version of Clojurescript, and drop the assets into
your local Maven repository.  If you don't have Maven, consider running `brew
install maven`.

# Step 2: Create a new project

```
$ cd /tmp
$ lein new figwheel hhnnngg
$ cd hhnnngg
```

# Step 3: Upgrade Dependencies

At the time of writing, the lein-figwheel template uses older libraries that do
not support our desired functionality.  We're going to upgrade them!

Inside `project.clj`:

## ClojureScript
Update `:dependencies` to use the version of Clojurescript you just installed in
Step 1 &mdash; check the output if you're not sure. For me, it was:

`[org.clojure/clojurescript "1.9.828"]`

## Plugins
Update `:plugins` to use:

```
:plugins [[lein-figwheel "0.5.11" :exclusions [org.clojure/clojurescript]]
          [lein-cljsbuild "1.1.6"]]
```

Near the bottom of the file, you will need to update the Figwheel sidecar to match:

```
[figwheel-sidecar "0.5.11"]
```

# Step 4: Add NPM Dependencies

In your `:cljsbuild`, you should have two `:compiler` sections: one for dev, and
one for prod.  We're going to modify them by **adding** the following:

```
:closure-defines {process.env/NODE_ENV "development"}
:npm-deps {:react "15.6.1" :react-dom "15.6.1"}
```

We use [`:clojure-defines`](https://github.com/clojure/clojurescript/wiki/Compiler-Options#closure-defines)
as a way to distinguish between `dev` and `min` builds; in the `min` build
defined in `:cljsbuild`'s `:builds`, you'll want to set:

```
:closure-defines {process.env/NODE_ENV "production"}
```

# Step 5: Add process.env namespace

When I first tried getting React to play nicely with Figwheel, I kept getting
errors about how `process` was undefined. Turns out, React relies on the
NODE_ENV environment variable (in Node, `process.env.NODE_ENV`) to know whether 
or not it's in development mode, and without this environment variable... well, 
there were errors.

To solve this issue, we take the following steps:

<ol>
  <li>Create a <code>process</code> directory:
    <pre><code> $ mkdir src/process </code></pre>
  </li>

  <li>Create <code>env.cljs</code> inside of <code>src/process</code> with the following contents:
    <pre><code>
(ns process.env
  "This file exists to inform the Google Closure Compiler that we expect variables
  to be defined in the process.env namespace; libraries like React require the
  process.env.NODE_ENV variable to exist, so we define them here")

(goog-define NODE_ENV "production")
    </code></pre>
  </li>
</ol>

# Step 6: Preload process.env
This is the secret sauce that makes it all work. Before we try and run any code
containing React, we need to make sure that the environment variables that
Javascript expects are present!

In the `:compiler` sections of your `:cljsbuild` builds, make the following
changes:

**dev build:**

Change `:preloads [devtools.preload]` <br/>to `:preloads [devtools.preload process.env]`

**min build:**

Add: `:preloads [process.env]`

# Step 7: Use React in your Clojurescript

Now, all the roadblocks are out of the way! Change `src/hhnnngg/core.cljs` to 
something that uses React, like:

```
(ns hhnnngg.core
  (:require [react :refer [createElement]]
            [react-dom :refer [render]]))

(js/console.log "Node enviornment is" process.env/NODE_ENV)

(def appDiv (.getElementById js/document "app"))

(render
  (createElement "h1" nil "Hello World!")
  appDiv)
```

# Step 8: Behold!

```
$ lein figwheel
```

# Conclusion
You're a wizard! But you're standing on the shoulders of giants.

The Clojurescript team has done absolutely fabulous work to bring this level of
Javascript interop to the masses in a way that leverages the Google Closure
Compiler to create smaller, better JS assets than weeks worth of Webpack tuning
has yielded for our team.

The only hurdle left to clear for Clojurescript adoption at BlackSky is the 
problem of "who is going to support this when you get hit by a bus?"
Not enough believers in the magic of Lisp.  

But that's why we show *and* tell.  ✌️
