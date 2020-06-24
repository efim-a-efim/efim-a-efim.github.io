---
layout: project
title: "Keepalived Cluster"
description: "Failover cluster for PostgreSQL and Tomcat based on Keepalived"
sourcelink: https://github.com/efim-a-efim/keepalived-cluster
---
{% include JB/setup %}

Some helper scripts and configs to organize Keepalived cluster with automatic failover and fallback. The main purpose is to prevent splitbrain.

Currently PostgreSQL and Tomcat are supported, but scripts may be used as templates for other services.