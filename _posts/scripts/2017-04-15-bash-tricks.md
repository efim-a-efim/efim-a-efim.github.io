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

## Some more tricks with variables

```bash
V='My=var=Data=123'
```

Cutting variable value without `cut`:
```bash
echo "${V#*=}" # Remove first occurence of template (after #) from the beginning of the string
# var=Data=123
echo "${V##*=}" # Remove all occurencies of template (after #) from the beginning of the string
# 123
echo "${V%=*}" # Same as # for string's end
# My=var=Data
echo "${V%%=*}" # Same as ## from strings end
# My
```

Pattern substitutions
```bash
V='My=var=Data=123'
echo "${V/=/,}"
# My,var=Data=123
echo "${V//=/,}"
# My,var,Data,123
echo "${V/#My/Their}" # Match at the beginning
# Their=var=Data=123
echo "${V/%123/456}" # Match at the beginning
# My=var=Data=456
echo "${V/%123}" # Remove pattern
# My=var=Data=
echo "${V/%=*}" # Wildcard!
# My
```

Same with arrays:

```bash
V=( 'My=Data=123' 'Their=Data=123' )
echo "${V[@]/%123/456}"
```

Upper and lower case:

```bash
echo "${V^}" # First char upper case
echo "${V^^}" # all chars
echo "${V,}" # First lower
echo "${V,,}" # all lower

echo "${V^t}" # all previous commands may be used with pattern, it will be checked and only matching characters will be transformed
```

# Get all variables matching prefix

Yes, Bash can do it!

```bash
MY_A='foo'
MY_B='bar'
VAR2='baz'
echo "${!MY_@}"
# MY_A MY_B
( IFS=',';  echo "${!MY_*}" )
# MY_A,MY_B
```

# Regular expressions

Bash support extended regexps in expressions:

```bash
[[ "--o=v" =~ ^\-\-[a-z]+= ]]
echo $?
# 0
```

# Functions and arguments

## Fancy long arguments handling

To handle long options with option and value separated by `=`, like: `--start=100 --end=1000 --path=/data`:

```bash
local -A OPTIONS
while [[ $# -gt 0 ]] && [[ "x${1:0:2}" =~ 'x--' ]]; do
    local _opt="${1%%=*}"
    OPTIONS["${_opt#--}"]="${1#*=}"
    [[ "${OPTIONS["${_opt#--}"]}" == "${_opt}" ]] && OPTIONS["${_opt#--}"]='true'
    shift
done
```

For options separated by space:
```bash
local -A OPTIONS
while [[ $# -gt 0 ]] && [[ "x${1:0:2}" =~ 'x--' ]]; do
    if [[ -z "$2" ]] || [[ "x${2:0:2}" =~ 'x--' ]]; then
        OPTIONS["${1#--}"]="true"
        shift
    else
        OPTIONS["${1#--}"]="$2"
        shift 2
    fi
done
```

After both option handlers you'll get:
* `OPTIONS` associative array where indexes are option names and values are their values supplied
* If option had no value supplied, it's treated as boolean and set to string `true`.
* All arguments that are not handled, are in `$@`.