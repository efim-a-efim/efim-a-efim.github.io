---
layout: post
category: bookmarks
title: "Link: Template files substituter for Docker"
tags :
 - docker
 - devops
---
{% include JB/setup %}

[Docker-gen](https://github.com/jwilder/docker-gen) is a templates-based files generator that uses Docker container metadata.

May be used both as a separate image and within Docker image.

Example: [nginx-proxy](https://github.com/jwilder/nginx-proxy) - uses Foreman (procfile-based process controller). [How it works](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/).