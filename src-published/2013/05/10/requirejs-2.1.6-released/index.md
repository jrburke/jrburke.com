title: RequireJS 2.1.6 Released
tags: [requirejs]
comments: https://github.com/jrburke/jrburke.com/issues/13
~

[RequireJS 2.1.6 is available](http://www.requirejs.org/docs/download.html).

[Source map support](http://www.requirejs.org/docs/optimization.html#sourcemaps) has been expanded. Previously, it was just supported for going from minified, bundled code to the unminified, bundled code. If **optimize: 'uglify2'** is used, it will now go back to the separated, unbundled files.

Source map support is still considered experimental though, so you may find bugs. If you find one, file an [r.js issue](https://github.com/jrburke/r.js/issues), ideally with a test case.

Full list of changes for 2.1.6:

* [Fixed require.js issues](https://github.com/jrburke/requirejs/issues?direction=desc&milestone=27&page=1&sort=created&state=closed)
* [Fixed r.js optimizer issues](https://github.com/jrburke/r.js/issues?milestone=25&page=1&state=closed)
