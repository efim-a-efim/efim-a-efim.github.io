---
layout: post
category: msbuild
title: "Building JS assets with MSBuild"
tags :
 - .net
 - msbuild
 - devops
 - build
 - config
 - transform
 - mimosa
 - js
---
{% include JB/setup %}

After [making MSBuild transform my files](/msbuild/2016/04/25/MSBuild-Transforms-for-All-Configs) I've faced with another problem - JavaScript.

Everyone knows a JS developers game: take a random word; google `RandomWord.js`; if you've found a JS library - drink. The best way to die of alcohol, really. As for JS "build" tools, there are several "flagmans" like Gulp, NPM and so on. But not everyone uses them. Remember the game...

## Problem

So we've got a JS "builder" such as [Mimosa](http://mimosa.io/) that's used in your project. And we need to integrate it with MSBuild, including transforms and all other features. The problem is that our build tool produces files in a separate folder that MSBuild doesn't know. So they won't be automatically copied to output and transforms won't be applied to them.

## Solution

Here's an example that will run Mimosa on `Content` folder of the .NET project. After build completes, it will add all produced files to MSBuild `Content` item group, so MSBuild will know what to copy to output.

{% include JB/gist gist_id="817a4f67b54abea1df21410611286ce5" gist_file="JS-build.csproj" %}

You can use any build/convert tool here, it will work fine :)