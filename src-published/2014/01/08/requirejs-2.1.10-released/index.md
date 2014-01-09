title: RequireJS 2.1.10 Released
tags: [requirejs]
comments: https://github.com/jrburke/jrburke.com/issues/19
~

[RequireJS 2.1.10 is available](http://www.requirejs.org/docs/download.html).

Mainly a maintenance release, and improves some cases when reusing code that was installed via npm. There are two new config options for the loader too:

* [nodeIdCompat](http://requirejs.org/docs/api.html#config-nodeIdCompat): some node modules installed by npm use module IDs like `example.js` and `example` interchangeably. Setting this config option to true will accommodate that style. [almond](https://github.com/jrburke/almond) 0.2.9+ also supports this option.
* [bundles](http://requirejs.org/docs/api.html#config-bundles): a more compact way to list a set of module IDs belonging to a bundle ID, and supports loader plugin resource IDs.

And for the optimizer, the [mainConfigFile](http://requirejs.org/docs/optimization.html#mainConfigFile) option can now take an array of file paths that have configs in them. Later values take precedent over earlier values.

Full list of changes:

* [Fixed require.js issues](https://github.com/jrburke/requirejs/issues?milestone=32&page=1&state=closed)
* [Fixed r.js optimizer issues](https://github.com/jrburke/r.js/issues?milestone=29&page=1&state=closed)
