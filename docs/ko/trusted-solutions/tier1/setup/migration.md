---
layout: default
title: Migration
pagename: tier1-setup-migration
lang: en
---

# Migration

Once everything is configured and properly tested with the migration test site, it's now time to migrate your production site to use the isolation features.

## Test migration

Ask the {% include product %} team to start the migration process in support ticket/slack.

  * {% include product %} will clone your production site database to your migration test site.
  * You will do a first sync of the media from {% include product %}'s S3 bucket to your bucket. {% include product %} will provide the exact instructions.
  * You can now test your site to be sure your existing media is available.

## Final migration

The second test is to definitly migrate your site to use your own S3 bucket.

  * You will do a second sync of the media from {% include product %}'s S3 bucket to your bucket.
  * {% include product %} will reconfigure your hosted site with media isolation. Some media will be missing until the final media sync is completed.
  * You will do a final media sync.

