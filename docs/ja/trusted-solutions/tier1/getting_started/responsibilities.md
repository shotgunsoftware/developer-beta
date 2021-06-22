---
layout: default
title: Client Responsibilities
pagename: tier1-getting_started-responsibilities
lang: en
---
  
# Client Responsibilities

Below, we have outlined setup responsibilities between Autodesk and You. 

## Isolation Setup

You are entirely responsible for the validity, security, and execution of the Isolation setup in Your AWS Account. Autodesk should not, under any circumstances, be granted access to Your AWS environment.
 
Autodesk is available during the process for assistance, but the configuration of Isolation features in Your AWS Account is to be executed by You on Your own.

Isolation feature set activation requires the {% include product %} Support team's intervention. Activation delays are to be expected and will depend on demand. You understand that an estimated period of 2-8 weeks is usually required to complete the setup necessary to implement the isolation feature set. The setup time is highly dependent on your cooperation, so please plan to dedicate resources for the setup before beginning the onboarding process.

Autodesk does not guarantee any timeline for setup completion.

## Onboarding

|Type|	Description / Agreement |	Responsibility	| Available for Assistance|
|--------|-----|----------|---------|
|AWS Knowledge	|	Acquiring the AWS-specific knowledge required to set up the isolation features.	|You	|N/A|
|S3|Setting up the S3 Bucket that will host Your media Securing access to the S3 Bucket. Additional high-availability measures (versioning, bucket replication, etc.)	|You	|{% include product %} and *AWS|
|Closed VPC	|Setting up DirectConnect/VPN, etc. to allow closed access to the VPC. Securing the VPC by putting the correct Security Groups in place.	|You	|*AWS |
|Media Isolation	|Creating the S3 end-points. Deploying the S3 Proxy.	|You|	{% include product %} and *AWS |
|Traffic Isolation	|Creating VPCs. Creating Subnets.|	You|{% include product %}|
|Private Access Point|Checking that the access point is only available from Your network.|	{% include product %}|	N/A|
|Monitoring and Reliability|Maintaining uptime up to Autodesk standards. High availability and redundancy of Cloud Services. Metadata and database resiliency and redundancy. Maintaining Recovery Point Objective (RPO) for metadata and database.	|{% include product %}|N/A|
|Service Level Objective|Maintaining {% include product %} target RPO and RTO (See [{% include product %} Security White Paper](https://support.shotgunsoftware.com/hc/en-us/articles/114094526153-Shotgun-security-white-paper) for more details).|{% include product %}|	N/A|
|Security and Governance |Maintaining the {% include product %} Cloud Services that Isolation clients are interfacing with, so that they are meeting expectations in terms of security, vulnerability patching, scanning, auditing, etc. (See [{% include product %} Security White Paper](https://support.shotgunsoftware.com/hc/en-us/articles/114094526153-Shotgun-security-white-paper) for more details).|	{% include product %}	|N/A|

*You are solely responsible to seek or obtain any support services AWS may provide under any existing relationship between You and AWS. Autodesk teams are not parties to Your relationship with AWS and therefore not responsible or liable for any services or lack thereof provided by AWS to You. 
