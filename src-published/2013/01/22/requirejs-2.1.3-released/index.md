title: RequireJS 2.1.3 Released
tags: [requirejs]
comments: https://github.com/jrburke/jrburke.com/issues/9
~

[RequireJS 2.1.3 is available](http://www.requirejs.org/docs/download.html).

Maintenance release. A change that may be noticeable:

[require.toUrl()](http://requirejs.org/docs/api.html#modulenotes-urls) now correctly generates URLs for string values passed to it without an extension. Previous versions of toUrl() would append a ".js" extension automatically. If you relied on that behavior, when you update to 2.1.3, then you may need to do a code change to append the .js extension yourself:


    require.toUrl('some/value') + '.js'

The text plugin has been updated to also work with this change, so if you want to generate non-extension paths for text resources, be sure to upgrade to the [2.0.4 version of text.js](https://raw.github.com/requirejs/text/latest/text.js).

Normal use of toUrl with a value that has an extension continues to work the same.

Full list of changes:

* [Fixed require.js issues](https://github.com/jrburke/requirejs/issues?milestone=24&sort=created&direction=desc&state=closed)
* [Fixed r.js optimizer issues](https://github.com/jrburke/r.js/issues?sort=created&direction=desc&state=closed&page=1&milestone=21)
