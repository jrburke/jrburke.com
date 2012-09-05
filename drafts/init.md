title: New Blog
tags: [meta]
comments:
~
I have started a new place to post, [jrburke.com](http://jrburke.com/). I have
been using [http://tagneto.blogspot.com](http://tagneto.blogspot.com), but
wanted something new.

## Motivation

I was motivated by the following:

* I want better display control over inlined code
* I want to code posts in Markdown. There is too many weird transforms when
using the "rich text" control in Blogger.
* I want a better online identity.

I was using Blogger because I do not like the idea of server administration. I
want to spend my time on solving code problems, not on computer bookkeeping
issues.

## Requirements

I considered using something like Octopress, but I find having to manage ruby
and installed gems disheartening (see above about bookeeping). This is not a
knock against ruby in particular, I have the same sinking feeling when using
python. I am also much more fluent in JavaScript, and I like how node programs
default to "local install" layouts.

My needs for a blog are also very modest. I do not show a bunch of images, need
image galleries. I want to be able to write text, and to show code. I also do
not want to run a server to dynamically generate content. This implies running a
server API service at the minimum, and then worrying about more bookkeeping.
I'm fine doing the code generation locally and uploading static content to the
server.

### JavaScript not required

I want my posts to work even if JS fails to load. Most of my time is spent is thinking about web apps that need JS, and I really like using JS. However, for plain content, I like just distributing the content. I just want to use JS for enhancements. If they fail to load, then the content should still be visible.

I also do not want to load a bunch of BS JS for "social sharing". Making sharing easier is the responsibility of browser chrome/UI. The browser is my "user agent".

I also do not like the idea of supporting business models that encourage tracking people across web sites. I appreciate it is a valid way to support a business (many people are fine with this tradeoff), and I may even work on applications that use it as a way to pay their costs, but this is my personal space, and I can afford to be a jerk about it here.

I got close to not needing any JS, but ended up using it to load some fonts from TypeKit, and for using Prism for making pretty code samples. However, these are additive, and loaded async. If they fail, the content can still be read.

While TypeKit does some tracking, I am fine with it, since it is about making sure to pay font designers for their work. I can appreciate they may be jerks and use that data in some other way, and I may need to revisit that choice later. For now, I load the TypeKit JS because I want to feel pretty. Well, at least I want this site to feel pretty. I am sure this is obvious, but I am not a designer, and one of the tricks I use to enhance the site is fonts.

### Comments

The static generation approach makes it difficult to deal with comments. I am wary of comments in general. I appreciate getting feedback, but I want it focused and on-topic. I also do
not believe my posting area needs to be where people should have the right to comment. They should create their own spaces for that.

The comment system should not allow anonymous comments. It is important for
the type of online discourse that I mostly do -- open source code development --
that people own their comments.

I seriously considered Disqus, and I appreciate the engineering work behind it. It solves
a hard problem. I use it to comment on other people's sites. See the <a href="#javascriptnotrequired">JavaScript not required</a> section for why I ended up not using it. I may reconsider if there is a way to just have a "comment here" link that would open a Disqus thread either in a new window/fresh page load. Something where I can just encode some ID in a link to Disqus, but not need to include any JS in a post to enable comments.

What I do for now for comments: if I want to receive comments on a post, I generate a GitHub Issue for the post where I want to allow comments, and then use that link in a "comments" link for the post.

This works out for me, since I use GitHub a lot and most of my posts are related to code. I appreciate this is not a general purpose commenting solution.

## Implementation

Any of this could change later, but as of today:

I ended up writing a tool to generate the posts from Markdown. It is called
[delposto](). It does not do much, and I
would not suggest others use it, at least not at this point. It is a tool for
me, for my particular idiosyncracies. Feel free to open tickets/do pull
requests, but set your expectations very low.

The basic idea: write a post in Markdown, use Showdown to convert to HTML, and
apply Mustache-based templates to generate the final static content.

The results should be deployable as a GitHub Pages site, particularly if you use
[volo-ghdeploy](). However, for this site, I have an existing hosting account,
and I just upload the content to that hosting account.

For the code samples, I use Prism.

The "source" for this blog is on GitHub at [jrburke/jrburke.com]().


