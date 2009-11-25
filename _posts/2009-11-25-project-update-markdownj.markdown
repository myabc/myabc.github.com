---
layout: post
title: "Project Update: MarkdownJ"
author: myabc
categories: code
---

The [Web Dingus for MarkdownJ is back](http://dingus.markdownj.org/) and
deployed to [Google App Engine](http://code.google.com/appengine/). This is the
first "application" I've deployed to GAE (I use inverted commas, as its a pretty
trivial application.) The experience was relatively straightforward, although one
issue I did encounter was that GAE's customised version of [Jetty][jetty] does
not handle `<jsp-file>`-type servlets correctly (see issue [1365][issue-1365]).
A simple workaround was to rename the jsp file I wanted to map as a servlet to
`index.jsp`.

My next step will be improving documentation and publishing the API docs
somewhere. The MarkdownJ site is currently hosted on [GitHub pages][gh-pages].
I'm using Maven as build/project tool and have played a little with the
possibility of publishing the generated docs to the `gh-pages` branch - its
possible (see [Grancher][grancher] for a Ruby tool that does this), but I don't
find it particularly elegant.

Planned for early in the new year is a new MarkdownJ parser. Currently MarkdownJ
relies on regular expressions, which doesn't provide for easy maintenance. Most
importantly though, regular expressions don't really *understand* the Markdown
code and don't provide for an intermediate representation of a Markdown
document. An internal representation would make possible such things as multiple
output formats (not just HTML, but also PDF and ODF) and pluggable converters to
other markup languages. "Eclipse Mylyn WikiText", formerly known as
[TextileJ][textilej] (eclipse.org must be hiring marketing managers from
Redmond) provides this type of functionality for the Textile markup language.

Finally, since I'm mostly working in the Ruby language these days, I'd also like
to see better support for JRuby. A thin wrapper Ruby-like API would be pretty
easy to accomplish. Its particularly needed as the fast Markdown libraries for
Ruby, [BlueCloth][bluecloth] and [RDiscount][rdiscount], both rely on C native
extensions, something JRuby does not support.


[jetty]:http://www.mortbay.org/jetty/
[issue-1365]:http://code.google.com/p/googleappengine/issues/detail?id=1365&q=jsp&colspec=ID%20Type%20Status%20Priority%20Stars%20Owner%20Summary%20Log%20Component
[gh-pages]:http://pages.github.com/
[grancher]:http://github.com/judofyr/grancher/tree/gh-pages
[textilej]:http://www.eclipse.org/mylyn
[bluecloth]:http://www.deveiate.org/projects/BlueCloth
[rdiscount]:http://github.com/rtomayko/rdiscount
