---
layout: default
title: Connecting Your Studio With Your AWS VPC
pagename: spc-knowledge-connecting
lang: en
---

# Connecting Your Studio With Your AWS VPC

You can connect your studio networks with AWS using one of many options, described in detail in this [Amazon Virtual Private Cloud Connectivity Options](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/network-to-amazon-vpc-connectivity-options.html) whitepaper.

Some of the common options our clients have used include:

## VPN Connection

Using a VPN appliance - AWS-managed or client-managed - your studio can establish a secure connection between your data center (or offices) to your AWS private VPC.


## AWS Direct Connect

[AWS Direct Connect](./direct_connect.md) creates a dedicated link between your studio and your AWS VPC. This will help segregate your studio's network traffic to your private AWS VPC from general internet traffic.

