---
layout: post
title: "Joining the bandwagon"
author: myabc
categories: code
---


<pre>history | awk '{a[$2]++}END{for(i in a){print a[i] " " i}}' | sort -rn | head</pre>
<pre>
140 git
103 ls
73 cd
55 mate
26 sake
26 mv
23 rake
8 autotest
5 gem
4 sudo
</pre>
