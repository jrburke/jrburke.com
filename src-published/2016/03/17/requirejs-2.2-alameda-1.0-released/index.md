title: RequireJS 2.2, alameda 1.0 released
tags: [requirejs]
comments: https://github.com/jrburke/jrburke.com/issues/35
~

* [RequireJS 2.2](http://www.requirejs.org/docs/download.html)
* [alameda 1.0](https://github.com/requirejs/alameda/releases)

RequireJS has a new **minor** version rev, prompted by a couple things:

1) The RequireJS project has transitioned over to the jQuery Foundation. New legal stuff for the copyright, and now just MIT license vs both MIT and BSD 2 dual license. According to the lawyer advice, MIT on its own is more permissive than BSD 2, so it should be a superset of all the possible use cases when BSD 2 might be used in the consuming project. Same permissable, free use as before, just simpler on the license front.

If you want to contribute to the codebase, you can [sign the jQuery Foundation CLA](http://requirejs.org/docs/contributing.html), which is generally useful. It clears the path for contributing to all of the other great jQuery Foundation projects.

2) Some larger changes for some new features. It should all be backwards compatible, but with the addition of the new features, it felt appropriate to indicate this was more than a patch version release.

This is not the same 2.2 release that I had in the planning phases for a couple of years, just a recalibration on existing needs, and wanting to better follow <a href="http://semver.org/">semver</a>.

## [Notable RequireJS loader changes](https://github.com/requirejs/requirejs/issues?q=milestone%3A2.2.0+is%3Aclosed)

### [urlArgs can be a function](http://requirejs.org/docs/api.html#config-urlArgs)

This allows finer tuning of querystring arguments, for things like different version strings for different files.

### [plugin IDs for data-main](https://github.com/requirejs/requirejs/issues/1472)

This sort of data-main value works now:

```html
<script src="require.js" data-main="lib/bootstrap!app/main"></script>
```

This assumes the baseUrl is the directory that holds the HTML document, in order to find the loader plugin.

## [Notable r.js optimizer changes](https://github.com/requirejs/r.js/milestones/2.2.0)

### [Faster!](https://github.com/requirejs/r.js/pull/900)

Thanks to the great work by [@petersondrew](https://github.com/petersondrew), build times are noticeably faster. Your mileage will vary depending on the type of project, but the r.js test suite runs almost twice as fast under Node.

### Uglify2 is now the default minifier

Before 2.2, `optimize: 'uglify'` used UglifyJS 1.3.x, where `uglify2` used UglifyJS 2. This was a result of the optimizer just being around a long time, during the uglify transition to their version 2 codebase. Now that version 2 is the standard UglifyJS codebase, the use of version 1 as a default was confusing.

So version 1 was removed from the optimizer, and now just version 2 is used. No need to update your build config. If you use `uglify2` as the optimize value, it will still work. Now the `uglify` value uses version 2 too.

### [Generate bundles config](http://requirejs.org/docs/api.html#config-bundles)

If you want to use [bundles config](http://requirejs.org/docs/api.html#config-bundles), the optimizer can now generate the bundle config and insert it into a requirejs.config() call via the `bundlesConfigOutFile` config option.

## Alternatives

I am mostly trying to keep the RequireJS codebase fairly static, no big changes.

If you do want to use loader that has a more modern, smaller codebase, try [alameda](https://github.com/requirejs/alameda). That is the codebase I would like to maintain for an AMD loader going forward, and where I would consider larger structural changes.

If you want to do fancier custom build processes, look at [amodro-trace](https://github.com/amodrojs/amodro-trace). It provides a programmatic API for use in Node programs for just tracing and normalizing AMD modules.

## Past and future

The primordial parts of the RequireJS loader started to take form in September 2009, with many usable releases through October 2011, when 1.0 was tagged. It is now 2016.

So, 6 & 1/2 years. I was hoping the need for these userland loaders would just last for a few years then a native loader and module system would obsolete the need for them. Funny how our desires for the future often outpace reality.

I am still hopeful that will happen at some point, so I am not investing a lot of energy to reimagining how an AMD loader might work. I would like to see a native module system actually ship before seeing if it made sense to make new tools around it.

That said, I like the smaller, modern core of alameda. I look forward to using it if a project calls for an AMD loader, although I am happy to work on projects that would not require it.

I want to keep the RequireJS codebase fairly stable. I still expect to do occasional patch version releases for RequireJS, maybe 2-3 a year. Particularly since alameda has a higher barrier to use: promise support in the browser and IE10+ for script loading. As time moves forward though, those constraints should be less of a concern.

I will not post here about every point release for the loaders. These places track the releases:

* [RequireJS release notes](http://requirejs.org/docs/download.html#releasenotes)
* [alameda releases](https://github.com/requirejs/alameda/releases)

Thanks for your time and use of the libraries. The goal for them was to be useful, help inform the future. They have lived longer than I would have wagered at the beginning. Here's to hoping they are not around for another 6 & 1/2 years! Please hasten their retirement by finishing a native JS module system!
