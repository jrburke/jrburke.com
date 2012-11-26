title: volo 0.2.5 released
tags: [volo]
comments: https://github.com/jrburke/jrburke.com/issues/8
~
This update for [volo](http://volojs.org) is mostly about working well with the new [GitHub rate limit changes for unauthenticated requests](http://developer.github.com/changes/2012-10-14-rate-limit-changes/). 0.2.5 will use GitHub auth tokens if the non-token API limit has been reached.

The other benefit with this approach, you can now pull down private repos via `volo create` and `volo add`!

It is strongly recommended you upgrade to 0.2.5:

    npm install -g volo

[List of fixed 0.2.5 issues](https://github.com/volojs/volo/issues?milestone=8&page=1&state=closed).