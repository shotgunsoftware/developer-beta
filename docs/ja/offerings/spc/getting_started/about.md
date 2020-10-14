---
layout: default
title: What is Shotgun Private Cloud?
pagename: spc-getting_started-about
lang: en
---

# What is Shotgun Private Cloud

Shotgun Private Cloud combines our Cloud Hosted Platform with client-managed AWS resources to provide a solution that satisfies the most stringent security and privacy requirements. Clients retain control of their sensitive content without having to host Shotgun on their infrastructure.

Shotgun Private Cloud delivers the following advantages compared to our Standard offering:

* Hosting of assets and attachments in a **client-owned S3 Bucket**
* **Web traffic isolation** from the public internet
* **Media traffic isolation** from the public internet
* Access to fully managed Shotgun Cloud Services
* Automatic and continuous version upgrades
* Ephemeral compute + in-memory segration between clients

In a nutshell, this means that with Shotgun Private Cloud, your Shotgun site and the data related to it cannot be reached by anyone outside of your studio network.

Shotgun Private Cloud is a solution that requires less upkeep, as well as less IT/System Administrator knowledge and skills, than hosting Shotgun on-premise. The list of advantages compared to on-premise includes, but is not limited to:

* No Shotgun specific knowledge required
* No manual Shotgun updates required
* Very low level of maintenance required for the AWS components

## What Shotgun Private Cloud is not

Shotgun Private Cloud is not a completely isolated solution. Both the compute services and the database services are shared amongst clients, and managed by Shotgun. From a hardware standpoint, Shotgun Private Cloud does not guarantee complete segregation.

# High Level Architecture
![spc-arch](../images/spc-about-arch.png)

Shotgun Cloud offerings can be decoupled at a high level in 3 parts:

**Compute Stack:** The part of the Shotgun Service that handles client requests and serves data to the client.

**Data Stack:** Metadata storage (databases).

**Media Storage:** Where the client's attachments, media, and assets are stored. Shotgun uses AWS S3 to store client content.

Please read [Securing Studio IP in AWS: Cloud-based VFX Project Management with Autodesk Shotgun blog post](https://aws.amazon.com/blogs/media/securing-studio-ip-in-aws-cloud-based-vfx-project-management-with-autodesk-shotgun/) for more details.

## Ephemeral compute and memory segregation
While Shotgun Private Cloud clients share the same infrastructure, Shotgun Cloud guarantees a complete in-memory isolation, both in transit and at rest, of client data. This makes Shotgun Cloud less prone to data leaking due to architecture flaws or software vulnerabilities exploiting memory, like buffer overflow.
