---
layout: default
title: Fine Tuning
pagename: tier1-setup-tuning
lang: en
---

# Fine Tuning

## Cost Management Recommendations

### S3 Infrequent Access

We recommend enabling S3 Infrequent Access to easily reduce costs without impacting performance. For the {% include product %} Cloud hosted offering, we apply a policy for all objects older than one month.

With Infrequent Access, objects are stored at a lower cost. However, if they are accessed, it will involve an additional cost. {% include product %} has observed that one month was the right policy to use globally, but you may want to adapt that policy to your studio workflows as needed.

Read more about S3 Infrequent Access and other storage classes [here](https://aws.amazon.com/s3/storage-classes/).

## S3 Bucket policy

We recommend you restrict access to your S3 bucket to only your VPC and {% include product %} transcoding services IPs. There is an example policy, replace `your_vpc_id` and `your_s3_bucket` by your values.

We strongly recommend you test media access and media transcoding in your migration test site right after applying the bucket policy changes to be sure your S3 bucket is still accessible from your VPC and from {% include product %} transcoders.

```
{
    "Version": "2012-10-17",
    "Id": "Policy1415115909152",
    "Statement": [
        {
            "Sid": "AllowSSLRequestsOnly",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::your_s3_bucket",
                "arn:aws:s3:::your_s3_bucket/*"
            ],
            "Condition": {
                "Bool": {
                    "aws:SecureTransport": "false"
                }
            }
        },
        {
            "Sid": "Access-to-specific-VPC-only and Shotgun transcoder",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::your_s3_bucket/*",
            "Condition": {
                "NotIpAddress": {
                    "aws:SourceIp": [
                        "34.200.155.69/32",
                        "34.224.232.103/32",
                        "34.202.127.170/32"
                    ]
                },
                "StringNotEquals": {
                    "aws:sourceVpc": [
                        "vpc-2fd62a56",
                        "your_vpc_id"
                    ]
                }
            }
        }
    ]
}
```

## S3 endpoint policy

We recommend setting a VPC endpoint policy on your S3 endpoint to allow access to your S3 bucket only. See [here](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-s3.html#vpc-endpoints-policies-s3) for an example.

## Application Load Balancer

  * We recommend you [enable deletion protection](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancers.html#deletion-protection) on the S3 proxy load balancer to prevent accidental deletion.
  * We recommend you [enable access logging](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html#enable-access-logging) on the S3 proxy load balancer to aid in traffic analysis and identification of security issues.

## Next Steps

See [Migration](./migration.md) to migrate your production site to use the isolation features.

Go to [Setup](./setup.md) for an overview of the possible next steps.
