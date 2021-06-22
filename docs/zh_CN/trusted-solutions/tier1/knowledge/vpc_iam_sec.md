---
layout: default
title: VPC / IAM / Security Group
pagename: tier1-knowledge-vpc_iam_sec
lang: en
---

# VPC / IAM / Security Group

[Amazon Virtual Private Cloud](https://aws.amazon.com/vpc/) permits users to logically separate virtual networks that host their AWS resources and provides you complete control over access to your AWS network.

Within a VPC, [security groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) act as a basic firewall and control what inbound and outbound connections are permitted to each given resource. For example, a security group can allow inbound **HTTPS** traffic to a proxy server but block all other inbound traffic.

With [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/), access to AWS resources and services can be controlled at a more fine-grained level. For example, IAM can be leveraged to control who or which resources can access S3 buckets used by {% include product %}.

All three of the above features are used in the {% include product %} isolation features implementation to ensure that you securely connect your closed VPC to {% include product %} and allow access to the [media S3 buckets](../setup/s3_bucket.md).
