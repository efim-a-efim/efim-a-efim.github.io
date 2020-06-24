---
layout: project
title: "GlusterFS SNMP sub-agent"
description: "GlusterFS monitoring through SNMP"
sourcelink: https://github.com/efim-a-efim/glister-snmp
---
{% include JB/setup %}

SNMP sub-agent for net-snmp to monitor GlusterFS status. Uses `pass_persist` facility to integrate woth SNMP agent.

Example usage:

    ## Security
    # Root of Gluster monitoring
    view	gluster		included .1.3.6.1.4.1.2312.11
    # security restrictions
    com2sec local           localhost       gluster
    access configGroup	""	v2c		noauth		exact	gluster		gluster		all

    ## Basic configuration
    syslocation TestBench
    syscontact Root <root@localhost>
    dontLogTCPWrappersConnects yes
    
    ## Call gluster-snmp
    # SNMP root ID must be equal to the one in view above
    pass_persist .1.3.6.1.4.1.2312.11 /usr/bin/python -u /usr/local/libexec/gluster-snmp/glusterfs.py
