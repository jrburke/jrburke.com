title: RequireJS 2.1.9 Released
tags: [requirejs]
comments: https://github.com/jrburke/jrburke.com/issues/18
~

[RequireJS 2.1.9 is available](http://www.requirejs.org/docs/download.html).

Mainly a maintenance release to fix bugs. There is a new [skipDataMain](http://requirejs.org/docs/api.html#config-skipDataMain) option in require.js to avoid the data-main work, which can be useful for browser extensions that should let the main content page's requirejs handle the data-main.

Full list of changes:

* [Fixed require.js issues](https://github.com/jrburke/requirejs/issues?milestone=31&page=1&state=closed)
* [Fixed r.js optimizer issues](https://github.com/jrburke/r.js/issues?milestone=28&page=1&state=closed)
