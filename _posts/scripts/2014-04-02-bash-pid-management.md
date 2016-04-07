---
layout: post
category : scripts
title: "PID file management in Bash"
tags :
 - bash
 - pid
 - script
 - linux
---
{% include JB/setup %}

PID file management.

Usage:

* Check PID file for current process
  ```sh
  pid check
  ```
* Create PID file
  ```sh
  pid set
  ```
* Remove PID file
  ```sh
  pid clear
  ```

Options:

* `-p PID` - set PID to check/write
* `-f FILE` - PID file name

<div style="text-shadow:none;"><script src="https://gist.github.com/{{ site.author.github }}/9931232.js"> </script></div>
