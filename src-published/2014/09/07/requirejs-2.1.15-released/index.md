title: RequireJS 2.1.15 Released
tags: [requirejs]
comments: https://github.com/jrburke/jrburke.com/issues/24
~

[RequireJS 2.1.15 is available](http://www.requirejs.org/docs/download.html).

Mainly fixes a regression from 2.1.14 in the r.js optimizer where [some define() calls were not found](https://github.com/jrburke/r.js/issues/704). The most common manifestations of the bug would be either an extra `define('jquery', function(){})` in the build output or namespaced builds not working. The fixes for 2.1.15 are just in the optimizer. Full list of changes:

* [Fixed r.js optimizer issues](https://github.com/jrburke/r.js/issues?q=milestone%3A2.1.15+is%3Aclosed)
