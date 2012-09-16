title: Understanding macro use cases for JavaScript
tags: [javascript, mozilla]
comments: https://github.com/jrburke/jrburke.com/issues/2
~
I just watched [this Mozilla video on some experimental work Tim Disney is doing
for JS macros](https://air.mozilla.org/sweetjs/). While
watching it, I also read came across [Dave Herman's homoiconicity post](http://calculist.org/blog/2012/04/17/homoiconicity-isnt-the-point/).

Interesting stuff, and helped me to better understand how macros might fit
into JavaScript, at least at a technical level.

My crude layman's explanation of macros: transpilers but only JS token
types are used, versus having to make your own whole grammar, and constrained to <strike>curlies</strike>
() {} [] as delimiters (thanks to Tim for the correction). These limitations are there in order to insert in a simplified
parser (called the reader) between the usual JS lexer and parser stages.

So macros seem to be useful when:

* you do not want a full transpiler, just some small syntax forms
* and using a function() to do the code reuse is not appropriate

I am having trouble visualizing those use cases though. My initial reaction is
that macros are something neat for a language enthusiast, but do not actually
make coding, the sharing and reading of code, that much better, particularly
since it involves a level of indirection that is subtle, the new grammar needs
to be tracked down. I find this is the hardest part of understanding codebases,
tracking down what different units of code are doing, where the definitions live.

My first impulse is to say, "just make a function" that does
what you want, using existing JS grammar.

If a function is not enough, what you are probably wanting is a full custom
language, so then a full grammar/transpiler makes sense.
The use of transpilers are easier to call out in their use -- normally a different
file extension, and by not being limited to JS token types/delimiters, you are
more likely to create a terser grammar.

I can see the need to make transpiler construction easier. And if someone wanted
to create a "like JS but with some tweaks", maybe create something higher
level than something like [jison](http://zaach.github.com/jison/) to do that.
I would have liked that when playing around with module syntax for JS, for
example.

So it is hard for me to see the case for macros, at least as described in
that video. They seem to be a middle ground between functions and full
transpilers whose benefits seem to be outweighed by making it more difficult
to comprehend code written in "JavaScript".

But I am trying to learn more about it. If anyone has pointers to the cases
where macros shine and their use outweighs my perceived readability/code tracing
costs, I would love to hear it. Feel free to leave a comment.

However, do not comment with at "yeah macros are dumb". This is an exploration
in enlightenment. I'm already skeptical of macros, so I do not need that
reinforcement. I'm looking for more concrete use cases, pros and cons on specific
points, not fear of change.

Similarly, comments of the form "because, Language X has them and
they are awesome" will likely not be informative. It is likely language X and
JS do not share other properties and cultures that are in JS, like the
propensity to create transpilers for JS. I want uses cases that for JS would not
be solved via functions or a full transpiler.
