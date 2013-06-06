title: volo 0.3.0 released
tags: [volo]
comments: https://github.com/jrburke/jrburke.com/issues/14
~
A new release for [volo](http://volojs.org) is out that adds the following:

* **Colorized output**: just a bit of color. Nothing crazy. Also the markdown help files are now a bit prettier thanks to an approach I first about from [Gord Tanner](https://github.com/gtanner).
* **volo.ignore**: For your library, you can set a [volo.ignore](https://github.com/volojs/volo/wiki/Library-best-practices#wiki-voloignore) property for files/directories that should not be part of the installed dependency.
* **remove command**: There is now an explicit remove command. It will remove a dependency from the volo.dependencies in package.json and remove the file(s) on disk.
* **update command**: Just an alias to `add -f`.

Some internal dependencies were updated. One of them was [Q](https://github.com/kriskowal/q), going from a 0.8 version to a 0.9 version. As part of that update, some of its APIs changed, so if you `v.require('q')` in your volofile, you may need to update your code. The only changes I needed to do internally:

* `q.call` is now `q.fcall`. If you need to support older and newer volo installs, you can use `q.fcall || q.call)` in your volofiles to be forward and backward compatible.
* `q.end` is now `q.done`.

See the full set of [0.9 Q API](https://github.com/kriskowal/q/blob/master/CHANGES.md#090) changes.

[Full list of fixed volo 0.3.0 issues](https://github.com/volojs/volo/issues?milestone=13&page=1&state=closed).

To upgrade to the latest release:

    npm install -g volo
