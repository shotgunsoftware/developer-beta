---
layout: default
title: Shotgun AWS Direct Connect Onboarding
pagename: spc-knowledge-direct_connect_onboarding
---

# Shotgun AWS Direct Connect Onboarding


## Introduction

AWS Direct Connect (DX) is used to establish private connectivity between AWS and an on-prem facility. DX provides a private, high bandwidth network connection between your network and AWS Virtual Private Cloud (VPC) and bypasses the public internet. AWS has established 100 Direct Connect locations globally and leverages the AWS Partner Network to extend the footprint. 

## Review of Shogun Configuration in AWS

![high-level-architecture](../images/spc-arch-s3_ue1.png)

## Customer Types

1. Customer already has AWS Direct Connect dedicated connection
2. Customer has equipment and network presence in an AWS Direct Connect location
3. Customer does not have equipment or presence in an AWS Direct Connect location


## Setup Options

1. Request a dedicated Direct Connect connection through AWS Console
    1. Provision the required connectivity yourself
    1. Work with Direct Connect Partner to help establish a dedicated connection to AWS equipment
2. Request a hosted Direct Connect connection through AWS Direct Connect Partner


## Criteria to Determine Setup Path 

If you answer “yes” to the following, then request a dedicated Direct Connect connection through the AWS Console (Option 1a):

- Do you already have equipment and presence in an AWS Direct Connect location?
- Do you know the process for requesting a cross-connect within the Direct Connect location facility?
- Are you looking for any one of the following - 1Gbps, 10Gbps port, or a dedicated connection?

If you answer “yes” to the following, then request a dedicated Direct Connect connection through the AWS Console and select a Partner to assist (Option 1b):

- Are you planning to use AWS Direct Connect to connect to other AWS resources outside of Shotgun?
- Do you have the time and resources to complete the setup?
- Are you looking for any one of the following - 1Gbps, 10Gbps port, or a dedicated connection?

If you answer “yes” to the following, then you should work with an AWS Direct Connect Partner to request a hosted Direct Connect connection (Option 2):

- Are you already working with an AWS Direct Connect Partner?
- Do you want a Partner to facilitate the setup?
- Are you looking for a port less than 1Gbps or a hosted connection?
***Disclaimer:***  *All options are valid and the criteria are just a guide to help simplify the selection process. You can still pick any option based on more specific criteria.*

## Setup Directions

### Request through AWS Console - Option 1 (a and b)
1. [Create a Connection in the AWS Console](https://docs.aws.amazon.com/directconnect/latest/UserGuide/getting_started.html#ConnectionRequest)
1. [Download the LOA-CFA](https://docs.aws.amazon.com/directconnect/latest/UserGuide/getting_started.html#DedicatedConnection). The LOA is the authorization to connect to AWS and is required to establish the cross-network connection.
1. (Option 1a only) Request cross-connects at AWS Direct Connect locations. Find contact information [here](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Colocation.html).
1. (Option 1b only) [Reach out to an AWS Partner](https://aws.amazon.com/directconnect/partners/) and share the LOA with them.
1. Once the dedicated connection is provisioned into your account, set up logical connectivity (Virtual Interfaces).

### Request through AWS Direct Connect Partner - Option 2

1. [Reach out to an AWS Partner](https://aws.amazon.com/directconnect/partners/). The criteria for choosing an AWS Partner are:
    - AWS Region
    - Providers
    - If you are already working with an AWS Direct Connect Partner
1. If hosted connection, [accept a hosted connection](https://docs.aws.amazon.com/directconnect/latest/UserGuide/getting_started.html#ConnectionRequest). More information can be found [here](https://docs.aws.amazon.com/directconnect/latest/UserGuide/accept-hosted-connection.html).
1. Once the hosted connection is provisioned into your account, set up logical connectivity (Virtual Interfaces).

## FAQ

**How long should it take to set up AWS Direct Connect?**

Short Answer - It depends. A lot of factors go into the time it takes to set up AWS Direct Connect. The timeline can vary from a few days to a few months. Some of the factors include current infrastructure, location of equipment, providers, partners, and more. If you are looking to expedite the process, consider using an AWS Direct Connect Partner who is geographically nearby.

**Further questions about AWS Direct Connect?**

[AWS Direct Connect FAQs](https://aws.amazon.com/directconnect/faqs/?nc=sn&loc=6)



## VPN

A site-to-site VPN can be used as an alternative to AWS Direct Connect. Learn more about [AWS VPN here](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html).

## AWS Documentation & Resources

- [What is AWS Direct Connect?](https://www.youtube.com/watch?v=eNxPhHTN8gY&feature=youtu.be&t=716)
- [Direct Connect User Guide](https://docs.aws.amazon.com/directconnect/latest/UserGuide/dc-ug.pdf)
- [re:Invent 2018 - 400 level Deep Dive on Direct Connect](https://www.youtube.com/watch?v=DXFooR95BYc)
- [AWS Direct Connect Locations](https://aws.amazon.com/directconnect/features/#AWS_Direct_Connect_Locations)
- [AWS Direct Connect Partners](https://aws.amazon.com/directconnect/partners/)
