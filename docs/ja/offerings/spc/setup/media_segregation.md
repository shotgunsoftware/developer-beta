---
layout: default
title: Media Segregation
pagename: spc-setup-media_segregation
lang: en
---

# Media Segregation

The media segregation allows your users to access your media in your AWS S3 bucket privately (not transiting on the public internet). 

## Add an S3 endpoint to your VPC

![Add endpoint](../images/spc-endpoint-create-1.png)
![Add endpoint](../images/spc-endpoint-create-2.png)
![Add endpoint](../images/spc-endpoint-create-3.png)

## Set up S3 proxy

You will need to deploy an S3 proxy in your VPC to proxy the traffic from your network into the S3 VPC endpoint. We provide an [S3 proxy CloudFormatiom Template](https://sg-shotgunsoftware.s3-us-west-2.amazonaws.com/tier1/cloudformation_templates/sg-s3-proxy.yml) as a starting point. This will create an ECS Cluster and an ECS service to run the S3 proxy on AWS Fargate behind an AWS ALB.

### Upload the Docker image to a private AWS Docker repositoty


Create an [AWS ECR Repository](https://aws.amazon.com/ecr/) named s3-proxy.

Upload the s3-proxy Docker image to your ECR repository.
 
  * You will need to install Docker on your workstation then run the following commands.
  * Do the docker login using instructions in your AWS Console *View push commands* button.

You will need to change the ECR repository URL to match yours.
  
```
docker pull quay.io/shotgun/s3-proxy:1.0.6
docker tag quay.io/shotgun/s3-proxy:1.0.6 627791357434.dkr.ecr.us-west-2.amazonaws.com/s3-proxy:1.0.6
docker push 627791357434.dkr.ecr.us-west-2.amazonaws.com/s3-proxy:1.0.6
```

### Create CloudFormation stack

Create a new stack in AWS Console using the [S3 proxy CloudFormatiom Template](https://sg-shotgunsoftware.s3-us-west-2.amazonaws.com/tier1/cloudformation_templates/sg-s3-proxy.yml).

### Configure HTTPS

Shotgun only support https S3 proxy. You will need to configure https support on the AWS ALB. 

  * First add a DNS Entry in your domain to access the S3 proxy. ie: https://s3-proxy.mystudio.com.
  * Get a SSL certificate for your url, we recommend using [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/).
  * Configure the HTTPS support on the S3 proxy by adding a new HTTPS listener on the AWS load balancer.

### Test the S3 proxy

Try to access your S3 proxy using the ping route. ie: https://s3-proxy.mystudio.com/ping 

### Configure your stating site to use your S3 proxy.

Go in your test site Preferences menu and set S3 Proxy Host Address to the S3 proxy url. ie: https://s3-proxy.mystudio.com.
Check if you call still access and upload new media.
