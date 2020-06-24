---
layout: project
title: "Heartbeat resource agents for AWS"
description: "Resource agents for Heartbeat to switch some AWS resources between machines."
sourcelink: https://github.com/efim-a-efim/heartbeat-aws
---
{% include JB/setup %}

Resource agents for Heartbeat to switch some AWS resources between machines.

Origenally developed for VyOS cluster to make it work with AWS resources failover.

## Supported resource types:
<dl>
<dt>AWS_IP</dt><dd>Instance IP address (in AWS). Supports multiple network interfaces per instance.</dd>
<dt>AWS_EIP</dt><dd>ElasticIP to instance mapping. Supports multiple network interfaces per instance.</dd>
<dt>AWS_Route</dt><dd>AWS VPC route table record changing to point to master node.</dd>
</dl>