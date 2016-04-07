---
layout: project
title: "RedHat cluster agents"
description: "RedHat cluster agents for Proxmox VE and other things"
sourcelink: https://github.com/efim-a-efim/redhat-cluster
---
{% include JB/setup %}

RedHat cluster agents. Currently supported:

## Fencing agents:
- fence_pvevm - Proxmox VE KVM node fencing.
- fence_pvect - Proxmox VE OpenVZ container fencing.

## Resource agents:
- GlusterFS
- Tomcat
- PostgreSQL DB server
- PostgreSQL master role
- PostgreSQL pg_receivexlog service failover

Some services (e.g. PostgreSQL) need special configuration to work with resource agents. Refer to `/conf` directory of source for examples.  