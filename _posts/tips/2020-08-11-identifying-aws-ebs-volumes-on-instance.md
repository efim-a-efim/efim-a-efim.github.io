---
layout: post
title: "Identifying AWS EBS volumes on instance"
description: ""
category: "tips"
tags: 
  - aws
  - amazon
  - ebs
  - bash
  - cloud-init
---
{% include JB/setup %}

If you have ever developed a user data script for an AWS instance, you maybe also initialized instances' volumes. And there's always a big problem: how to map AWS volumes to Linux instance's volumes and back? Here's how to solve it for modern AWS instance types (m5/c5/t3 and upper).

## Where the mapping is

In previous versions you could use instance metadata to request device mappings, and in some old times it even worked:

```bash
$ curl http://169.254.169.254/latest/meta-data/block-device-mapping/
/dev/sda1
$ curl http://169.254.169.254/latest/meta-data/block-device-mapping/ebs54
sdb
```

In theory you could map this to kernel's `/dev/sd*` or `/dev/xvd*`. But in modern AWS instance types all volumes are mounted using NVMe emulation, so typical setup looks like:

```
$ ls /dev/nvm*
/dev/nvme0  /dev/nvme0n1  /dev/nvme1  /dev/nvme1n1  /dev/nvme2  /dev/nvme2n1  /dev/nvme3  /dev/nvme3n1  /dev/nvme4  /dev/nvme4n1  /dev/nvme5n1  /dev/nvme5n1p1
```

For the root volume you can use a filesystem label to identify it (for some distributions like Ubuntu):

```
$ ls -lh  /dev/disk/by-label/
total 0
lrwxrwxrwx 1 root root 15 Jul 17 05:54 cloudimg-rootfs -> ../../nvme5n1p1
```

But if you need to bootstrap your instance, you usually want to create filesystems, so you have no labels. And here `nvme-cli` comes. It's a tool that can manipulate NVMe-specific settings and metadata. Install it using `apt` and let's look at one of the volumes:

```
~$ sudo nvme id-ctrl -v /dev/nvme0n1
NVME Identify Controller:
vid     : 0x1d0f
ssvid   : 0x1d0f
sn      : vol00112233445566778
mn      : Amazon Elastic Block Store
............................................
vs[]:
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: 73 64 68 20 20 20 20 20 20 20 20 20 20 20 20 20 "xvda............."
............................................
```

So, here we see 2 important things: in `sn` field there's a serial number and in the very beginning of `vs` field there's a volume name for EC2 attachment. Pay attention to the device names: you need `/dev/nvmeXn1`, not `/dev/nvmeX` as we're looking for volumes, not controllers.

## Scripts

Let's write a simple script that will list all our volumes with their AWS mapping:

```bash
#!/bin/bash
for d in /dev/nvme*n1 ; do
  _attachment="$(sudo nvme id-ctrl -v "$d" | grep '^0000: ' | cut -d '"' -f 2 | tr -d '.')"
  _vol_id="$(sudo nvme id-ctrl -v "$d" | grep '^sn' | cut -d ':' -f 2 | tr -d ' ')"
  printf "%s %25s %5s\n" "$d" "vol-${_vol_id#vol}" "${_attachment}"
done
```

So, now you can use your provisioning tool (like Terraform, Ansible or any other one) to map the devices to their actual attachment and volume IDs. For example, the following script adds such mapping to ansible [local facts](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#local-facts-facts-d):

```bash
_return='{}'

for d in /dev/nvme*n1 ; do
  _attachment="$(sudo nvme id-ctrl -v "$d" | grep '^0000: ' | cut -d '"' -f 2 | tr -d '.')"
  _vol_id="$(sudo nvme id-ctrl -v "$d" | grep '^sn' | cut -d ':' -f 2 | tr -d ' ')"
  _return="$(echo "${_return}" | jq --arg d "$(basename "$d")" --arg v "vol-${_vol_id#vol}" --arg a "${_attachment}" '. * { ($d): { "volume_id": ($v), "attachment": ($a) } }')"
done

echo "${_return}"
```

Put it into `/etc/ansible/facts.d/volume_mapping.fact` with executable permissions (`chmod +x /etc/ansible/facts.d/volume_mapping.fact`) and use the facts like: `{% raw %}{{ ansible_local['volume_mapping']['nvme0n1']['attachment'] }}{% endraw %}`. Reverse mapping is also fairly simple:

```bash
_return='{}'

for d in /dev/nvme*n1 ; do
  _attachment="$(sudo nvme id-ctrl -v "$d" | grep '^0000: ' | cut -d '"' -f 2 | tr -d '.')"
  _vol_id="$(sudo nvme id-ctrl -v "$d" | grep '^sn' | cut -d ':' -f 2 | tr -d ' ')"
  _return="$(echo "${_return}" | jq --arg d "$(basename "$d")" --arg v "vol-${_vol_id#vol}" --arg a "${_attachment}" '. * { ($a): { "volume_id": ($v), "device": ($d) } }')"
done

echo "${_return}"
```
