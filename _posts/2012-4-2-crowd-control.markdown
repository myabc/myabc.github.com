---
layout: post
title: "Crowd Control"
author: myabc
categories: code
tags:   [bundler, ruby]
---

## Crowd-funding meets open-source

Something makes me deeply uncomfortable about [Yehuda Katz]'s ([@wycats]) recently announced [crowd-funding effort for rails.app](http://www.kickstarter.com/projects/1397300529/railsapp), and I have been trying to put my finger on it. To a certain extent, writing anything on the subject is a moot point. As of 2nd April, the target of $25,000 has been far exceeded. There is clearly interest (and money) in our industry for such a tool.


Nevertheless, I think [Eric Hodel] [@drbrain] is conflating points in his latest blog post ["On Community Funding of Open Source"](http://blog.segment7.net/2012/03/29/on-community-funding-of-open-source).

Over the last week, there have been those that have expressed their dissatisfaction with the way Yehuda lead refactoring of Rails 3. There were doubts about whether he was the right person to do the job. And there have, indeed, been tweets that border on the obnoxious and insulting. If we end up resorting to personal attacks, then Eric is right to call people out on that.

But, `sceptic != hater`. I am sceptical, and as such have found myself playing _agent provocateur_, retweeting various opposing views.

So why am I uneasy?


It boils down to the following three reasons:

### 1. No proof of concept. No sweat.

Yehuda is trying to appeal to a community of developers. We generally prefer to talk code, and not pour over Functional Requirement documents. A prototype, or proof of concept of some sort, would have been great.

This takes place in the context of a community that often espouses _bootstrapping_. The [Travis CI] developers sweated before asking for crowd-sourced funding. Prominent Rubyists like [Amy Hoy] champion _bootstrapping_ whole-heartedly.

In the midst of this controversy, [Jeremy McAnally] ([@jm]) has already come up with [railcar], a proof of concept that implements some parts of what Yehuda is proposing.

### 2. Oversight

When an open-source project comes about organically, through the sweat and initiative of its founder, there is usually no need for oversight. People will either use it, or not. A common adage of proponents of liberal licenses like the MIT (X11) is "I don't owe you fuck".

When a company sponsors the project, there is oversight in the sense that the company's profit imperative will usually keep the project somewhat on track.

But when a community is crowd-funding directly with a freelancer, how will this process work in practice? The scepticism does not arise from a mistrust of Yehuda. Rather, it arises from the concern about how individual developers can protect themselves against potential fall out when crowd-sourced projects go wrong.

rails.app will, inevitably, result in conflicting interests stemming from differing requirements. It could be anything from _minutiae_ like the colour of the icon to the plugins bundled by default. For a foretaste of the problems that arise, you only have to look at the Rails and Bundler issue trackers. Their developers, on the other hand, can turn around and say "I don't owe you fuck". Yehuda won't be able to do this.

Whose interests will be prioritised? Which funder? The one who paid the most, or the one whose idea appears to make the most sense?

### 3. Money. Money. Money.

This is probably the least salient of the three main points, but I will mention it regardless. Here I could also easily be accused of _jealously_ - perhaps my own fault for not monetising my own work as much as I should. But Yehuda is soliciting for work where he will be paid at a substantial rate – $275 per hour. In some parts of the world, this amount would fund an entire team.

### … and a couple open questions

It is also important to consider what sort of precedent this will set for doing open-source. To what extent will this detract from funding projects that are already established? For instance, [Rails Installer] is planning a Mac version. If I were the maintainer of [Ruby Installer], I  imagine I would feel a bit resentful after ploughing hundreds of personal hours into a product (I am not the maintainer though – and knowing [Luis Lavena] ([@luislavena]), he is far more cool than me).

### A final note

I cut my Ruby teeth on [Merb], a spectacularly well-engineered Ruby MVC Framework. I have played with [Janus] and [Thor]. I use [Bundler] constantly. I am currently building products with Rails 3 and [ember.js]. I am a great fan of Yehuda's work, and enormously grateful for the contribution he has made to open-source. His work has enriched the Ruby community.

I also do not doubt the quality of his work. I would be happy to be half the programmer Yehuda is. For some, he may have made Rails 3 seem overly complex. For my needs at least, Yehuda made Rails 3 viable.


Do I doubt we need something like rails.app? At the beginning, I was not sure if there would be demand, but I will admit, my mind has changed after participating in the discussion on Twitter.

Yehuda deserves respect, and deserves to be paid a decent rate. I hope I have summed up some of the issues with the crowd-funding approach. If the community is taking on the role of investor, we must be more demanding. Yehuda is proposing a new model of funding open-source, and it is entirely right to criticise this process, in order to refine it.

_As a footnote: this whole issue reminded me that the folks at Travis were due a little contribution from me. I was in the middle of an internal launch back when their crowd-funding started – read: living under a rock – and so it passed me by. I would encourage you to support an open-source project out there! Be it with cash donations or patches._


[Yehuda Katz]:http://yehudakatz.com/
[@wycats]:https://twitter.com/#!/wycats
[Eric Hodel]:http://blog.segment7.net/
[@drbrain]:https://twitter.com/#!/drbrain
[Travis CI]:http://travis-ci.org/
[Amy Hoy]:https://twitter.com/#!/amyhoy
[Jeremy McAnally]:http://omgbloglol.com/
[@jm]:https://twitter.com/#!/jm
[railcar]:http://jeremymcanally.com/images/railcar.mov
[Rails Installer]:http://railsinstaller.org/
[Ruby Installer]:http://rubyinstaller.org/
[Luis Lavena]:http://blog.mmediasys.com/
[@luislavena]:https://twitter.com/#!/jm
[Merb]:http://www.merbivore.com/
[Bundler]:https://github.com/carlhuda/bundler
[Janus]:https://github.com/carlhuda/janus
[Thor]:http://github.com/wycats/thor
[ember.js]:http://emberjs.com/
