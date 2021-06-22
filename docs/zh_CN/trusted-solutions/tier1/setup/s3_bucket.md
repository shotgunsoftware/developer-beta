---
layout: default
title: Media Isolation
pagename: tier1-setup-s3_bucket
lang: en
---

# Media Isolation

{% include info title="Disclaimer" content="The security of your S3 bucket is solely a client responsibility, and the integrity of your data will be at risk without it. We very strongly recommend [securing your S3 bucket properly](https://aws.amazon.com/premiumsupport/knowledge-center/secure-s3-resources/)." %}

## AWS Account Creation

You can quickly [create your AWS Account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/).
You should also contact your AWS contacts to get help with your AWS account setup.

## AWS CloudFormation template

It's possible to start from the [Private S3 bucket AWS CloudFormation template](https://sg-shotgunsoftware.s3-us-west-2.amazonaws.com/tier1/cloudformation_templates/sg-private-s3-bucket.yml) and customize it for your needs for a faster deployment.

{% include info title="Disclaimer" content="This template is provided as an example only. It is your responsibility to validate that running the template will result in the [configuration/policy/security settings your studio requires](https://aws.amazon.com/premiumsupport/knowledge-center/secure-s3-resources/)." %}

  * Go the CloudFormation service in AWS Console
  * Select Template is ready
  * Set Amazon S3 URL to https://sg-shotgunsoftware.s3-us-west-2.amazonaws.com/tier1/cloudformation_templates/sg-private-s3-bucket.yml
  * Next
  * Set a stack name like {% include product %}-s3-bucket
  * Set your S3 bucket name and your {% include product %} site name
  * Next
  * Accept `I acknowledge that AWS CloudFormation might create IAM resources`
  * Next

### CORS Configuration

CORS policy on your S3 bucket will be minimally configured, allowing only the required origin (your site) and methods, amongst other things.

### IAM Role

The template will create an AWS Role with the following permissions on your bucket:

* Allow {% include product %} to access your S3 bucket.
* Allow the {% include product %} account to assume the role by setting the role Trust Relationship.

## Media Isolation Activation

Please contact {% include product %} support via the dedicated Slack channel and provide the following information:
  * S3 bucket name
  * AWS Region
  * {% include product %} Role ARN

{% include product %} will configure your test site to use your own S3 bucket.

## Validation

At this stage, you should be able to upload and download media. The {% include product %} Transcoding Service should also be able to read, transcode and write back the thumbnails, filmstrip and web friendly versions of your media back to your S3 Bucket. To validate this:

1. Log in your Migration Test Site.
2. From the Navigation Bar, go the the Media app
3. Once in the Media App, drag and drop or upload an image or a video from your computer. If you didn't created a Project yet, you may have to create one first.
4. A version should appear, with a thumbnail, in the Media App.
5. Validate that you can playback the media by clicking the Play button.
6. To validate that the media has been stored in your S3 bucket, from the media viewer, click on the cog and then select or hover over ‘view source’. The HTTPS link should contain your bucket name.

## Next Steps

See [Media Traffic Isolation](./media_segregation.md) to activate the Media Traffic Isolation feature.

See [Web Traffic Isolation](./traffic_segregation.md) to activate the Web Traffic Isolation feature.

See [Media Replication](./s3_replication.md) to activate the Web Traffic Isolation feature.

Go to [Setup](./setup.md) for an overview of the possible next steps.

