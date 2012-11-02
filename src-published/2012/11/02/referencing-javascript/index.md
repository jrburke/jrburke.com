title: Referencing JavaScript
tags: [mozilla, javascript]
comments: https://github.com/jrburke/jrburke.com/issues/6
~

Referencing JavaScript. How should it be done? Is there an approach that works for module systems, package managers and project layouts?

The following describes the approach in use by [AMD](https://github.com/amdjs/amdjs-api/wiki)/[RequireJS](http://requirejs.org) and [volo](http://volojs.org), but parts are found in other places like [Node](http://www.nodejs.org/), [CommonJS](http://www.commonjs.org/) and [Dojo](http://dojotoolkit.org/).

The following approach works well for browser-based, web development. Since browser-based development is the harder environment to support, the approaches can carry over well to other JS environments. Other environments have the capability to add more power to this approach, but it would be nice to see a baseline available across all JS environments.

If you work on module systems, package managers/tools or project layouts, please consider this approach. There could be other ways to accomplish the same end result, but the following approach is cohesive and have been proven out in practice.

## Summary

The high level summary:

* **[Defaults for single file libraries](#defaultsforsinglefilelibraries)**. This encourages small reusable modules, and leads to a simpler project layout.
* **[Reference code using string IDs](#referencecodeusingstringids)**. These IDs are not paths, but have an easy convention to map to paths. Treat the IDs as an ID namespace, not as path entities.
* **[Use a simple ID-to-path convention](#useasimpleidtopathconvention)**. baseUrl + ID + ".js".
* **[Allow declarative configuration](#allowdeclarativeconfiguration)**. Some things do not fit in convention.

More detail below, and how these points affect module systems, package managers/tools, and project layouts.

## Defaults for single file libraries

The ideal project layout should be a collection of single file JS libraries that each do one thing well. Ideally the functionality is exported as a single value, commonly a function.

There will be libraries that want to do a few things and provide a few modules in one package. Those should be possible to support, but the defaults are optimized for single file JS libraries.

If the defaults and conventions are around single file JS libraries, it will give a nudge to consider that direction. It is also simpler for new people to understand. With the configuration mentioned later, the system can grow as they grow.

### Single file libs: module systems

Allow exporting a single function as the module value. This works in Node by assigning the export to `module.exports`. In AMD, it works by returning a value from the factory function.

Ideally Node would have just used `return` too, but it is difficult to support in JS since `return` is not allowed at the top level of a script. Technically, it would work to use `return` in Node, since under the covers node wraps the module code in a function before evaluating it, but the source files would not lint or parse well -- source editors and tools that check syntax would complain.

For circular dependencies, there are cases where returning the value makes it hard to resolve the cycle. The `exports` concept in CommonJS is useful for those cases, so that kind of capability should be available. However, the majority of modules in AMD and Node just set the module value, so there should be an easy way to just set the value, and in particular allow function values for modules.

### Single file libs: package managers

Allow registering single file JS files as a "package". For communicating module metadata, allow an inline `/*package */` comment to express the JSON metadata instead of mandating a separate package.json file.

volo supports this already. [npm](http://npmjs.org/) supports this in a limited fashion, and I believe [Bower](http://twitter.github.com/bower/) uses the same [metadata extraction library](https://github.com/isaacs/read-package-json) as npm.

When installing a package into a project, just install the single JS file as the name given with a ".js" extension. So `install foo` should result in just a foo.js on disk, instead of creating a folder just for foo.js.

This is important for the "simple convention" approach later. It is also just cleaner.

npm could improve the state of its installation artifacts. Do not encourage projects to package a readme or tests such that those things show up installed in a project. This is unnecessary noise that is not needed once the dependency is installed.

There needs to be a way to get more info on a dependency, but that should be fetched via commands to npm, not having to scan extra local file debris. If someone wants to run tests in their project, querying npm to find or install the source repo is a better way to go about this.

For package managers in general, there needs to be a way to allow a directory as the package install. There are cases that have binary components, and even richer packages that are made up of more than one module.

However, the defaults should make it easy to use single file JS files. This is something Bower could improve on. Currently Bower creates a directory for each dependency, and every dependency directory has an "index.js" in it. This is bad for debugging and creates a bunch of directory and file debris that is not useful for running the application.

If a package manager needs to track metadata for a dependency, either rely on inline `/*package */` metadata, or use a project's package.json file to store the dependency ID/version number/location, and then use that later to look up info about the dependency.

### Single file libs: project layouts

Projects should be able to specify a directory in which dependencies should go. More on this in the convention and configuration sections.

## Reference code using string IDs

This is how it is done in CommonJS/Node and in AMD/RequireJS. So to use jQuery as dependency, specify "jquery" as the dependency ID. Do not include the ".js" extension. This allows for different path resolutions.

In Node, it may result in a nested directory IO lookup, and reading of a package.json "main" property. In AMD solutions, it may simply map directly to a 'jquery.js' under a "baseUrl", or be subject to some declarative configuration.

IDs should allow relative references. So, a module with ID of "some/thing", can use "./other" to refer to the "some/other" module.

**Module IDs are not path entities**. They are keys in a module ID space. This is a subtle point, and it creeps up once multiple modules are inlined into one file, either for library distribution or for optimization purposes. Those IDs should be maintained as module IDs, and not as path entries. This is something I think is missing in Node, where internally it uses paths as the underlying key for a module. This carries over to how tools like [browserify](https://github.com/substack/node-browserify) inlines modules.

Since module IDs are not paths, it means resolving relative IDs using a reference module ID and not a path.

### String IDs: module systems

Use strings as IDs when referencing and defining modules. Always use those string IDs, even in built source form. This allows for supporting loader plugins.

[Loader plugins](http://requirejs.org/docs/plugins.html) have been incredibly useful for AMD loaders, and I believe they are better than the file extension-based system that Node uses currently, since loader plugins are more direct in usage. Their use is controlled by the caller, not by some module acting at a distance.

For example, a "[text](https://github.com/requirejs/text)" loader plugin can load a .html file or a .txt file, and a "mustache" loader plugin could also handle the same .html file as the text plugin. How the file will be used is subject to the caller's context, not the file extension.

Loader plugins can do conditional logic before satisfying the dependency. This opens up feature detection-based dependencies that are expressible as string IDs and participate in optimization/build steps.

Loader plugins are a great way to implement transpilers too. Examples from the AMD/RequireJS world: [CoffeeScript](https://github.com/jrburke/require-cs), [TypeScript](https://github.com/iammerrick/require-ts).

### String IDs: package managers

Since there is a layer of indirection between the module ID and the path, this opens up some flexibility on install location. However, as mentioned in the path convention section below, using a "baseUrl" convention is useful.

### String IDs: project layouts

The string IDs are useful when combined with the default ID-to-path convention for keeping IDs short, but easy to map to locations on disk. No need to type out the directory and ".js" for each dependency. Just look in the directory used as the "baseUrl" to find all the dependencies.

## Use a simple ID-to-path convention

Module IDs are not paths, but it should be easy to map an ID to a path.

This should just be baseUrl + [module ID] + ".js".

Why is ".js" given special status? Because this is about loading JavaScript, and .js is already the recognized file type for JavaScript. Loader plugins allow referencing of non-JS files. The loader plugin just has to turn the dependency into some sort of JavaScript result.

For JS endpoints that are URLs that do not end in ".js", but deliver JavaScript, those can be configured via the declarative configuration. These are rare, and they may need a loader plugin to properly load if they have multiple async cycles to fully complete, like the Google APIs.

Since libraries are encouraged to be single JS files, this simple convention makes it easy for anyone to guess where the dependency exists in a default layout. If a developer is looking inside **www/js/bar.js** and it references "foo", then odds are it is at **www/js/foo.js**, or somehow tied to **www/js/foo**.

It is easier for new developers to start with JS: "just drop all scripts in this directory and refer to them without the '.js'". It also helps enforce single purpose, single file libraries.

### Path convention: module systems

This approach means there is one IO lookup per module ID. This is important on the web, where trying multiple file/URL locations for a module ID would lead to terrible performance and confusing 404 errors in developer tools.

This also means avoiding looking up modules in a global file location. This is not really an issue for browser environments, everything is "local" to the project. However, for non-browser JS environments, using a global file location to look up modules for specific projects is not recommended.

Node's local node_modules approach is the right direction to go, versus the Python and Ruby models that favor the global space more. Most of my headaches when using Python and Ruby stem from this choice. While virtual environments are a way to work around that, it is a non-default behavior and a continual tax to use them.

### Path convention: package managers

The biggest outcome of this approach is avoiding a global installation space, and to install all dependencies locally within a project.

This is done in npm and Node. While npm allows installing into a global directory, those packages are not visible to a specific project's require calls. So those kinds of installs are mostly useful for command line tools that are built using Node.

The package manager should be able to read the "baseUrl" location either by a convention with project layout or via an explicit metadata property in the project's package.json. "baseUrl" is recommended as the property name, since it ties in with the runtime configuration of a module loader.

volo supports a "baseUrl" property in the package metadata, and if specified, will install all dependencies in that directory. If no baseUrl property, then volo will use one of these locations:

* **js** directory, if exists
* **scripts** directory, if exists
* current working directory

There are some dependencies that will want to be directories, since they may have binary components, or they are larger libraries that provide a few related modules.

These directory-based libraries can be supported in one of two ways:

1) Allow updating a config object that is used at runtime with extra info. [Jam](http://jamjs.org/) does this. See the declarative config section below for more info on config values.

2) Install a proxy module that points to the directory's "main" module. So if installing a package "foo" that is a directory and its package.json has a `"main": "index"` property, use this directory layout:

* **foo.js** (created by the package manager)
* **foo/** (fetched from the package repository)
    * index.js

And **foo.js** just requires "foo/index" and returning the "foo/index" module value. Volo takes this route.

### Path convention: project layouts

Use the project's package.json to store a property that indicates the baseUrl. Calling it `baseUrl` matches what the runtime module system will use.

I favor the project layout as specified in [volojs/create-template](https://github.com/volojs/create-template):

* **tools**: directory to hold local project/build tools and assets.
* **www**: the source directory, what is served up by a web server in dev.
    * **js/lib**: the "baseUrl" used by the project.
* **www-built**: the built version of the project. It should be possible to do most development in the www directory, and when ready for deployment and optimization, trying out the built version in www-built.

The package.json for that project layout uses `"baseUrl": "js/lib"` to indicate the the place to install dependencies.

## Allow declarative configuration

Not all styles fit into the default convention. We have seen this in AMD, and AMD loaders are working towards a [common set of config options](https://github.com/amdjs/amdjs-api/wiki/Common-Config).

By allowing a declarative configuration, it removes the need for additional "helper libraries" that would likely be needed in the browser to configure an imperative hook for ID resolution.

### Declarative config: module systems

Support a declarative set of configuration values that are used at runtime, like the ones in the [AMD Common Config](https://github.com/amdjs/amdjs-api/wiki/Common-Config):

* **baseUrl**: sets the baseUrl for the ID-to-path conversions.
* **paths**: configures a base path for a module ID prefix.
* **map**: maps IDs used by some ID prefixes to other ID prefixes. It may be worth renaming this to "alias" to make it clearer how it differs from paths.
* **config**: passes some configuration data to a particular module ID. This is useful for passing data that changes depending on the environment or context. User IDs and API end points are good examples.
* **packages**: configures CommonJS/Node style "package" directories with a "main" property without using an adapter/proxy module.

For supporting code that may not actually call the module system, RequireJS users have found [shim config](http://requirejs.org/docs/api.html#config-shim) to be useful.

A module system may still have an imperative hook to override final ID-to-path resolution. I can see this useful for environments like Node: use the declarative config, and if that does not translate to a file that exists, then fall back to the "scan nested node_modules directories" approach.

However, given the IO restrictions for browser environments, a declarative config should be available as a default.

### Declarative config: package managers

Stamp the project config if installing a dependency would require a configuration entry. Examples: support "directory packages" or to set up a map/alias so that certain modules use version 1 of a library while the rest of the modules in the project use version 2.

To do this, the package manager needs to know the location of the configuration.

While this is not implemented yet in volo, I favor piggybacking on a "baseUrl" that is specified in a project's package.json file. By default, assume the config is in baseUrl + 'main.js'. However, allow for a configuration override via a package.json property. Perhaps call it "mainConfigFile", as [the r.js optimizer does](http://requirejs.org/docs/optimization.html#mainConfigFile), and it would be a path within the project to the JS file that has the configuration.

The configuration will be passed to a runtime JS module API call, so to be able to properly extract the config, update it, and save it back, the package manager needs to handle updating a file that is written in JavaScript. This is possible by using tools like [Esprima](http://esprima.org/) or [Acorn](http://marijnhaverbeke.nl/acorn/).

### Declarative config: project layouts

The baseUrl approach works well for getting started, but many projects like to keep third party dependencies separate from the application code. For that kind of project layout, set the baseUrl to a **lib** directory to hold third party code, and then specify an "app" paths config for an **app** directory that is a sibling to **lib** for the app code.

The [volojs/create-template project template](https://github.com/volojs/create-template) does this in the main [app.js file](https://github.com/volojs/create-template/blob/761945fac2557bc50e22bcae4984305a03e0f45d/www/js/app.js). The project
layout:

* **www**
    * **index.html**: uses require.js to load www/js/app.js via `data-main="js/app"`
    * **js**
        * **app.js**: loaded by index.html, has the declarative config.
        * **app**: directory to hold app code
            * **main.js**: module that runs the main logic for the app.
        * **lib**: directory to hold third party code.

where app.js holds the small configuration for the app:

```javascript
// For any third party dependencies, like jQuery,
// place them in the lib folder.

// Configure loading modules from the lib directory,
// except for 'app' ones, which are in a sibling
// directory.
requirejs.config({
    baseUrl: 'js/lib',
    paths: {
        app: '../app'
    }
});

// Start loading the main app file. Put all of
// your application logic in there.
requirejs(['app/main']);
```

Any app modules are stored in the **app** directory. The app modules refer to each other using relative module IDs.

Some projects that use RequireJS end up with large config blocks, mostly with a paths setting for each dependency. This should be discouraged.

With the lib baseUrl and the app paths setting, it gives the smallest amount of config that does not need continual updates and still separates third party code from the application's code.

One of the reasons some people use big config blocks is so they can track the version number of the dependency in the file name, like `jquery: 'lib/jquery-1.8.2'`. Instead, track this information in a project's package.json file, since it is most useful to track as part of a package manager's work, to resolve version conflicts.

## Secondary implementation notes

### project or library metadata

In the above points, a `package.json` is mentioned a few times. This is a project or library level file that holds the JSON-based metadata about the project. It is not information that is needed at runtime, but information about what the project or library needs to have installed to function at runtime.

For single file lib support, this metadata should be allowed inline in the JS file as a comment:

```javascript
/*package
{}
*/
```

While npm relies heavily on package.json and assumes certain formats for "dependencies" and requires a "name", other tools should feel free to use package.json to store metadata.

A tool-specific name in the package.json can be used to encapsulate info that is only appropriate for that tool, or while that tool is working out a common config format that later could be used by multiple tools. For instance, volo will look in a "volo" property to find volo-specific configuration details.

This should be favored over creating separate .json files. Bower does this with component.json. This adds clutter. The nature of JSON key-based storage means that tool-specific info can be stored under unique keys in the package.json. Bower could use a "component" property in the package.json instead.

One thing I wish npm did differently was to not place so much dependence on the "name" field in the package.json.

Packages naming themselves have the same problems as modules naming themselves. The name should really be specified by the context consuming the package or module.

A package could have a very different name depending on the package repo or context it is used. For example, the requirejs "text" module is a loader plugin. Claiming the "text" name would be bad to do in a general package repository. However, in the context of a loader plugin it makes more sense. If a repository holds JS packages that can be used for multiple contexts, like browser and Node, a single name specified by the package is too limiting and often misleading.

One way to solve that is to store modules for a different contexts (like browser vs Node, in different package repositories. Even then, the same package could be useful in both, but need different names in each repository.

Another approach is to allow a larger namespace in a package repo. Volo does this by using GitHub as the repo, and GitHub has a two level namespace: "owner/repo", and the name of the repo is something specified outside of the package metadata.

## Alternatives

Some approaches that may mitigate the need for the four main points discussed above:

### Everything in a directory

Do not allow single JS files, and work out a convention that is based on each dependency in a directory.

This does not carry over well once you start coding application logic, where you have multiple single purpose modules that favor the single JS file approach for each module ID. It also creates a bunch of extra noise in a project, and feels too much like Java.

This approach works in Node because it can do multiple IO lookup operations for a module ID. This does not work well for the browser case.

### Build everything

Who cares about file debris and big configuration blocks, mandate builds to clean up the mess for browsers!

This is not how front end projects have worked in the past. There is great simplicity, immediacy, and "view source" value in being able to just code some static files and load them directly in the browser.

I cannot see any future module system in ECMAScript mandating builds just to use modules in the browser. We need to figure out an approach that works without needing build tools.

Making sure a system can work without build tools helps nudge people to simpler libraries and project layouts. Once build tools become involved, it means more levels to debug and more things to set up before getting started. It leads down the path to Java and [WS-Deathstar](http://en.wikipedia.org/wiki/List_of_web_service_specifications). Keep the web simpler.

Build tools are great if you want to squeeze out every last bit of performance, but they should be optional. Many folks run AMD projects without building, and they are fine with the performance.

Build tools are also useful when trying to work out how something should work in the future, like CSS. CSS has been so neglected over the years that using LESS or SASS-like systems is alluring. Hopefully those are just intermediate efforts to better inform improvements in CSS, so that they eventually become built in to CSS itself.

If not, hopefully module systems support loader plugins so that these CSS builders can run in the browser without needing builds.

## Summary

The above approach as been demonstrated to work, and it is in active use. While the "everything is a directory" and "build everything" do not seem like viable long range solutions, it would be great to hear of other solutions that work well in practice.
