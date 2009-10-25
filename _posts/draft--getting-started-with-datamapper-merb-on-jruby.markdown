---
layout: post
title: "Getting Started with DataMapper on JRuby"
author: myabc
categories: code
---

DataMapper builds on top of DataObjects, an attempt to rewrite existing Ruby
database drivers to conform to a standard interface. Late this Spring, tucked
away in the DataObjects' Release Notes for version 0.9.12, was mention of support
for JRuby.

Unfortunately the support that featured in 0.9.12 was incomplete at best. We
needed a little longer to get things working well, but over the summer we made
significant support in supporting a [large number of][readme] databases on the
JRuby platform.

I've had a Gist up on GitHub for the last couple weeks, but promised to follow
up with a more detailed tutorial on how to get up and running with DataMapper on
JRuby.

For the purposes of this tutorial, I am using Merb as the MVC Framework, but you
should also be able to use Rails, Camping, or Sinatra.

Preflight
---------

### Install Java and the JDK

If you are running a recent version of Mac OS X, you should already have both
the Java Runtime Environment and Java Development Kit (JDK). Apple supplies
their own set of Java tools (although the [Soylatte][soylatte] implementation
is available if you want Java 6 on Tiger and Leopard 32-bit machines).

On Windows or a Linux distribution, you will likely already have the Java
Runtime Environment, but unless you've been doing Java development, you may be
without the JDK. The JDK is available from [Sun's Java downloads page][java-downloads].

### Install JRuby

We generally test against the latest stable version of JRuby. I've also tested
against v1.2.0, but I'd recommend being on 1.3.1 or 1.4.0RC2. Installing JRuby
is as simple as downloading and decompressing the distributed `.zip`/`.tar.gz`
file (or running the new `.exe` installer, if you're on Windows).

Verbose installation instructions for JRuby are available on the [Data Objects' wiki][do-wiki].

### Configure your Path

Firstly, make sure JRuby is in your PATH. Typing `jruby --version` should give
print out the version:

    $ jruby --version
    jruby 1.4.0RC2 (ruby 1.8.7 patchlevel 174) (2009-10-21 7e77f32) (Java HotSpot(TM) Client VM 1.6.0_15) [i386-java]

Because these instructions rely on installation from source, we'll also be
compiling extensions that are written in Java. Hence make sure the JDK (and the
Java compiler, `javac`) are in your PATH:

    c:\Users\alexbcoles>javac -version
    javac 1.6.0_16

On Windows for example, this would mean adding `C:\Program Files\Java\jdk1.6.0_16\bin`
to PATH. Again, more verbose instructions can be found in the [wiki][do-wiki].

Install DataMapper + Merb
-------------------------

A major requirement in developing DataObjects'/DataMapper's JDBC support was
parity in the installation procedure. Installing on JRuby should be just as easy
as for MRI / 1.9.

The good news is that if you want the latest published gems, you should be able
to install them as follows:

    jruby -S gem install do_sqlite3 dm-core dm-more merb

That's it! Unlike with ActiveRecord, you don't have to specify a separate JDBC
version.

However, for the purpose of these instructions and because our JRuby support is
still developing at a rapid pace (for instance new drivers are being made
available all the time), we'll cover installing the *edge* version of
DataObjects / DataMapper.

### Install Pre-Requisites

**Note**: Under Windows Vista/7, you may have to run cmd.exe with Administrator
privileges (Start -> type `cmd.exe`, right click, Run as Administrator).

Install Addressable and an implementation of JSON:

    jruby -S gem install json_pure addressable

### Install Extlib

Change to the directory where you keep your Code. (So `~/Dev`, `c:\code`,
`c:\Documents and Settings\harry\Code`, etc.)

We start by installing Extlib:

    git clone git://github.com/datamapper/extlib.git
    cd extlib
    git checkout -b next --track origin/next
    jruby -S rake install
    cd ..

**Note:** If the `rake install` task fails under Windows, try again in two steps:

    jruby -S rake package
    jruby -S gem install pkg\extlib-0.9.14.gem

### Install DataObjects

Then proceed to install DataObjects:

    git clone git://github.com/datamapper/do.git
    cd do
    git checkout -b next --track origin/next

    cd data_objects
    jruby -S rake install
    cd ..

Next, decide on the driver for the database vendor you wish to use. If you're
installing from source, there will be three components to install:

 * `do_jdbc`: DataObjects' JDBC support library. This is a Java library,
   packaged as a Jar, and then wrapped up as a RubyGem.
 * the JDBC driver: the database vendor's driver using the JDBC API. Also a Java
   Jar, and in most cases, available wrapped up as RubyGems.
 * the actual DataObjects' driver: written in Java and relies on the above two
   libraries at runtime. Compiled against `do_jdbc`.

To install `do_jdbc` (the DataObjects' JDBC support library):

    cd do_jdbc
    jruby -S rake compile
    jruby -S rake install
    cd ..

To install the JDBC driver for the DataObjects driver, if you're using one of
*mysql*, *postgres*, *sqlite3* then we rely on ActiveRecord-JDBC to package the
JDBC jar files as Gems for us. Installation should be a matter:

    jruby -S gem install jdbc-[DBNAME]
    jdbc-derby
    jdbc-h2
    jdbc-hsqldb
    jdbc-mysql
    jdbc-postgres
    jdbc-sqlite3

If you're on *Oracle*, then you need to place the Oracle JDBC driver `ojdbc14.jar`
in your Java load path. Read [Raimond's blog article][oracle-adapter] for more
instructions.

If you're on *SQL Server*, the ActiveRecord-JDBC has not yet packaged the jTDS
JDBC driver as a RubyGem. However, we have this available in the DataObjects
repository (you may need hoe first `jruby -S gem install hoe`):

    cd jdbc_drivers/sqlserver
    jruby -S rake install

Then you can proceed to install the DataObjects' driver:

    cd do_[DBNAME]
    jruby -S rake compile
    jruby -S rake install
    cd ../..

### Install DataMapper

Installing DataMapper's core and more gems should then be very straight-forward:

    git clone git://github.com/datamapper/dm-core.git
    cd dm-core
    git checkout -b next --track origin/next
    jruby -S rake install
    cd ..

    git clone git://github.com/datamapper/dm-more.git
    cd cd dm-more
    git checkout -b next --track origin/next
    jruby -S rake install
    cd ..

### Install Merb (edge)

There is not yet a Merb release that is fully compatible with the DataMapper
0.10.0 release, so you'll also need to install Merb from source:

    git clone git://github.com/merb/merb.git merb
    jruby -S rake install


Create a Merb Project
---------------------

Change to the project directory of your choice and use the `merb-gen` generator
to create a "jump-start" Merb application. DataMapper is the default ORM for
this generator type.

    cd ~/Projects
    jruby -S merb-gen app myapp

Before you run the application, make sure the dependency versions in
`config/dependencies.rb` correspond with the "edge" versions you just installed.

Open config/dependencies.rb in your favourite text editor.

If the dependencies are out-of-date:

{% highlight ruby %}
    merb_gems_version = "1.0.12"
    dm_gems_version   = "0.9.11"
    do_gems_version   = "0.9.11"
{% endhighlight %}

Then bump them appropriately:

{% highlight ruby %}
    merb_gems_version = "1.1"
    dm_gems_version   = "0.10.2"
    do_gems_version   = "0.10.1"
{% endhighlight %}

Also change the following:

{% highlight ruby %}
    dependency "merb_datamapper", merb_gems_version
{% endhighlight %}

to:

{% highlight ruby %}
    dependency "merb_datamapper", "1.1"
{% endhighlight %}

Run Merb
--------

Before adding in any customisatons of your own, check you can run the
application.

Merb is built on Rack, and Rack needs a webserver. Webrick is a part of the Ruby
standard library, so we could use that:

    jruby -S merb -a webrick

(the `-a` flag specifies an adapter)

I encountered problems shutting down Webrick though, and had to kill it.
Additionally, you'll probably want to use the server you'll end up deploying to,
so let's try with Mongrel instead:

    jruby -S gem install mongrel
    jruby -S merb -V

(this time with `-V` to give us some verbose output)

I am personally a big fan of GlassFish for Java/JVM deployments. You can use
something like Warbler to create a WAR file for deployment, but if you're coming
from the Ruby world, the following is much simpler:

    jruby -S gem install glassfish
    glassfish

Add a Model
-----------

{% highlight ruby %}

{% endhighlight %}

    jruby -S rake db:migrate

    jruby -S merb
    glassfish

Add a UI
--------

Here I'm sorry to say we're going to cheat. We're not actually going to spend any time building a UI, but instead we'll rely on `merb-admin`, a project that automatically generates an admin front-end similar to the Django.

    sudo jruby -S gem install builder
    sudo jruby -S gem install sferik-merb-admin -s http://gems.github.com

In your app, add the following dependency to config/dependencies.rb:

{% highlight ruby %}
dependency "sferik-merb-admin", "0.3.1", :require_as => "merb-admin"
{% endhighlight %}

Add the following route to config/router.rb:

{% highlight ruby %}
slice(:MerbAdmin, :name_prefix => nil, :path_prefix => "", :default_routes => false)
{% endhighlight %}

Then, run the following rake task:

    sudo jruby -S rake slices:merb-admin:install

Visit http://localhost:4000/admin, or if you're running with glassfish,
http://localhost:3000/admin.



[readme]:http://github.com/datamapper/do/blob/next/README.markdown
[java-downloads]:http://java.sun.com/javase/downloads/index.jsp
[soylatte]:http://landonf.bikemonkey.org/static/soylatte/
[do-wiki]:http://wiki.github.com/datamapper/do/jruby
[glassfish]:http://jruby.org/getting-started
[oracle-adapter]:http://blog.rayapps.com/2009/07/21/initial-version-of-datamapper-oracle-adapter/
[migrate-0100]:http://sick.snusnu.info/2009/06/03/migrating-to-datamapper-0100/
