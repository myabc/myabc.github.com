---
layout: post
title: "A really simple way to integrate your Trac with GitHub"
author: myabc
---

 
Thanks to GitHub's API, there are definite possibilities for <a href="http://github.com/myabc/github-trac/tree/master">integrating Trac with GitHub</a>. However, for right now the simplest way is to just provide a link. Thanks to customizable navigation introduced in Trac 0.11, this should now be really simple.

If you're using the web Administration screen, click through to <em>Manage Plugins</em> and from there, ensure trac.versioncontrol.web_ui.* modules are deselected, with the exception of the <strong>BrowserModule</strong> ( trac.versioncontrol.web_ui.browser). You can also enable/disable the appropriate plugins by editing your trac.ini file.

Then you'll need to remap the browser module link to GitHub. This time, this must be done by directly editing your trac.ini file. Open trac.ini in your favourite Editor and add the following section, if it does not already exist:
<pre>
[mainnav]
</pre>

and then the following line, changing the URL to match that of your GitHub project:
<pre>
browser.href = http://github.com/myabc/github-trac/tree/master
</pre>

And that should be it!
