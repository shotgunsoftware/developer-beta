---
layout: default
title: Web Traffic Segregation
pagename: spc-setup-traffic_segregation
lang: en
---

# Web Traffic Segregation

The goal is to set up an AWS PrivateLink to privately access your Shotgun site.

## Set up PrivateLink to Shotgun

* Ask Shotgun support to provide you with the Shotgun PrivateLink service name for your AWS region.

* Add a new VPC Endpoint in your VPC

* For the security group, Shotgun service only requires the inbound port tcp/443 to be open.

* Please contact Shotgun Support to get the endpoint approved, so that you can start using it

![Create endpoint](../images/spc-endpoint-create_privatelink.png)

## Split Horizon DNS

You need to configre your office DNS server to resolve your shotgun site to your Shotgun VPC Endpoint DNS name.

Example DNS entry

```
mystudio-staging.shotgunstudio.com   CNAME   vpce-048447456a4f57e14-1j3wh50q.vpce-svc-0b054415458f57634.us-west-2.vpce.amazonaws.com
```
