---
layout: post
category : scripts
title: "Lock file management in Bash"
tags :
 - bash
 - lock
 - script
 - linux
---
{% include JB/setup %}

Sometimes you need to lock a generic resource in one place and check it in another place. Below is my version of a lock file management function - in fact it's a mutex.  

<div style="text-shadow:none;"><script src="https://gist.github.com/{{ site.author.github }}/9931244.js"> </script></div>
