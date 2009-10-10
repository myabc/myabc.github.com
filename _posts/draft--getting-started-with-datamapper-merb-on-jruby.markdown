---
layout: post
title: "Getting Started with DataMapper on JRuby"
author: myabc
---


DataMapper + Merb on JRuby Getting Started
------------------------------------------

Briefly tucked away in the Release Notes of the 0.9.12 release of DataObjects (the database connection / )

The 0.9.12 release of DataMapper (and DataObjects, attempt to rewrite existing Ruby database drivers to conform to one, standard interface.) was the release to support JRuby. The support was basic, and we'd yet.

JRuby support for DataMapper has been l.

I've had a Gist up on GitHub for the last couple weeks, but promised to
follow up with a more detailed tutorial on how to get up and running with DataMapper on JRuby.

For the purposes of this tutorial, I am using Merb as the MVC Framework, but you should also be able to use Rails, Camping, or Sinatra.


http://blog.rayapps.com/2009/07/21/initial-version-of-datamapper-oracle-adapter/
http://sick.snusnu.info/2009/06/03/migrating-to-datamapper-0100/


Install DataMapper + Merb
-------------------------

The good news is that we're planning on making the installation procedure for JRuby just as easy as for MRI/1.9. With our next release, installation should be as simple as:

    jruby -S gem install do_sqlite3 dm-core dm-more merb

That's it! Unlike with ActiveRecord, you won't have to specify a separate JDBC version

The bad news is this support isn't released yet. In the meantime, you'll have to jump through a few hoops spend a few minutes installing from git.


* Install JRuby (tested with v.1.2.0 and 1.3.1)

* Install Addressable, Extlib, DataObjects + DataMapper (edge)

asd
    jruby -S gem install addressable

    git clone datamapper/extlib && git checkout -b next --track origin/next
    cd extlib && jruby -S rake install

    git clone datamapper/do && git checkout -b next --track origin/next
    cd do && jruby -S rake install

    git clone datamapper/dm-core && git checkout -b next --track origin/next
    cd dm-core && jruby -S rake install

    git clone datamapper/dm-more && git checkout -b next --track origin/next
    cd dm-more && jruby -S rake install

* Install Merb (edge)

asdfa

    http://github.com/snusnu/merb/commits/dm-0.10.0

    git clone git://github.com/snusnu/merb.git merb-snusnu
    && cd merb-snusnu
    &&
    git checkout -b dm-0.10.0 --track origin/dm-0.10.0
    jruby -S rake install


Create a Merb Project
---------------------

Change to the project directory of your choice and use the `merb-gen` generator to create a "jump-start" Merb application. DataMapper is the default ORM for this generator type.

    cd ~/Projects
    jruby -S merb-gen app myapp

Before you run the application, make sure the dependency versions in `config/dependencies.rb` correspond with the "edge" versions you just installed.

Open config/dependencies.rb in your favourite text editor.

If the dependencies are out-of-date:

<macro:code lang="ruby">
    merb_gems_version = "1.0.12"
    dm_gems_version   = "0.9.11"
    do_gems_version   = "0.9.11"
</macro:code>

Then bump them appropriately:

<macro:code lang="ruby">
    merb_gems_version = "1.0.12"
    dm_gems_version   = "0.10.0"
    do_gems_version   = "0.10.0"
</macro:code>

Also change the following:

<macro:code lang="ruby">
    dependency "merb_datamapper", merb_gems_version
</macro:code>

to:

<macro:code lang="ruby">
    dependency "merb_datamapper", "1.1"
</macro:code>

Run Merb
--------

Before adding in any customisatons of your own, check you can run the application.

Merb is built on Rack, and Rack needs a webserver. Webrick is a part of the Ruby standard library, so we could use that:

    jruby -S merb -a webrick

(the `-a` flag specifies an adapter)

I encountered problems shutting down Webrick though, and had to kill it. Additionally, you'll probably want to use the server you'll end up deploying to, so let's try with Mongrel instead:

    jruby -S gem install mongrel
    jruby -S merb -V

(this time with `-V` to give us some verbose output)

I am personally a big fan of GlassFish for Java/JVM deployments. You can use something like Warbler to create a WAR file for deployment, but if you're coming from the Ruby world, the following is much simpler:

    jruby -S gem install glassfish
    glassfish

Add a Model
-----------

<macro:code lang="ruby">

</macro:code>

    jruby -S rake db:migrate

    jruby -S merb
    glassfish

Add a UI
--------

Here we cheat. We're not actually going to build a UI, but instead let the U. Similar to Django's magic UI, there's a project available for Merb.

    sudo jruby -S gem install builder
    sudo jruby -S gem install sferik-merb-admin -s http://gems.github.com

In your app, add the following dependency to config/dependencies.rb:

<macro:code lang="ruby">
dependency "sferik-merb-admin", "0.3.1", :require_as => "merb-admin"
</macro:code>

Add the following route to config/router.rb:

<macro:code lang="ruby">
slice(:MerbAdmin, :name_prefix => nil, :path_prefix => "", :default_routes => false)
</macro:code>

Then, run the following rake task:

    sudo jruby -S rake slices:merb-admin:install

Visit http://localhost:4000/admin, or if you're running with glassfish, http://localhost:3000/admin.




[glassfish]:http://jruby.org/getting-started
