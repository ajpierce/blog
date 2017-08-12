---
title: "ClojureScript: NPM in Figwheel in 60 Seconds"
date: 2017-08-11T17:11:21-04:00
draft: false
---

The ClojureScript team continues to delight!

The [last time](/posts/react-figwheel-npm/) we set up a new [Figwheel](https://github.com/bhauman/lein-figwheel)
project to demo Javascript/ClojureScript interop, we had to:

+ Build ClojureScript from Scratch
+ Update dependency versions in the Figwheel template
+ Shim `process.env/NODE_ENV` so the `:closure-defines` could find it
+ Add our shim to `:preloads` so NPM libs could reference `NODE_ENV`

Now, none of this is necessary!

Starting a ClojureScript/Figwheel project from scratch and configuring it to use
Javascript modules from NPM is now a 1 minute process. *One minute!*

## Step 1: Start a new Fighweel Project

```
$ lein new figwheel splort
```

## Step 2: Modify project.clj

We need to add NPM dependencies, install them, and tell the closure compiler what
our NODE_ENV is.  We do that by adding the following three lines to the `:compile`
section of our `"dev"` build in `:cljsbuild`:

```
:closure-defines {process.env/NODE_ENV "development"}
:npm-deps {:react "15.6.1" :react-dom "15.6.1"}
:install-deps true
```

Then, in the `"min"` section:

```
:closure-defines {process.env/NODE_ENV "production"}
:npm-deps {:react "15.6.1" :react-dom "15.6.1"}
:install-deps true
```

## Step 3: Use React (natively!) in ClojureScript

Paste the following into `src/splort/core.cljs`:

```
(ns splort.core
  (:require [react :refer [createElement]]
            [react-dom :refer [render]]))

(js/console.log "Node enviornment is" process.env/NODE_ENV)

(def appDiv (.getElementById js/document "app"))

(render
  (createElement "h1" nil "Hello, World!")
  appDiv)
```

## Step 4: Build it!

```
$ lein figwheel
```

If everything goes well, the site should render like so:

<img src="/react-figwheel-npm2.png" />

## Conclusion

It hasn't even been two weeks since the [first](/posts/react-figwheel-npm/)
Figwheel/NPM interop tutorial went live, and the ClojureScript community has
already reduced the friction for JS interop by an order of magnitude.

I don't think it can be overstated that you get all the benefits of the Google
Closure compiler, out of the box, for free.  Dead code elimination, Javascript
compilation, better production assets than you can get with hours of Webpack
tuning, and with all the benefits of Clojure to accompany it.

## With Gratitude
All of this is made possible thanks to the hard work of the ClojureScript
community.

Special thanks to:

+ [David Nolen](https://twitter.com/swannodette) for submitting [CLJS 2280](https://github.com/clojure/clojurescript/commit/537f60c975a29983c62647b4ea67b0bc08979366) mere hours after reading the previous tutorial
+ [Bruce Hauman](https://twitter.com/bhauman) for his patience and guidance with [PR 588](https://github.com/bhauman/lein-figwheel/pull/588)
+ [Mike Fikes](https://twitter.com/mfikes) for being the first person to retweet the first tutorial, giving it the visibility necessary to affect positive change

I've had such a positive experience interacting with members of the ClojureScript
community. I'm convinced that it's the excellent people (in addition to the
excellent tooling) that makes Clojure such a delightful language to work with.
