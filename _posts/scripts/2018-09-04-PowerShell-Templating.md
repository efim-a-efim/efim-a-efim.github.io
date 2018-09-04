---
layout: post
category: scripts
title: "Multiple ways of PowerShell templating"
tags :
 - scripts
 - powershell
---
{% include JB/setup %}

From time to time everyone faces this. You need templates - either HTML, email or just a couple of strings, but anyway... Here I've selected several solutions that may help you. If you know any other good solutions, feel free to mention them in comments - I'll add them to this post.

## ExpandString - fast and dirty

[Original post by NerdyMishka](https://nerdymishka.com/articles/expand-string-in-powershell/)

```powershell
$Name = "Dude"
$ExecutionContext.InvokeCommand.ExpandString('Hello ${Name}!!!');
```

You can use PowerShell execution context on a string to replace variables with their values. It wil actually pass the string through PowerShell interpreter, so you can use any PowerShell syntax inside. The main problems are:

* You must prevent arbitraty code execution, because ExecutionContext supports `$(...)`
* Context has access to all variables within execution scope. So the better way is to prepare the variables and call the context in a separate function.

## String.Replace and Mustache templates

Yes, just replace in a string. Look at the code, it's really simple:

```powershell
function Merge-Tokens($template, $tokens)
{
    return [regex]::Replace(
        $template,
        '\$\{(?<tokenName>\w+)\}\$',
        {
            param($match)

            $tokenName = $match.Groups['tokenName'].Value

            return $tokens[$tokenName]
        })
}
```

There are lots of implementations: [PSTokens](https://github.com/craibuc/PsTokens) and lots of GitHub gists.

## Mustache

There's a very good and straightforward standard called [mustache]{https://mustache.github.io/} that covers around 90% of all "templater" needs. Though it's not officially mentioned, there are some PowerShell implementations, particularly [Postache](https://github.com/baldator/Poshstache) (based on .NET implementation) and [BladePS](https://github.com/dfinke/BladePS) - not really full implementation, but still works. I'd personally recommend Postache.

## EPS - ERB-like templates

So, here's the monster - [EPS](https://github.com/straightdave/eps). It's based on Ruby ERB templates ideas and internally executes PowerShell code in templates. Just read the Readme - it will explain things better than me.

## PSStringTemplate

[This engine's](https://github.com/SeeminglyScience/PSStringTemplate) implements StringTemplate4 - Java template engine with ports to several other languages. The templater docs can be found [here](https://github.com/antlr/stringtemplate4/blob/master/doc/index.md)

# What to choose

For me, EPS and Mustache are the choices. In simple cases and to make the templates compact, I'll use Mustache. But to feel all the power - EPS is the best for me.