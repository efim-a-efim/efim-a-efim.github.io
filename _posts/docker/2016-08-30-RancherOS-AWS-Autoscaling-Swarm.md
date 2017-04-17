---
layout: post
category: docker
title: "Configuring RancherOS for use with AWS autoscaling + Swarm cluster"
tags :
 - docker
 - scripts
 - swarm
 - rancheros
 - amazon
 - aws
---
{% include JB/setup %}

## Story and disposition

In my new project I had to implement Docker Swarm cluster on AWS. I know about AWS ECS, but it didn't support autoscaling based on system load at the moment, so it looked easier to use Swarm.

There's a problem here. Swarm (at least, before docker 1.12) doesn't support autoscaling and automatic rebalancing, so if you've started containers on 1 instance and than added 10 new instances, it won't migrate the containers until the first instance would fail.

Another problem is autoscaling itself. Cloud is quite specific piece of infrastructure: you have to store "state" of your app separately and create a "stateless" config to launch nodes dynamically.

So, here's the challenge. Let's go :)

## RancherOS Docker configuration

To configure Docker in RancherOS, you need to change `rancher.docker.args` parameter in config and *restart Docker*. The problem is that you cannot correctly restart Docker automatically after it has started. It may have some containers that will be lost after restart.

Solution is to use [system-docker](http://docs.rancher.com/os/configuration/docker/) to run a container with scripts that will reconfigure Docker *before* actual docker start. I created an image [deadroot/rancheros-ec2-metadata](https://hub.docker.com/r/deadroot/rancheros-ec2-metadata/) for  this. See [this article](/projects/rancheros-ec2-metadata) and [GitHub project](https://github.com/efim-a-efim/rancheros-ec2-metadata).

## AWS Autoscaling

To create a "stateless" autoscaling configuration, we need to:

1. Create [user-data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) that will correctly launch our instance with all configuration.
2. Take configuration of the instance from AWS instance's data where possible.

### AWS tags

I've configured AWS autoscaling group the way it creates several tags:

- `docker.environment` - "environment": staging, production, qa, etc.
- `docker.role` - instance role. I use 'frontend', 'db', 'queue', 'services', 'web', etc.

Also there are some special tags. They are used to bypass config variables from AWS autoscaling group to Rancher's environment:

- `example.db.host` - database host address
- `example.queue.host` - queue host address

Here you may use any variables that should be configured manually. For example, you can bypass DNS name of your env, EFS filesystem name, etc. See User-data below.

### User-data

RancherOS can take its config from EC2 instance's [user-data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html). Here's my user-data sample that configures "generic" instance for Swarm cluster:

{% include JB/gist gist_id="7e887cfb597a5a7fd7128afa623ff829" %}

Here:

* `consul://consul.example.internal:8500` - discovery address
* `docker.example.internal:5000` - Docker private registry address

What it does:

1. Load AWS metadata and tags. Look at [deadroot/rancheros-ec2-metadata](https://hub.docker.com/r/deadroot/rancheros-ec2-metadata/) for details. Here it:
  - Imports all AWS metadata to environment;
  - Adds all tags beginning with `docker.` to Docker host labels
  - Adds tag named `example.db.host` as environment variable `DB_HOST` and `example.queue.host` as `QUEUE_HOST`
2. Launches Swarm agent and joins to the Swarm cluster.

## Conclusion

Here we configured a Swarm cluster "node". You can create several autoscaling groups with different tags and use them to launch dirfferently-tagged instances. After this you can use [Swarm filters](https://docs.docker.com/swarm/scheduler/filter/) to launch containers on specific instances.
