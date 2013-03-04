title: RequireJS 2.1.5 Released
tags: [requirejs]
comments: https://github.com/jrburke/jrburke.com/issues/12
~

[RequireJS 2.1.5 is available](http://www.requirejs.org/docs/download.html).

Biggest change is support for running the optimizer and loading AMD modules in
[xpcshell](https://developer.mozilla.org/en-US/docs/XPConnect/xpcshell). It is a bit tricky to get xpcshell on your machine. It normally means creating a debug build of Firefox. But if you have a build or test environment built around xpcshell, RequireJS and the optimizer now run in that environment.

I tried to get [r.js to work in Windows Script Host](https://github.com/jrburke/r.js/issues/201), but unfortunately, cscript is too limited to be useful.

If someone else knows how to run JS on the Windows command line that has access to file IO natively on Windows, please let me know in that issue. However, if it means downloading some sort of binary, I am more likely to discard that approach and encourage developers to take the already available node path for Windows.

Similarly, I wish [JSC](https://trac.webkit.org/wiki/JSC) on OS X had a way to access the file system. If it did, it would be easy to write an adapter for it. However, JSC seems limited to just stdin/stdout.

Full list of changes for 2.1.5:

* [Fixed require.js issues](https://github.com/jrburke/requirejs/issues?direction=desc&milestone=26&sort=created&state=closed)
* [Fixed r.js optimizer issues](https://github.com/jrburke/r.js/issues?milestone=23&state=closed)
