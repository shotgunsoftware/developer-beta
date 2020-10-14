---
layout: default
title: Private S3 Bucket
pagename: spc-setup-s3_bucket
lang: en
---

# Private S3 Bucket

{% include info title="Disclaimer" content="The security of your S3 bucket is solely a client responsibility, and the integrity of your data will be at risk without it. We very strongly recommend [securing your S3 bucket properly](https://aws.amazon.com/premiumsupport/knowledge-center/secure-s3-resources/)." %}

## AWS CloudFormation template

It's possible to start from the [Private S3 bucket AWS CloudFormation template](https://github.com/shotgunsoftware/shotgun-vpc/blob/master/sg-private-s3-bucket.yml) and customize it for your needs for a faster deployment.

## Steps

* Create your S3 bucket in your selected region. Please avoid `.` in the bucket name, Shotgun doesn't support them.
* Configure the bucket Default Encryption, as Shotgun uses the bucket Default Encryption to encrypt new S3 objects.

## CORS Configuration

You need to configure CORS policy on your new S3 bucket. Replace mystudio.shotgunstudio.com with your Shotgun hosted site URL.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
 <AllowedOrigin>https://mystudio.shotgunstudio.com</AllowedOrigin>
 <AllowedMethod>GET</AllowedMethod>
 <AllowedMethod>PUT</AllowedMethod>
 <AllowedMethod>HEAD</AllowedMethod>
 <MaxAgeSeconds>3000</MaxAgeSeconds>
 <ExposeHeader>ETag</ExposeHeader>
 <AllowedHeader>*</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```

## IAM Role

Create an AWS Role with the following permissions on your bucket, using the below policies:

* Add a policy to allow Shotgun to access your S3 bucket. Example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ShotgunActions",
            "Effect": "Allow",
            "Action": [
              "s3:AbortMultipartUpload",
              "s3:GetObject",
              "s3:GetObjectAcl",
              "s3:ListMultipartUploadParts",
              "s3:PutObject"
            ],
            "Resource": ["arn:aws:s3:::bucket-name/*"]
        }
    ]
}
```

* Add a policy to allow access to KMS if you are using AWS-KMS S3 encryption. Example:

```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
      "kms:Encrypt",
      "kms:Decrypt",
      "kms:GenerateDataKey"
    ],
    "Resource": [
      "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    ]
  }
}
```

* Allow the Shotgun account to assume the role by editing the role Trust Relationship. Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::468106423547:role/cos_ctr_shotgun-p-ue1-db",
          "arn:aws:iam::468106423547:role/cos_ctr_shotgun-p-ue1-wa",
          "arn:aws:iam::468106423547:role/cos_ctr_shotts-p-ue1"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
