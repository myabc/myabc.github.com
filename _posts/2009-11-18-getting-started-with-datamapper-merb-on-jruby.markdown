---
layout: post
title: "Getting Started with DataMapper on JRuby"
author: myabc
categories: code
---

Underneath DataMapper (at least if you haven't bitten the [NoSQL][nosql] bullet
just yet) lies DataObjects. DataObjects is an attempt to rewrite existing Ruby
database drivers to conform to one, standard interface. If you're a recent
arrival from the world of PHP or .Net, then it the DO API will not look too
disimilar to either PDO or ADO.net.

Late this Spring, tucked away in the DataObjects' Release Notes for version
0.9.12, was mention of support for JRuby. Unfortunately the support that
featured in 0.9.12 was incomplete at best. We needed a little longer to get
things working well, but over the summer we made significant support in
supporting a [large number of][readme] databases on the JRuby platform.

For a long while, I had a Gist up on Github illustrating the steps needed to get
up and running with DataMapper on JRuby. I promised to follow up with a more
detailed tutorial, but its taken some time to get it published. In part, this
has been down to the fact that going through the process of writing a tutorial
has helped me uncover a plethora of bugs and inconsistencies, and I wanted to
take the time to deal with them before hitting the metaphorical "publish"
button. In any case, I hope this will be the first of many future Ruby postings.

For the purposes of this tutorial, I am using [Merb][merb] as the MVC Framework,
but you should also be able to use Rails, Camping, or Sinatra.

Many thanks to [Dan Kubb (dkubb)](http://twitter.com/dkubb) for looking over
this post and to [George Adamson](http://twitter.com/GeorgeAdamson) for actually
having the patience to test out these steps (on Windows, of all platforms!)

Preflight
---------

### Install Java and the JDK

If you are running a recent version of Mac OS X, you should already have both
the Java Runtime Environment and Java Development Kit (JDK). Apple supplies
their own set of Java tools (although the [Soylatte][soylatte] implementation is
available if you want Java 6 on Tiger and Leopard 32-bit machines).

On Windows or a Linux distribution, you will likely already have the Java
Runtime Environment, but unless you've been doing Java development, you may be
without the JDK. The JDK is available from [Sun's Java downloads page][java-downloads].

### Install JRuby

We generally test against the latest stable version of JRuby. I've also tested
against v1.2.0, but I'd recommend being on 1.3.1 or 1.4.0. Installing JRuby
is as simple as downloading and decompressing the distributed `.zip`/`.tar.gz`
file (or running the new `.exe` installer, if you're on Windows).

Verbose installation instructions for JRuby are now available on the
[Data Objects' wiki][do-wiki].

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
variant.

However, for the purpose of these instructions and because our JRuby support is
still developing at a rapid pace (for instance, our SQL Server support is only
available in our Git repository), we'll cover installing the *edge* version of
DataObjects / DataMapper.

### Install Pre-Requisites

**Note**: Under Windows Vista/7, if you've used the JRuby installer and have
your system gems directory in `C:\Program Files\` then you may have to run
`cmd.exe` with Administrator privileges (**Start** -> type `cmd.exe`, right
click, **Run as Administrator**).

**Note**: On *NIX platforms, you may need to prefix commands with sudo,
depending on where you installed `jruby` (unless, of course, you're using
[rvm][rvm]).

Install Addressable and an implementation of JSON:

    jruby -S gem install json_pure addressable

### Install Extlib

Change to the directory where you keep your Code. (So `~/Dev`, `c:\code`,
`c:\Documents and Settings\harry\Code`, etc.)

We start by installing [Extlib][extlib]:

    git clone git://github.com/datamapper/extlib.git
    cd extlib
    jruby -S rake install
    cd ..

**Note:** If the `rake install` task fails under Windows, try again in two steps:

    jruby -S rake package
    jruby -S gem install pkg\extlib-0.9.14.gem

Extlib is a collection of extensions to core Ruby classes and is currently used
by both the DataMapper and Merb projects. However, it'll go away in future
versions as we start to take advantage of the new, modular ActiveSupport in
Rails 3.0.

### Install DataObjects

Then proceed to install DataObjects:

    git clone git://github.com/datamapper/do.git
    cd do

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
JDBC jar files as Gems for us. Installation should be a matter of:

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
    cd ../..


Unless you've got your own project or have your mind set on a specific vendor's
relational database, then follow this tutorial all the way through and install
SQLite3 support (it means you won't have to do any configuration later!):

    jruby -S gem install jdbc-sqlite3


Then you can proceed to install the DataObjects' driver:

    cd do_[DBNAME]
    jruby -S rake compile
    jruby -S rake install
    cd ../..

### Install DataMapper

Installing DataMapper's core and more gems should then be very straight-forward:

    git clone git://github.com/datamapper/dm-core.git
    cd dm-core
    jruby -S rake install
    cd ..

    git clone git://github.com/datamapper/dm-more.git
    cd dm-more
    jruby -S rake install
    cd ..

### Install Merb

The Merb maintainers just came out with a release (1.0.15) that should be
compatible with DataMapper 0.10.x series. You should be able to install the gems
as follows:

    sudo jruby -S gem install merb-core merb-more

Create a Merb Project
---------------------

Change to the project directory of your choice and use the `merb-gen` generator
to create a "jump-start" Merb application. DataMapper is the default ORM for
this generator type.

    cd ~/Projects
    jruby -S merb-gen app myapp
    cd myapp

Before you run the application, make sure the dependency versions in
`config/dependencies.rb` correspond with the "edge" versions you just installed.

Open config/dependencies.rb in your favourite text editor.

If the dependencies are out-of-date, then bump them appropriately:

{% highlight ruby %}
merb_gems_version = "1.0.15"
dm_gems_version   = "~> 0.10"
do_gems_version   = "~> 0.10"
{% endhighlight %}

**Note**: The next major version of Merb, 1.1, will completely replace the
dependency handling mechanism described here with [Yehuda Katz's (wycats)][wycats]
[Bundler][bundler]. Its technically possible to use the Bundler with Merb 1.0.x
series, but for concision, I won't elaborate on doing that here.

Run Merb
--------

Before adding in any customisatons of your own, check you're able to run the
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

I am personally a big fan of [GlassFish][glassfish] for Java/JVM deployments.
You can use something like Warbler to create a WAR file for deployment, but if
you're coming from the Ruby world, the following is much simpler:

    jruby -S gem install glassfish
    glassfish

Add a Model
-----------

Here I start to get lazy. Bereft of ideas, and starting to think about needing
to cook, I decided to create a simple Recipe tracker. While introducing some of
the cooler (and defining) DataMapper features will have to wait until a future
article, this simple child-parent model should be immediately familiar to anyone
who's done basic web development.

Create a file `ingredient.rb` in `app/models` with the following contents:

{% highlight ruby %}
class Ingredient
  include DataMapper::Resource

  property :id,           Serial
  property :name,         String, :length => 50, :nullable => false

  belongs_to :recipe, :nullable => true

end
{% endhighlight %}

and a file `recipe.rb` in `app/models` with the following contents:

{% highlight ruby %}
class Recipe
  include DataMapper::Resource

  property :id,           Serial
  property :title,        String, :length => 100, :nullable => false
  property :instructions, Text
  property :cooking_mins, Integer, :min => 5, :max => 120

  has n, :ingredients

end
{% endhighlight %}


If you chose SQLite3 as the relational database you wanted to use earlier, then
you don't need to change any database configuration. If you chose another
database, then open `config/database.yml` and change appropriately. For
PostgreSQL, or Microsoft SQL Server configurations might look like the
following:

{% highlight yaml %}
# PostgreSQL
development: &defaults
  adapter: postgres
  database: sample_development
  username: postgres
  password: 
  host: localhost

# Microsoft SQL Server
development: &defaults
  adapter: sqlserver
  database: sample_development;instance=SQLEXPRESS;
  username: mswin
  password: doze
  host: sqlserver.localnet
{% endhighlight %}

Automigrate the database:

    jruby -S rake db:automigrate

And run the application with *ONE* of the following commands:

    jruby -S merb
    glassfish

Add a UI
--------

Here I'm sorry to say we're going to cheat (yet again!). We're not actually
going to spend any time building a UI, but instead we'll rely on `merb-admin`, a
project that automatically generates an admin front-end similar to [Django][django],
the Python web application framework.

    jruby -S gem install builder
    jruby -S gem install merb-admin -s http://gemcutter.org

In your app, add the following dependency to config/dependencies.rb:

{% highlight ruby %}
dependency "merb-admin", "0.6.6"
{% endhighlight %}

Add the following route to config/router.rb (just after `:merb_auth_slice_password`):

{% highlight ruby %}
slice(:merb_auth_slice_password, :name_prefix => nil, :path_prefix => "")
slice(:merb_admin, :path_prefix => "admin")
{% endhighlight %}

Then, run the following rake task:

    jruby -S rake slices:merb-admin:install

Fire up your application, with *ONE* of the following commands:

    jruby -S merb
    glassfish

Visit http://localhost:4000/admin, or if you're running with glassfish,
http://localhost:3000/admin.

You should have now have an application with a (very) simple interface for
adding recipes and ingredients.

<div class="thumbnail"><a href="http://skitch.com/alexcoles/nebyi/new-recipe-merbadmin"><img src="http://img.skitch.com/20091118-nu1ewthus4nu1ncrbkcyau6jsb.preview.jpg" alt="New recipe | MerbAdmin" /></a><br /><span style="font-family: Lucida Grande, Trebuchet, sans-serif, Helvetica, Arial; font-size: 10px; color: #808080">Uploaded with <a href="http://plasq.com/">plasq</a>'s <a href="http://skitch.com">Skitch</a>!</span></div>

You can find the [source code for this application on GitHub](http://github.com/myabc/recipes-sample-app).

That's it for now! Please leave a comment and let me know what (if anything) was
useful, so that I can focus my efforts for future articles.


[readme]:http://github.com/datamapper/do/blob/next/README.markdown
[nosql]:http://en.wikipedia.org/wiki/NoSQL
[merb]:http://merbivore.com/
[java-downloads]:http://java.sun.com/javase/downloads/index.jsp
[soylatte]:http://landonf.bikemonkey.org/static/soylatte/
[do-wiki]:http://wiki.github.com/datamapper/do/jruby
[rvm]:http://rvm.beginrescueend.com/
[extlib]:http://github.com/datamapper/extlib/
[warbler]:http://warbler.kenai.com/
[glassfish]:http://jruby.org/getting-started
[oracle-adapter]:http://blog.rayapps.com/2009/07/21/initial-version-of-datamapper-oracle-adapter/
[migrate-0100]:http://sick.snusnu.info/2009/06/03/migrating-to-datamapper-0100/
[wycats]:http://yehudakatz.com/
[bundler]:http://github.com/wycats/bundler
[django]:http://www.djangoproject.com/
