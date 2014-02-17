title: RequireJS 2.1.11 Released
tags: [requirejs]
comments: https://github.com/jrburke/jrburke.com/issues/20
~

[RequireJS 2.1.11 is available](http://www.requirejs.org/docs/download.html).

Some bug fixes, with the most notable addition is an optimizer option, `wrapShim`. This will wrap shimmed dependencies in a define() call so that they work better after a build when their upstream dependencies are also AMD modules with dependencies.

The most notable case is probably when using an AMD-aware version of Backbone, but using shim config for scripts that depend on Backbone. If this is your use case, then setting `wrapShim: true` in the optimizer config will likely fix any post-build problem you might see. More details in [the bug ticket](https://github.com/jrburke/r.js/issues/623).

Full list of changes:

* [Fixed require.js issues](https://github.com/jrburke/requirejs/issues?milestone=33&page=1&state=closed)
* [Fixed r.js optimizer issues](https://github.com/jrburke/r.js/issues?milestone=30&page=1&state=closed)
