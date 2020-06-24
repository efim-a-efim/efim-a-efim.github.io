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

{% include JB/gist gist_id="9931232" %}

