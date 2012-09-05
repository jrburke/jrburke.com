title: New Blog
tags: [meta]
comments: https://github.com/jrburke/jrburke.com/issues/1
~
I will do new posts from this site, [jrburke.com](http://jrburke.com/). There is
a feed link in the header if you want to track this site in an feed reader.

I was using [tagneto.blogspot.com](http://tagneto.blogspot.com), but
I have wanted something different for a while now.

## Motivation

* Write posts in Markdown. There is too many weird transforms when
using the "rich text" control in Blogger.
* Better display of inlined code.
* More control over the site.

I was using Blogger because I do not like the idea of server administration. I
want to spend my time on solving code problems, not on computer bookkeeping.

## Requirements

### Basic

My needs for a blog are very modest. I do not need image galleries. I want
to be able to write text, and to show code. I do not want to run a server to
dynamically generate content. This implies running a server API service at the
minimum, and then worrying about more bookkeeping. I'm fine doing the code
generation locally and uploading static content to the server.

### JavaScript tooling

I considered using something like [Octopress](http://octopress.org/), but I find
having to manage Ruby and installing gems disheartening (see above about
bookeeping). This is not a knock against Ruby in particular, I have the same
sinking feeling when using Python. I like how node programs default to "local
install" layouts, and I am much more fluent in JavaScript.

### JavaScript not required in browser

Most of my time is spent is thinking about web apps that need JavaScript, and I
really like using JavaScript. However, for plain content, I just want to use
JavaScript for enhancements. If the JavaScript fails to load, then the content
should still be reachable and readable.

I also do not want to load a bunch of JavaScript for "social sharing". Making sharing
easier is the responsibility of browser chrome/UI. The browser is my "user agent".

I also do not like supporting business models that encourage
tracking people across web sites. It is a valid way to support a
business, and I may even work on applications that use it as a way to pay their
costs, but this is my personal space, and I can afford to be a jerk about it here.

I got close to not needing any JavaScript, but ended up using it to load some fonts from
[TypeKit](https://typekit.com/), and for using [Prism](http://prismjs.com/) for
making pretty code samples. However, if they fail to load,
the content can still be read.

While TypeKit does some tracking, I am fine with it since it is about paying
font designers for their work, and I want to feel pretty. Or at the very least,
I want the site to look pretty.

### Comments

The static generation approach makes it difficult to deal with comments. I am
wary of comments in general. I appreciate getting feedback, but I want it
focused and on-topic. I also do not believe my posting area is a place
where people should have the right to comment. They should create their own
spaces for that.

The comment system should not allow anonymous comments. It is important for
the type of online discourse that I mostly do -- open source code development --
that people own their comments.

I seriously considered Disqus, and I appreciate the engineering work behind it.
It solves a hard problem. I use it to comment on other people's sites. See the
"<a href="#javascriptnotrequiredinbrowser">JavaScript not required</a>" section for why I
ended up not using it. I may reconsider if there is a way to just have a
"comment here" link that would open a Disqus thread either in a new window/fresh
page load. Something where I can just encode some ID in a link to Disqus, but
not need to include any JavaScript in a post to enable comments.

If I want to receive comments on a post, I generate a GitHub Issue for
the post, and then use that URL in a "comments" link in the post.

This works out for me, at least for now, since I use GitHub and most of my posts
are related to code. I appreciate this is not a general purpose commenting
solution.

## Implementation

I wrote a [Node-based](http://nodejs.org/) tool called
[delposto](https://github.com/jrburke/delposto). It does not do much, and I
do not suggest others use it. It is a tool for me, based on my idiosyncracies. Feel free to
open tickets and do pull requests, but set your expectations very low. I am not very motivated to improve it beyond what I need.

The basic idea: write a post in [Markdown](http://daringfireball.net/projects/markdown/),
use [Showdown](https://github.com/coreyti/showdown) to convert to HTML, and
apply [Mustache](https://github.com/janl/mustache.js) templates to generate
the final static content.

The results should be deployable to [GitHub Pages](http://pages.github.com/), particularly if you use
[volo-ghdeploy](https://github.com/volojs/volo-ghdeploy). For this site,
I have an existing hosting account and I just upload the content to that
hosting account.

For the code samples, I use [Prism](http://prismjs.com/). For fonts, I use
[TypeKit](https://typekit.com/).

The "source" for this blog is on GitHub at
[jrburke/jrburke.com](https://github.com/jrburke/jrburke.com).

Any of this could change later, but that is how it works today.
