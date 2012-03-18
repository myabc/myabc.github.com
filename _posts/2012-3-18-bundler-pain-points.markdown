---
layout: post
title: "Bundler Pain Points"
author: myabc
categories: code
tags:   [bundler, ruby]
---

In so many ways, Bundler is a godsend. Yet, like all gems (forgive the wordplay), it's not without a few rough edges.

Let's get this clear:

The Ruby world before Bundler was a much more painful place. Manually installing Gems (with the ensuing Gem activation conflicts) was not fun. The Rails 2 way of dependency managment, `config.gem` was buggy, at best.

But unfortunately, that's it for the positive.


While at the (absolutely fantastic) [Wroclaw.rb](http://wrocloverb.com/) last weekend, I finally took a moment to round up some of these issues into a [lightning talk](http://wrocloverb.alexbcoles.com/). Unfortunately, not having paced myself correctly I got cut off just before the final couple slides. So I resolved to reformulate the content into a blog post. A further impetus was a recent [gist](https://gist.github.com/2063855) by [Sven Fuchs](http://twitter.com/svenfuchs) (der [Travis CI](http://travis-ci.org/)- und Ruby I18n-Meister).


The aim of this article is to highlight a a handful of (what I consider) to be the rough edges, and profer potential solutions/workarounds.

Perhaps surprisingly, somethign I'm not going to feature is not in this article is any mention of performance (i.e. in other words that infamous `Fetching source index…` wait). That's largely [resolved](http://patshaughnessy.net/2011/10/14/why-bundler-1-1-will-be-much-faster) with the recent launch of Bundler 1.1. If you haven't upgraded, do. Bundler uses semantic versioning and this release should be fully API-compatible with Bundler 1.0.x.


Here's my run down of my **top 3 Bundler pain points**:

1. **Lack of Optional Groups**
2. **Lack of Overridable Version Dependencies**
3. **Not being able to declaring a Dependency more than once**


## I. Lack of Optional Groups
### Use Case

- You're developing an **application** (as opposed to a **library**) and you want it to be usable and deployable "out of the box." You don't want your users to have to manually edit (or uncomment lines in) their `Gemfile`.
- This problem nearly always arises with database drivers.

### In more detail…

    bundle install --without=postgres,sqlite3

You could use groups, but you have to provide a list of what you want to
exclude rather than include.

{% highlight ruby %}
gem 'pg',       groups: 'postgres'
gem 'sqlite3',  groups: 'sqlite3'
gem 'mysql',    groups: 'mysql'
{% endhighlight %}

### Discussion

proposal from **@FooBarWidget** and **@mislav**:
<https://github.com/carlhuda/bundler/issues/1636>


### Fix

    bundle install --with=postgresql

In the `Gemfile`, groups would look like this:

{% highlight ruby %}
group :postgres, optional: true do
  gem 'pg'
end
{% endhighlight %}

Allowing applications to be distributed without requiring manual editing
of the Gemfile. Mislav's suggestion for optional groups strikes me as sensible.
There's a GitHub issue for this, so I'll let the issue do the talking.

## II. Lack of Optional Groups
### Use Case

It can be frustrating when the latest version of a Rails appears, only to discover that developers haven't updated their plugins yet. The current, recommended workflow is to fork the particular library or plugin in question, specify it as a gem dependency in your `Gemfile`.

Overriding dependencies when a particular gem `gemspec` is out of date.

{% highlight ruby %}
gem 'sass-rails' do
  override 'railties', '4.0.0'
end
{% endhighlight %}



### Discussion and Fix

<https://github.com/carlhuda/bundler/issues/1549>

There is an issue open for this, but this is not something the Bundler
developers currently want to add in. They prefer the fork and git hub dependency
workflow. I'd encourage you to +1 this issue!



## III. Declaring a Dependency more than once
### Use Case

- test against multiple versions of a resource (for example, in CI) _(problem A)_
- facilitates developing componentised systems:
  (perhaps working against a git repository of a dependency, or an Engine in git)
  _(problem B)_

Travis CI allows you to specify multiple Gemfiles in your `.travis.yml`.
The Engines issue is my big motivation for trying to deal with this issue.


### Fix

Concept is EASY: The solutions for both these issues are to employ multiple Gemfiles.


### Fix for problem A

Multiple `Gemfile`s, where you want to use different versions or tighten versions
that might have been defined in the `.gemspec`.

I've seen this solution used by Ripple, for example.

{% highlight ruby %}
    # Taken from:
    # https://github.com/seancribbs/ripple/blob/master/Gemfile.rails31

    gem 'activemodel', '~> 3.1.0'

    instance_eval File.read(File.expand_path('../Gemfile', __FILE__))
{% endhighlight %}

### Fix for problem B

Set `BUNDLE_GEMFILE` and use a `Gemfile.local`

Multiple Gemfiles, where you want to use different versions or tighten versions
that might have been defined in the `.gemspec`.


Handling multiple Gemfiles is of course, a big pain. So use Rake:

{% highlight ruby %}
    task :local_gemfile do
      gemfile = File.read('Gemfile')
      gemfile.gsub!(/git: ["']git@github\.com:payango\/([\w-]+)\.git["']/, 'path: \'../\1\'')
      File.open('Gemfile.local', 'w') { |file| file.write(gemfile) }
{% endhighlight %}

<https://gist.github.com/2016633>


When it comes to working with the repository, just make sure your `BUNDLE_GEMFILE` is set.

    BUNDLE_GEMFILE=Gemfile.local bundle install

My Bash skills leave much to be desired. Of course you can wrap this in a script:

    local_gemfile_on
    local_gemfile_off

Still to be fixed: autoloading of Engines, libraries. Upgrade `BUNDLE_GEMFILE`.
If you're using Rails 3.1, you may also have to update your `config/boot.rb`
Autoloading - any solutions for this?


I respectfully disagree with [@indirect]'s assertion that Bundler is not a time-saving tool. It is. Just as with a vaccuum cleaner. With the notable exception of its resolution engine, there's little you couldn't accomplish without Bundler (you can iterate through a list of required gems from a Manifest.txt and install them
manually).

_What are your pain points?_

[FooBarWidget]:https://github.com/FooBarWidget
[mislav]:https://github.com/mislav
[indirect]:https://github.com/indirect

