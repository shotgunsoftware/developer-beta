---
layout: default
title: Setup Workflow
pagename: spc-setup-workflow
lang: en
---

<script async src="https://unpkg.com/mermaid@8.4.6/dist/mermaid.min.js"></script>

# Setup Workflow

<div class="mermaid" align="center">
graph TB
   a[/Plan Your Setup\]
   b[AWS Account Setup]
   c[Private S3 Bucket]
   d[Closed VPC]
   e[Media Segregation]
   f[Traffic Segregation]
   g["Fine Tuning<br>[Optional]"]
   h[\Done/]
   a-->b
   b-->c
   c-->d
   d-->e
   e-->f
   f-.->h
   f-->g
   g-->h
</div>