---
layout: post
category: bookmarks
title: "Link: RDP URI scheme"
tags :
  - windows
  - rdp
  - sysops
  - devops
---
{% include JB/setup %}

Maybe someone knew it already, but I did not...

[RDP URI scheme](https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/clients/remote-desktop-uri) for Windows 8.1+. Works for Android, iOS and Mac too (with official MS remote desktop app).

Short description:

```
rdp://full%20address=s:mypc:3389&audiomode=i:2&disable%20themes=i:1
```

Useful options (replace spaces with `%20`):

* `full address=s:<string>` - host[:port]
* `username=s:<string>`
* `domain=s:<string>`
* `audiomode=i:<0, 1, or 2>` - 2 to disable audio
* `connect to console=i:<0 or 1>` - connect to real console
* `screen mode id=i:<1 or 2>` - 1 for windowed, 2 for fullscreen