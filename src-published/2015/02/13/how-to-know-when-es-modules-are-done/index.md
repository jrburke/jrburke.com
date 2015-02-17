title: How to know when ES modules are done
tags: [javascript]
comments: https://github.com/jrburke/jrburke.com/issues/30
~

There are few pieces of a module system that need to be available for it to be fully functional. I will describe them here and talk a bit about where ECMAScript (ES) modules seem to be at the moment, from an outside public perspective.

I am not on TC-39, the committee that works on the ES language specification (otherwise known as JavaScript, JS). Just someone who has worked on [a](http://dojotoolkit.org/) [few](https://github.com/requirejs/cajon) [JS](http://requirejs.org/) [module](https://github.com/requirejs/alameda) [systems](https://github.com/jrburke/module).

This is a long piece. A table of contents for the top level sections:

* [Module system pieces](#modulesystempieces)
* [Interlocking pieces](#interlockingpieces)
* [Where are we now?](#wherearewenow)
* [Hazards on the way to done](#hazardsonthewaytodone)
* [Summary](#summary)

## Module system pieces

There are three main pieces of a module system:

* [Static module definition](#staticmoduledefinition)
* [Execution-time module capabilities](#executiontimemodulecapabilities)
* [Module loader](#moduleloader)

Some might argue that these pieces are separable and could be specified by different standards groups. So an "ES module system" may not be the right term, as ES may only specify one or two pieces.

For me, they are all part of a coherent module system, so I will be referring to the future direction for them as the "ES module system", even if the URLs for each specification end up on different domains.

### Static module definition

This is how you statically declare a piece of code as a module with dependencies. In this context ***static*** means: does not change depending on the execution environment. Static dependencies can be parsed out of a module without actually running the module in a JS environment, the loader just needs to parse the text of the module to find them.

In AMD modules, it looks like this:

```javascript
define(function(require, exports, module) {
  // Statically parsable dependencies.
  var glow = require('glow'),
      add = require('math').add;
});
```

In CommonJS and Node (for shorthand's sake referred to as "CJS" for the rest of this post), there is a similar idea, just without the define() wrapper.

It is a bit more nuanced in CJS systems: the require(StringLiteral) calls are not parsed prior to execution, one of the major reasons that format is not fully suitable for a full module system on the front end, where async networking is involved. You can get some front end functionality by using something like browserify or webpack to do the static search for dependencies, but just for bundling. Fine enough for libraries but starts to break down on the app level where you want to incrementally load functionality as the user goes to use it, use a dynamic router.

In ES, it looks like this currently:

```javascript
// Statically parsable dependencies.
import glow from 'glow';
import { add } from 'math';
```

ES also statically indicates the named export keys too:

```javascript
// Statically parsable dependencies.
import glow from 'glow';
import { add } from 'math';

// Statically indicate this module will have a 'default'
// and 'other' export keys.
export default function() {};
export other funtion() {};
```

While this helps statically match up any keys given to the exported values to the ones used in import statements, the export value is not statically exported, just an indication of its name.

For AMD/CJS systems, there really is just one exported value per module, but it could be an object with multiple properties. There is no static analysis of the export value in those systems.

This part of the ES module system is the piece that is the most specified at the moment.

#### Inline modules

However, the ES system does not allow for what will be called "inline modules" for the purposes of this post. Inline modules are just the ability to statically declare more than one module in a file. This is commonly used for bundling modules together, but [has other purposes](https://github.com/jrburke/module/blob/master/docs/inlining.md).

In AMD, those are just named `define()`s:

```javascript
define('glow', function(require, exports, module) {
  return function glow() {
  };
});

define('app', function(require, exports, module) {
  // Statically declare dependencies.
  var glow = require('glow'),
      add = require('math').add;
});
```

For CJS, there are conventions for doing this via tools like browserify and webpack, but they are much less declarative. The module IDs are converted to array indices/numbers. This makes dynamic module loading harder.

For ES there is nothing for this. The last I heard, the hope was for capabilities like HTTP2 and zip bundles so that no new language syntax is needed, however I believe [that is not sufficient](https://github.com/jrburke/module/blob/master/docs/inlining.md).

In the AMD/CJS world, it has become more common to deal with nested groups of modules bundled together. An example would be some browserified base libraries that are then combined with some AMD modules in an app. The browserified ones have a conceptual inner module structure that should not be visible outside the module.

AMD and CJS do not do well with this right now. I have considered supporting something like this in my AMD loaders to allow for it:

```javascript
define(function(require, exports, module, define) {
  // The define passed in here is a local
  // define for modules only visible to
  // this module.
});
```

There are some interesting characterstics around how to define the module this way when it can have async resolved dependencies. That has been more fully explored in [this module experiment](https://github.com/jrburke/module), so I believe it can work.

The end result, I see modules now as units of code that can be nested. Similar to how functions work, but instead of identifiers for names, module ID strings are used, and their export may be resolved asynchronously, so a bit of syntax is needed for that.

The other option I have heard for ES would be to compile down the module into ES5 code, and use the ES module loader lifecycle hooks to get that into an ES6 module loader.

That option looks like a leaky abstraction. In addition, there are some tricks with the way ES6 imports are mutable slots and the syntax around getting to the execution-time module capabilities that require some extra thought.

### Execution-time module capabilities

There are some properties and capabilities that need to be exposed during the execution of a module. This means it cannot be statically determined, it is only known once the module is executing in a JS engine.

In the AMD world, the execution-time capabilities come in these forms:

* `module.id` - the module's normalized ID for the module loader instance.
* `module.url` - the URL or path to the module.
* `require([aJsStringValue], function(c) {})` - dynamically get a handle on modules. Load them if need be, but in the loader that loaded this module. Should also handle relative IDs.
* `require.toUrl('./d')` - Converts module ID './d' to a full path, useful for assets related to the module, like images or style sheets.

In Node:

* `module.id/module.url` - they end up being the same, but conceptually should be different.
* `__filename` - could be derived from module.url.
* `__dirname` - could be derived from module.url.
* `require(aJsStringValue)` - dynamically, synchronously load and execute module in the loader that loaded this module.
* `require.resolve(AJsStingValue)` - converts module ID to a path.

(Synchronous return from that dynamic require is one of the reasons the CJS system is not the right fit for a general purpose front end module system in the browser.)

In ES, this piece is not formally specified yet. In the ES world, I believe this is referred to as the "module meta", if you come across that phrase. The most recent hint of how it might be done in ES looked something like:

```javascript
import local from this;
console.log('Normalized module ID is: ' + local.id);
console.log('Normalized module URL is: ' + local.url);
local.import(aJsStringValue).then(Function(someModule) {});
```

I am making up the name of the properties for id, url, and import. I am not sure what their real names will be, just that `from this`, or some `from`-based form, was being considered as the way to aquire this functionality.

### Module loader

This is an API that runs at execution time. It kicks off module loading, and allows ways to resolve module IDs to paths, handles the loading and proper execution order of the modules, caches the module values.

In AMD, the main module loader API is `require([String], function(e) {})`. There is usually something like `require` for top-level, default loader loading, and each module can get its own local `require`. Some AMD loaders can create multiple module loader instances.

It is common for AMD loaders to support the idea of a loader plugin, a module that provides a `normalize` and `load` method that are plugged in to the AMD loader's `normalize` and `load` lifecycle steps.

This allows extending the base loader to handle transpiled code without requiring plugins to be loaded up front, before main module loading starts.

In CJS, `require(String)` is the main API to the module loader. There is a way to extend the loader capabilities via `require('module')._extensions['.fileExtension'] = function() {}`. This requires the extension to be installed before modules that depend on it are loaded. This works fine in Node's synchronous module execution environment, but does not translate to async loading in the browser.

For ES, this part is still being defined. There was a [previous sketch for it](https://github.com/jorendorff/js-loaders), but it seems like that is being redone now. I do not feel it is useful to link to the current attempt at the sketch because it is incomplete, and they likely want to work on it themselves to get it in a more usuable state before getting a lot of feedback about it.

The previous sketch did have the concept of a module loading lifecycle, and a way for userland code to plug in to that lifecycle, and I can see this concept carrying forward in some fashion:

* `normalize`: normalize module IDs.
* `locate`: translating an ID to a URL/path.
* `fetch`: getting the text contents of the module from the URL/path.
* `translate`: allows translating the text into JavaScript that can be executed.
* `instantiate`: allows a way for legacy module syntaxes to be converted into a form usable by the ES loader.

The granularity of these steps are better than the ones in AMD loader plugins, which just have a concept of `normalize` and `load`. `load` is really `locate`, `fetch`, `translate`, `instantiate` in one method. It would be good to have more granular steps.

However, there was no built in way in the ES loader to know how to load the hooks as part of normal module loading.

For AMD systems, module IDs of the form `pluginId!resourceId` meant the loader would load the module for `pluginId`, then wire it into the loader lifecycle, then delegate to that plugin's lifecycle methods for IDs that begin with `pluginId!`.

That approach avoids a two-tiered loading system in a web page where the all the loader plugins are loaded first, and then continue with the rest of module loading. The two tiered approach is slower and breaks encapsulation. Any package that used a loader plugin would need to somehow get the plugin registered in the correct loader instance up front. It also gets tricky if those loader plugins have regular JS module dependencies.

## Interlocking pieces

While the three pieces of a module system could in some way be considered separate, they all have interlocking pieces, and those pieces need to fit well together.

### Module IDs

The rules around the module IDs needs to be understood for the pieces to work well together. If someone is just working with the static module definition part and just uses a plain path for the ID, that will likely conflict with the module loader part, since the IDs should be separate string concepts from paths to support conceptual string namespaces for things like loader plugins and packages that do not have direct path equivalents.

### Loader extensions

This is tied a bit into the module ID coordination, but also involves module loader load order and how much a given module needs to know about how loader extensions (like transpilers) get wired into the system.

One option is to say that is something that is configured and wired up separately from the modules themselves, out of band, like via package config and some coordinated way to get those registered with a loader up front. This breaks encapsulation though, and makes it hard for the plugins to use modules for their own dependencies. The loader plugin approach in AMD is a much more sane way to go about it.

### Execution-time module capabilities vs static module definition

In the ES sketch above, `from this` for the execution-time module capabilities is a specific language construct that needs to be built into the static module definition.

### Loaders and execution-time module capabilities

The execution-time module capabilities also relate to methods on the module loader, like the capability to dynamically load code.

## Where are we now?

I believe the plan is for the ES6 spec is to just contain the static module definition piece, and for the other bits to be specified in separate specifications coming later.

The trouble is people are starting to use the static module definition piece via transpilers, but without having the other interlocking pieces sorted out.

The transpilers often just compile down to AMD or CJS modules for actual use, and these have some differences with the likely final ES plan. The main issues are:

### Module IDs are not sorted out

AMD has a stricter separation to module ID vs path, where CJS as practiced in Node is more file path based. IDs really need to be different things than paths. For regular JS modules, they can have an easy simple transform to a path, but need to be conceptually different.

### Export models are different

The ES export module is different than AMD/CJS. In ES all exports are named. The name `default` just has some extra syntax sugar for `import`, but no sugar when the module is referenced via the execution-time module capabilities. Expect to be typing `.default` for that.

AMD/CJS exports are really just single exports, but those systems are nice enough to create an export object if you want to use the `exports.foo = ''` form of adding properties to the export object.

### No execution-time module capabilities

There is no ES specification for the execution-time module capabilities. So there is no way with the ES syntax and APIs to build a dynamic router. You will need to know the AMD/CJS system you are using underneath to do that part.

What is meant by "dynamic router"? A module that looks at a piece of runtime path information (typically a URL segment), then translates that to a module ID for a view and dynamically loads that view via module APIs (either `require([varName])` in AMD or `require(varName)` in CJS).

Dynamic routers are really handy to avoid loading all possible routes and views up front, helps with performance.

Using the module ID via `module.id` is useful in cases where there are global spaces, like the DOM, and the module wants to construct class names, DOM data that will be in that global space. Basing its values on the module ID helps scope selectors and data access for that module.

### No static definition to allow inline modules

This is a big missing piece in ES. Right now, expect to use AMD/CJS approaches here.

## Hazards on the way to done

So, do not consider the ES module system done that with the publication of the ES6 spec. It just has one part of the system, and in many ways the most straight-forward piece. It is somewhat complicated by all the forms for `export` and `import`, but that was a design choice given TC-39's goals.

The real action comes with the module loader parts: if that is worked out, you might be able to skip the ES6 static definition parts.

So hopefully the other parts of the module system will come along. Some hazards to avoid on the way:

* Module IDs are not kept separate from paths/URLs, or if the loader uses paths/URLs instead of IDs for its internal keys.
* No solid story for inline modules.
* You need a script loader bootstrap even for the most basic wiring of all of the three pieces together.
* The module loader needs a two-stage loading mechanism where the first stage loads up all possible loader lifecycle plugins before starting the second stage to load regular modules.
* The `<module>` tag in HTML comes into existence before the module loader. Keeping with the extensible web philosophy, the module tag should just fall out of the module loader primitive and perhaps custom elements.
* [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) configuration is required to use plain JS modules across domains. CORS should only be required if the `translate` loader lifecycle hook wants to look at it. Otherwise, the `translate` hook is skipped for those scripts.

## Summary

Making a module system for ES is hard, and it is not done yet. I wish the process would have been different to date with more dialog outside of TC-39. However, it seems like the people working on it are just not done with all the pieces. I can appreciate it is hard to talk about it until the fuller picture is worked out.

The unfortunate part for me is seeing people starting to use the ES6 static module definition and transpiling to ES5 module systems to ship code. I think it is just too early to do that.

In the grand tradition of languages that can transpile to JS, you can get something to work and ship code to users. You can use CoffeeScript too. So if you are having fun with the transpiling route, that is great. Just know the sharp edges.

You are adding another layer of abstraction on top, and in the case of modules, you will likely need to directly use or know the properties of the ES5 module system you are using underneath to get the full breadth of module system functionality.

For me, fewer layers of abstraction are better. I will be waiting until more of the ES pieces are defined and shown to work well together before considering them done and using it to ship code.

