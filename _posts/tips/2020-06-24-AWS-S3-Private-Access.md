---
layout: post
category: aws
title: "AWS S3 Website with private-only access"
tags :
 - amazon
 - aws
 - s3
 - security
---
{% include JB/setup %}

Hosting a static website on S3 has become a standard solution and there are lots of articles over the Internet describing the setup. But what if you need your website to be accessible only through your internal network? Let's se how to do this :)

## TL;DR

Use [S3 VPC endpoints](https://docs.amazonaws.cn/en_us/vpc/latest/userguide/vpc-endpoints-s3.html) + route tables + bucket policy [with conditions](https://docs.amazonaws.cn/en_us/vpc/latest/userguide/vpc-endpoints-s3.html#vpc-endpoints-s3-bucket-policies) to limit your bucket acces to private nets only.

## Setup description

Assume that you have several VPCs and running instances in them. You need to access your S3 buckets securely and without Internet traffic expenses, so you create S3 endpoints in all VPCs and configure route tables respectively.

At some moment you set up an S3 website. It has DNS name in your private DNS zone, e.g. it's not resolvable from outside of your network. But website setup requires to give all clients access to read S3 objects:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::yoursite.company.io/*"
        }
    ]
}
```

To limit access preserving site functionality, add a Condition to your bucket policy statement(s) that will apply it only if the client comes through particular endpoints:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::yoursite.company.io/*",
            "Condition": {
                "StringEquals": {
                    "aws:sourceVpce": [
                        "vpce-0011223344",
                        "vpce-aabbccddee"
                    ]
                }
            }
        }
    ]
}
```


## Accessing S3 from AWS Client VPN

If you use AWS Client VPN and want to use a setup above, you'll need to set up VPN routing. To do this:

1. Go to any of the VPC route tables with S3 Endpoint route added. Find the route mentioned.
2. Take S3 subnets list from the route's "Destination" field. For me it looked like:
  ```com.amazonaws.us-east-1.s3, 54.231.0.0/17, 52.216.0.0/15, 3.5.16.0/21, 3.5.0.0/20```
3. For each IP subnet (e.g. all except `com.amazonaws.us-east-1.s3`) add a route to your Client VPN Access point configuration. You'll need to add route to each S3 CIDR through each VPN subnet.

After this reconnect to your Client VPN. New routes will contain routes to S3.

**NOTE**: to prevent problems with S3 access, use the S3 endpoint that has no limitations (endpoint policy allows access to any bucket in any account from any client - the default).
