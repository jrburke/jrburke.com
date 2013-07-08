title: RequireJS 2.1.7 Released
tags: [requirejs]
comments: https://github.com/jrburke/jrburke.com/issues/16
~

[RequireJS 2.1.7 is available](http://www.requirejs.org/docs/download.html).

The main changes for this release:

* For xpcshell, the optimizer uses the built in Reflect parser API instead of Esprima. xpcshell, on Linux and Windows in particular, has a constrained stack, and normal Esprima use was not possible. To accommodate this change, some of the parsing approaches used internally by r.js moved away from token scanning to tree walking. The only visible output change you may see is different use of space characters in transformed code.
* The source map support was updated to use the new //# syntax as specified by the spec. This change is still making its way through the browsers, so if you need source map or sourceURL support with 2.1.7, you may need to use Firefox Aurora or Chrome Canary channels. The browser support levels should get better in around six weeks time.

Full list of changes:

* [Fixed require.js issues](https://github.com/jrburke/requirejs/issues?direction=desc&milestone=28&page=1&sort=created&state=closed)
* [Fixed r.js optimizer issues](https://github.com/jrburke/r.js/issues?milestone=26&page=1&state=closed)

