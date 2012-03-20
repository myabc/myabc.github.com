---
layout: post
title: "Bundler Pain Points"
author: myabc
categories: code
tags:   [bundler, ruby]
---

In so many ways, [Bundler](http://gembundler.com/) is a godsend. Yet, like all gems (forgive the wordplay), it is not without a few rough edges.

While at the (absolutely fantastic) [Wroclaw.rb](http://wrocloverb.com/) last weekend, I finally took a moment to round up some of these issues into a [lightning talk](http://wrocloverb.alexbcoles.com/). Unfortunately, not having paced myself correctly I got cut off just before the final couple of slides. So I resolved to reformulate the content into a blog post. A further **impetus** was a recent [gist](https://gist.github.com/2063855) by [Sven Fuchs](http://twitter.com/svenfuchs) (der [Travis CI](http://travis-ci.org/)- und Ruby I18n-Meister).


The aim of this article is to highlight a handful of — what I consider to be — the rough edges, and to proffer potential solutions/workarounds.

Perhaps surprisingly, one thing I am not going to talk about is the issue of **performance** (in other words, that **infamous** `Fetching source index…` wait). With the recent launch of Bundler 1.1 you should notice [a big speedup](http://patshaughnessy.net/2011/10/14/why-bundler-1-1-will-be-much-faster). If you have not upgraded, please do. Bundler uses [semantic versioning](http://semver.org/) and this release should be fully API-compatible with Bundler 1.0.x.


Here is my run down of **the top 3 Bundler pain points**:

1. **Lack of optional groups**
2. **Lack of overridable version dependencies**
3. **Not being able to declaring a dependency more than once**


## 1. Lack of optional groups
### Use case

You are developing an **application** (as opposed to a **library**) and you want it to be usable and deployable 'out of the box.' You do not want your users to have to manually edit (or uncomment lines in) their `Gemfile`. You will find that this problem arises nearly always with database drivers.

### Workaround

The current best workaround is something along these lines:

    bundle install --without=postgres,sqlite3

In other words, rather than specifying groups to _include_, you have to provide a list of what you want to _exclude_.

{% highlight ruby %}
gem 'pg',       groups: 'postgres'
gem 'sqlite3',  groups: 'sqlite3'
gem 'mysql',    groups: 'mysql'
{% endhighlight %}

### A better fix

There is a [proposal](https://github.com/carlhuda/bundler/issues/1636) from [@FooBarWidget] and [@mislav], which I find to be very pragmatic. In summary, this is what it would look like:

Installing optional groups,

    bundle install --with=postgresql

and in the `Gemfile`,

{% highlight ruby %}
group :postgres, optional: true do
  gem 'pg'
end
{% endhighlight %}


## 2. Lack of overridable version dependencies
### Use case

When a new version of Rails appears, it is frustrating to upgrade, only to discover that you cannot proceed until the plugins and libraries you depend upon are updated. In some cases, this is down to definite API incompatibilities, but in other cases, it might have been that the developer of the particular library was just too cautious in specifying the versions the library supports.

### Workaround

The current, recommended workflow is to fork the particular library or plugin and specify it as a git dependency in your `Gemfile`.

### A better fix

There is [an issue open for this](https://github.com/carlhuda/bundler/issues/1549).

Thus, overriding dependencies when a particular gem `gemspec` is out of date:

{% highlight ruby %}
gem 'sass-rails' do
  override 'railties', '4.0.0'
end
{% endhighlight %}

This is not something the Bundler developers are considering adding in, they assert the fork and git dependency workflow works well. But I respectfully disagree.
I would even encourage you to +1 this issue!

## 3. Not being able to declaring a dependency more than once

### Use case

Bundler will not currently let you declare a dependency more than once. This means you cannot have different versions or sources for a dependency, even if you scope your dependencies using groups.

Not being able to declare a dependency more than once makes the following things painful:

- testing against multiple versions of a resource (for example, in CI) _(problem A)_
- development of componentised systems:
  (perhaps working against a git repository of a dependency, or an Engine in git)
  _(problem B)_

In fact, _problem B_ provided my initial motivation for investigating the whole question. At [Payango](http://payango.com/en), we are currently building a new platform for prepaid VISA cards; our previous platform was built as a monolithic Rails application, but the new platform is built on the foundations of a [Service-oriented Architecture](http://en.wikipedia.org/wiki/Service-oriented_architecture). For each given product on our platform, we have multiple applications running. To share code, we are employing Rails engines and internal libraries extensively.

Of course, in a perfect world — and because we aim to keep concerns separate — our layers would be perfectly tested and respect their contracts. In this perfect world, we would never need to break down, debug or change views in our upstream library or Engine. Unfortunately — despite the best efforts and good design — we do sometimes need to work on a library and the host application simultaneously.

### Workaround …

The concept is **easy**: the solution for both problems is to employ multiple Gemfiles.


#### … for problem A

Create more than one `Gemfile`, where you want to use different versions or a specify a tighter version than that defined in the `.gemspec`.

I have seen this solution used by [Ripple](http://seancribbs.com/ripple/), for example:

{% highlight ruby %}
    # Taken from:
    # https://github.com/seancribbs/ripple/blob/master/Gemfile.rails31

    gem 'activemodel', '~> 3.1.0'

    instance_eval File.read(File.expand_path('../Gemfile', __FILE__))
{% endhighlight %}

where the `.gemspec` may contain,

{% highlight ruby %}
gem.add_dependency "activemodel", [">= 3.0.0", "< 3.3.0"]
{% endhighlight %}


such an approach is useful for automated testing, particularly [Continuous Integration](http://en.wikipedia.org/wiki/Continuous_integration). Travis CI allows you to specify multiple Gemfiles in your `.travis.yml`,

{% highlight yaml %}
gemfile:
  - Gemfile.rails30
  - Gemfile.rails31
  - Gemfile.rails32
{% endhighlight %}

#### … for problem B

The solution lies in the environment variable `BUNDLE_GEMFILE`. First create a copy of your `Gemfile` as `Gemfile.local` and specify your path dependencies in your `Gemfile.local`, keeping your git or gem dependencies in your original `Gemfile`.

When it comes to working with the host application or library, just make sure `BUNDLE_GEMFILE` is set:

    BUNDLE_GEMFILE=Gemfile.local bundle install
    BUNDLE_GEMFILE=Gemfile.local rake spec
    BUNDLE_GEMFILE=Gemfile.local rails server

of course you can wrap this into a shell function,

    local_gemfile_on
    local_gemfile_off

(See the [gist](https://gist.github.com/2126969) for example functions you can add to your `.bash_profile`. You might also consider adding your current `BUNDLE_GEMFILE` to `PS1`)

Keeping more than one `Gemfile` is, of course, a big pain. So, use Rake to keep the copy in sync:

{% highlight ruby %}
    task :local_gemfile do
      gemfile = File.read('Gemfile')
      gemfile.gsub!(/git: ["']git@github\.com:payango\/([\w-]+)\.git["']/, 'path: \'../\1\'')
      File.open('Gemfile.local', 'w') { |file| file.write(gemfile) }
    end
{% endhighlight %}

See the [gist](https://gist.github.com/2016633) for the full tasks.

Two points to bear in mind when using this technique with Rails:

- This will not handle code reloading: just because you do not have to `bundle update` each time you want to pull in the latest version of your library/Engine, it does not mean you will see all changes straight away. I have not yet worked out a solution for reloading Engines code on change.

- If you are building on top of an older point release of Rails 3.x, you may need to update your `config/boot.rb` file. Because, older generated boot files overwrote the `BUNDLE_GEMFILE` environment variable, make sure your file looks [like this](https://github.com/rails/rails/blob/master/railties/lib/rails/generators/rails/app/templates/config/boot.rb).

### A better fix

A better fix is on its way. Just as I was finishing this article, the following [pull request](https://github.com/carlhuda/bundler/pull/1779) from [@josevalim] was merged. I have yet to dig through the code, and it will take some time until the feature is released, but what is there already looks promising.

## As a conclusion

Let us get this clear (and to end on a positive note):

The Ruby world before Bundler was a much more painful place. Manually installing Gems — with the ensuing Gem activation conflicts — was not fun. The Rails 2 way of dependency management, `config.gem` was buggy, at best.

Bundler has collectively spared us thousands of hours, and I hope to see it improved further.

_What are your pain points?_



[@FooBarWidget]:https://github.com/FooBarWidget
[@mislav]:https://github.com/mislav
[@josevalim]:(http://github.com/josevalim)
