---
layout: default
title: Media Traffic Isolation
pagename: tier1-features-media-traffic-isolation
lang: en
---

# Media Traffic Isolation

Communication between your client systems and S3 bucket targets a number of AWS network endpoints and data traverses the open Internet by default. Media Traffic Isolation allows you to limit the number of network endpoints used to transfer data to and from your S3 bucket and optionally restrict access to your AWS VPC or a defined set of public address scopes.

<img alt="media-traffic-isolation-overview" src="../images/media-traffic-isolation-overview.png" width="400">

## Configuration
An S3 Proxy component is deployed within your VPC; which is then used as the endpoint for all S3 communication. It can also be made publicly available using AWS Global Accelerator. 

## How it works
{% include product %} can be configured to use an S3 Proxy address to communicate with your S3 bucket. Deploying the S3 Proxy component within your VPC makes it possible to isolate traffic from the public Internet completely, or to allow more tightly controlled access from the Internet to your media.

<img alt="media-traffic-isolation-arch" src="../images/media-traffic-isolation-arch.png" width="400">

### Secure communication
You are responsible for supplying and renewing SSL certificates for the S3 Proxy component.

## Costs
Activating the Media Traffic Isolation feature will increase your AWS costs. Before activating, be aware that:
1. There are costs associated with running the S3 Proxy component. See [AWS Fargate Pricing](https://aws.amazon.com/fargate/pricing/) for more details.
2. If you choose to make your S3 Proxy publicly accessible, there are also additional costs associated with AWS Global Accelerator. See [AWS Global Accelerator Pricing](https://aws.amazon.com/global-accelerator/pricing) for more details.

## Next Steps
See [Media Traffic Isolation](../setup/media_segregation.md) for setup instructions.
