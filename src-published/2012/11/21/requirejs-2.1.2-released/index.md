title: RequireJS 2.1.2 Released
tags: [requirejs]
comments: https://github.com/jrburke/jrburke.com/issues/7
~

[RequireJS 2.1.2 is available](http://www.requirejs.org/docs/download.html).

The big changes for this release are in the optimizer:

* The optimizer can now [run in the browser](http://www.requirejs.org/docs/optimization.html#requirements), to enable web-based custom builds of your library.
* "uglify2" is an allowed "optimize" value now, using UglifyJS 2.1.11.
* Experimental support for [source maps](http://www.requirejs.org/docs/optimization.html#sourcemaps).
* The optimizer runs faster now, and has some [speed options](http://www.requirejs.org/docs/optimization.html#turbo).

Full list of changes:

* [Fixed require.js issues](https://github.com/jrburke/requirejs/issues?milestone=22&sort=created&direction=desc&state=closed)
* [Fixed r.js optimizer issues](https://github.com/jrburke/r.js/issues?sort=created&direction=desc&state=closed&page=1&milestone=20)
