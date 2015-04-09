title: amodro-trace and AMD loaders
tags: [requirejs, javascript, amodro]
comments: https://github.com/jrburke/jrburke.com/issues/33
~

A new tool, and some AMD loader rambling:

I have started a new project around AMD modules, [amodro-trace](https://github.com/amodrojs/amodro-trace). It is a tool that understands AMD modules and is meant to be used in other node-based build systems. The README has more background, but the general use cases that drove it:

* Get a dependency tree for a module ID
* Allows non-file inputs
* Allows transpiling inputs to AMD before tracing
* Allows content transforms after AMD normalization

Think of amodro-trace as a lower level imperative tool that something like the requirejs optimizer could use to implement its declarative API.

amodro-trace comes from some code in the requirejs optimizer, and has some smaller unit tests. I ran it over a larger project, but still expect to fine tune some things around API and operation, so feel free to give feedback in the issues list if the use cases fit your needs but have trouble using it.

I would also like to construct a new AMD loader, something that assumes more modern browsers and can improve on some things learned from requirejs.

I do not expect requirejs to go away any time soon, and it will still be my recommendation for general AMD loading across a wide set of browsers. There will still be maintenance releases, but I expect to do any new work that non-trivially modifies behavior to be done under a new name. This helps set stable expectations, particularly for tools that have been built on top of requirejs.

I still want to explore some things with AMD loaders though, particularly since an [operational ES module system is still far off](http://jrburke.com/2015/02/13/how-to-know-when-es-modules-are-done/), and transpilers that guess at ES module syntax still benefit from good AMD loader options to back them.

## AMD loader options

First, a bit about some AMD loader options that I have worked on. The nice thing about AMD modules is that there are more options besides this set, and other tooling around them. This is just about where and how I have spent my time in this space.

* [requirejs](http://requirejs.org/): the baseline loader. Along with its [optimizer](http://requirejs.org/docs/optimization.html), it has a fairly large set of tests and stability. (Side note: the code that would become requirejs started in 2009, first releases in 2010, with some ideas stretching back much earlier from Dojo.)
* [cajon](https://github.com/requirejs/cajon): Built around requirejs, but bundles an XHR fetch option to allow for consuming CommonJS modules directly as well as AMD modules. Comes with the caveats of using an XHR loader though, like [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) and [CSP](https://developer.mozilla.org/en-US/docs/Web/Security/CSP/Introducing_Content_Security_Policy) issues.
* [alameda](https://github.com/requirejs/alameda): Like requirejs, but assumes more modern browsers as a baseline (mainly means does not support IE9 and below), uses promises under the hood. There is a [native-promise](https://github.com/requirejs/alameda/tree/native-promise) branch that does not use a promise shim, just native browser promises. alameda is used by some [Gaia](https://github.com/mozilla-b2g/gaia) apps that use modules.
* [module](https://github.com/jrburke/module): Not AMD, but what I imagine a module system would look like given what we have learned from CommonJS and AMD modules, and assuming a browser loader capability that avoids CORS/CSP issues for text fetching and evaluation. While a useful exercise, and more operational and fleshed out than an ES system, more of an experimental playground.

## amodro loader

For a new AMD loader, I am thinking of putting it under the amodro (pronounced a-mo-dro) name. amodro-trace is the start of what I would see as its equivalent of the requirejs optimizer piece. amodro-trace currently uses requirejs under the hood for module tracing, but ideally that would migrate over time to a new loader.

I would not want to modify any of the AMD APIs for declaring a module or for the dynamic require calls. So no changes in module syntax to allow the most reuse of existing AMD modules.

However, I want to rethink some of the loader APIs and loader plugin APIs to do something like what an older draft of the ES-related loader had for a module lifecycle: normalize, locate, fetch, translate, instantiate. The loader plugin API as supported in requirejs-like loaders is not as granular, and supporting a more granular API would help with some issues that have come up with the loader plugins to date: it can be hard to break cycles for some loader plugins, and can make building more complicated.

The `module` loader mentioned above makes an attempt at that sort of solution for loader plugins, and it works out well. There is a good chance existing loader plugins would still work too since their APIs can be seen as a coarser API that could be supported by the more granular API. Still a bit of work to be done there, but it seems promising.

So I expect amodro would be like the `module` loader, but designed to work with the AMD APIs instead of the `module` API in that loader, and probably using some of the alameda ideas too.

I may not get to it though. Just sharing my thoughts around loader work. I have a day job that I really like, and we are doing some interesting work. There are some (non-loader) ideas I want to implement there, and I am excited to try out service workers in that context.

The [Dojo folks](http://dojotoolkit.org/) are also thinking about this space, as well as [John Hann](http://unscriptable.com/) and [Tim Branyen](http://tbranyen.com/), so other options may come out of their efforts too. It is good to have options.

End result, more in this space worth pursuing.

## More convention over configuration

For AMD projects in general, and something that does not depend on any new loader work:

We can help improve the perception of difficulties with configuration by starting to advocate more for standard project layouts that avoid big configuration blocks for the loader. Effort in this space would likely benefit an ES module solution too, as it will need to operate in the same async network space that AMD modules operate.

To me, that means using a starting project layout that looks like [this sample project](https://github.com/volojs/create-template). The `lib` directory could be a `node_modules` or `bower_components` directory.

[adapt-pkg-main](https://github.com/jrburke/adapt-pkg-main) can be used after an npm/bower install to fix up the installed dependency to work with the file layout convention that works best for general web module loading, without running into CORS or 404 issues.

Then hopefully the package managers get better about these file layouts over time (maybe absorb what adapt-pkg-main does), and in the case of npm, [remove some sharp edges](https://github.com/jrburke/notobo/blob/master/docs/npm-sharp-edges.md) for front end development.

## Summary

You might try [amodro-trace](https://github.com/amodrojs/amodro-trace) if its use cases fit your needs. While it comes from some code that has had a good amount of testing, it is still a new approach on it and may have some bugs, so I am keeping the version low for now. However, it is the kind of AMD build tool I would like to support longer term: provide a primitive focused solely on the AMD tracing and normalization so that others can build on top of it.

The requirejs optimizer was built at a time when node was not a thing yet, and more batteries needed to be included for command line JavaScript tooling. It has been a good approach for the requirejs optimizer: it runs in node, Nashorn, Rhino, xpcshell and even in the browser. It gives a bunch of communities a chance at some good AMD-based optimization options.

However, I do not expect to keep pace with all the possible variations in build tool styles with the requirejs optimizer's more declarative options-based approach. amodro-trace should be helpful for those cases.

Here's to more AMD loaders and tools for the future!

