---
layout: post
category: msbuild
title: "Applying Web.config transforms to all config files"
tags :
 - .net
 - msbuild
 - devops
 - build
 - config
 - transform
---
{% include JB/setup %}

## Situation

You have a .NET Web app or just a .NET application that has config files. You need this config files to be different for developer's machine, CI server and Production. Also you have some text/HTML/JS/CSS files that must be adjusted for development, production and CI.

## Solution

You'll be surprized, but even native MSBuild with Web extensions can do it properly. Brief description follows:

### 1. Create solution configurations

Open your `.sln` in Visual Studio and open Configurations manager (click on your current build configuration dropdowh at the top of the window and select Configuration manager).

Create a new configuration(s), one for every needed config files modification. E.g. if you need separate config transformation logic for CI server and Production, create `CI` and `Production` configurations. You can create empty configurations or clone them from existing ones. I usually clone from `Release` config. Checkbox `Create project configurations` must be checked.

### 2. Create config transformations

If you're not familiar with [Web config transforms](https://msdn.microsoft.com/en-us/library/dd465326(v=vs.110).aspx), go and read the docs! Lots of them over the Internet...

For every config file you have, you should create file(s) named like `<FileName>.<Configuration>.<Extension>`, example: for `App.config` create `App.CI.config` and `App.Production.config`. Put corresponding transformations into them.

### 3. Create replacement files

If you have some plaintext files or other non-XML files to be modified, just create replacement files for all of them like `<FileName>.<Configuration>.<Extension>`. Put all needed content to replacement files. And yes, you'll need to duplicate some parts.

### 4. Add files to projects

Now we need to add our transforms to project files and tell MSBuild that they are dependent upon our config/original files.

Open your project(s) (`.csproj`, `.vbproj`, etc.) in text editor. Find your config file record, it looks like this:

{% include JB/gist gist_id="817a4f67b54abea1df21410611286ce5" gist_file="sample-config-entry.csproj" %}

You should replace it with this:

{% include JB/gist gist_id="817a4f67b54abea1df21410611286ce5" gist_file="sample-config-entry-modified.csproj" %}

You see, we've set some additional properties. To be short: we made config file to copy to output folder after build and denied copying for transforms.

In `DependentUpon` tag, always use file paths relative to your project directory.

Now you can open your solution in Visual Studio and see how pretty are your config files with folded-in transforms :)

### 5. Run transforms on config files

Here's a sample MSBuild XML that does the following:

1. Run config transforms on all XML configs. It detects App.config and uses special logic for it.
2. Replace needed files.

Place this XML in your project file, preferrably after PropertyGroup definitions in file's start.

{% include JB/gist gist_id="817a4f67b54abea1df21410611286ce5" gist_file="Transform-Application.csproj" %}

For Web projects, use the following:

{% include JB/gist gist_id="817a4f67b54abea1df21410611286ce5" gist_file="Transform-Web.csproj" %}

It has a side effect: while building, it replaces your `Web.config` files with transformed ones (though you can find original content in `Web.Base.config`). Originally, transforms run in MSBuild only when you publish the project and this problem doesn't appear. 
