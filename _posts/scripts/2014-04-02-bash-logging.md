---
layout: post
category : scripts
title: "Logging routine for Bash"
tags :
 - bash
 - log
 - script
 - linux
---
{% include JB/setup %}

Logging routine. Supports message stream from STDIN or message from options.

Usage:

```
'Log meassage' | log
log -l ERROR 'Something goes wrong :('
some_command | | log -l DEBUG -f local1
```

Options:

* `-f FACILITY` - default = local1
* `-l LEVEL` - default = debug
* `-o` - print log to STDOUT
* `-p PATH` - either directory path for logs or `syslog` to log to Syslog
* `-t TIME` - manually specify date and time for log records

<div style="text-shadow:none;"><script src="https://gist.github.com/{{ site.author.github }}/9931171.js"> </script></div>
