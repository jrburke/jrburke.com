title: RequireJS 2.1.4 Released
tags: [requirejs]
comments: https://github.com/jrburke/jrburke.com/issues/10
~

[RequireJS 2.1.4 is available](http://www.requirejs.org/docs/download.html).

Quick release for a bug that slipped in the 2.1.3 release in the r.js optimizer. So even though require.js now has a 2.1.4 version, it is the same as 2.1.3, and the optimizer is the same as 2.1.3 except for this one fix:

* [Bug 356](https://github.com/jrburke/r.js/issues/356): cssPrefix normalization always needs to happen

Without this fix, in some cases 2.1.3 would insert "undefined" in some optimized CSS files, making them unusable.