---
layout: post
category : scripts
title: "Bash tricks you didn't usually use"
tags :
 - bash
 - script
 - linux
---
{% include JB/setup %}

After some experience with Bash programming (and after lots of script I've read and refactored) I've learned and figured out some useful tricks that may make code much more clean and pretty. Most of things here are in Bash manual, I'm just writing practical cases.

This article may be extended and/or rewritten in future.

# Common tricks

## Setting defaults for variables

Tired writing `[[ "$variable" ]] || variable="default"`? Look here!
```bash
: "${VARIABLE:=default value}"
: "${VARIABLE:=$(command to get default)}"
```
Explaination:
* `:` - "empty" command. It passes all arguments to Bash interpreter, but does nothing.
* `"..."` - to prevent bugs/attacks. Just in case.
* `${VARIABLE:=defaultvalue}` - if variable `VARIABLE` is not set, set its value to `defaultvalue`.

## Reading data with separators correctly

Assume you need to parse a string with separators (maybe non-spaces). Use this:
```bash
IFS='_' read -a INPUTVALUES <<<"input_string_with_underscores_as_separators"
```
You'll get a Bash array named `INPUTVALUES` with all needed data parsed.

## Reading newline-separated output

If you need to read newline-separated output of some command, don't use `while` - it's slow. Look at this:
```bash
INPUTDATA=( $(command) )
```

`INPUTDATA` will contain all output strings from your command. Note spaces near brackets - they indicate that it's an array.

## Fancy array output

Imagine you have an array like:
```bash
RESULT=( "my first string" "second one" )
```

You need it to be output, say, coma-separated. You start thinking about something like `for i in ${RESULT[@]}; do `. But there's a simpler solution!
```bash
(IFS=','; echo "${RESULT[*]}")
```
Note that using `[@]` instead of `[*]` will ignore `IFS` and output a space-separated list. It's because `[@]` returns strings list, and `[*]` generates a single string with `IFS` as a separator.

## ANSI C quoting

If you need to get "Tab" character or Unicode special character in a string, or just pass newlines as `\n`, use following form:
```bash
echo $'My cat\nIs mad'
```
This will write:
```
My cat
Is mad
```

Be careful: `$'...'` and `$"..."` are different. First one is for [ANSI C quoting](http://www.gnu.org/software/bash/manual/html_node/ANSI_002dC-Quoting.html), and second is used to translate your string to current locale.

## Setting variables in `while` loop and inside subprocesses

Everyone has this issue:
```bash
KIND='awful'
echo 'My String Is Awesome' | while read w; do
    KIND="$w"
done
echo "KIND=${KIND}"
# KIND='awful'
# WTF???
```

In this case you have a pipeline (`|`). Pipelines are not just I/O redirections, they cause Bash to create a new "subprocess" for `while` loop (or any other builtin). Variable `KIND` is not marked as exported by default (and you usually don't want to export it), so Bash honestly sets it in subprocess, but doesn't pass to parent process.

How to solve:
```bash
KIND='awful'
while read w; do
    KIND="$w"
done < <(echo 'My String Is Awesome')
echo "KIND=${KIND}"
# KIND=Awesome
```

Here we use input redirection, which does not cause subprocess creation for `while`.
