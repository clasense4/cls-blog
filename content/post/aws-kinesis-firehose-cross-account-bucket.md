+++
title = "AWS Kinesis Firehose Cross Account Bucket"
author = "Fajri Abdillah"
share = true
tags = ["aws", "kinesis", "firehose", "bucket", "cross-account-bucket"]
draft = false
menu = ""
image = "images/matt-popovich-604362.jpg"
slug = "aws-kinesis-firehose-cross-account-bucket"
date = "2017-08-10T12:46:00+07:00"
comments = true
+++

<!--more-->

## Background

I have a request to send data to different account bucket using firehose. Fortunately, it is possible. We will use terraform to achieve our goal. My  terraform version is `0.9.5`.

## AWS Account Up and Running

Setup Account A

```
$ aws configure --profile=Account_A
AWS Access Key ID [None]: .....
AWS Secret Access Key [None]: .....
Default region name [None]: us-east-1
Default output format [None]:
```

Setup Account B

```
$ aws configure --profile=Account_B
AWS Access Key ID [None]: .....
AWS Secret Access Key [None]: .....
Default region name [None]: us-east-1
Default output format [None]:
```

## Project Structure

```
Firehose_Cross_Account_Stream $ tree
.
├── Account_A
│   └── firehose_cross_account.tf
└── Account_B
    └── s3_bucket_policy.tf

2 directories, 2 files
```

## Firehose, Role and Policy in Account A

*firehose_cross_account.tf*

>>> Remember to change **["CHANGETHIS_TO_ACCOUNT_A_ID","CHANGETHIS_TO_ACCOUNT_B_ID"]**

```
resource "aws_iam_role" "firehose_role" {
   name = "CrossAccountRole"
   assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "firehose.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": "",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId":["CHANGETHIS_TO_ACCOUNT_A_ID","CHANGETHIS_TO_ACCOUNT_B_ID"]
        }
      }
    }
  ]
}
EOF
}

resource "aws_iam_role_policy" "firehose_role_policy" {
   name = "CrossAccountPolicy"
   role = "${aws_iam_role.firehose_role.id}"
   policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Action": [
        "s3:AbortMultipartUpload",
        "s3:GetBucketLocation",
        "s3:GetObject",
        "s3:ListBucket",
        "s3:ListBucketMultipartUploads",
        "s3:PutObject",
        "s3:PutObjectAcl"
      ],
      "Resource": [
        "arn:aws:s3:::CrossAccountBucket",
        "arn:aws:s3:::CrossAccountBucket/*"
      ]
    }
  ]
}
EOF
}

resource "aws_kinesis_firehose_delivery_stream" "delivery_stream" {
  count = "1"
  name = "CrossAccountStream"
  destination = "s3"
  s3_configuration {
    role_arn = "${aws_iam_role.firehose_role.arn}"
    bucket_arn = "arn:aws:s3:::CrossAccountBucket"
    buffer_size = "5"
    buffer_interval = "60"
  }
}
```

## S3 Bucket in Account B

*s3_bucket_policy.tf*

>>> Remember to change **CHANGETHIS_TO_ACCOUNT_A_ID**

```
resource "aws_s3_bucket" "bucket" {
    bucket = "CrossAccountBucket"

    lifecycle_rule {
        enabled = true
        prefix = ""

        transition {
            days = 30
            storage_class = "STANDARD_IA"
        }
    }
}

data "aws_iam_policy_document" "b" {
  statement {
    sid = "1"

    effect = "Allow"

    principals {
      type        = "AWS"
      identifiers = ["arn:aws:iam::CHANGETHIS_TO_ACCOUNT_A_ID:role/CrossAccountRole"]
    }

    actions = [
      "s3:AbortMultipartUpload",
      "s3:GetBucketLocation",
      "s3:GetObject",
      "s3:ListBucket",
      "s3:ListBucketMultipartUploads",
      "s3:PutObject",
      "s3:PutObjectAcl"
    ]

    resources = [
      "arn:aws:s3:::${aws_s3_bucket.bucket.bucket}",
      "arn:aws:s3:::${aws_s3_bucket.bucket.bucket}/*"
    ]
  }
}

resource "aws_s3_bucket_policy" "b" {
  bucket = "${aws_s3_bucket.bucket.bucket}"
  policy = "${data.aws_iam_policy_document.b.json}"
}
```
## Orchestrate Account B

First let's move to `Account_B` and run the script.

```
Account_B $ AWS_PROFILE=Account_B terraform plan
...

+ aws_s3_bucket.bucket
...

+ aws_s3_bucket_policy.b
...

<= data.aws_iam_policy_document.b
...


Plan: 2 to add, 0 to change, 0 to destroy.
```

After we are sure, let's apply.

>> REMEMBER : always do plan before apply

```
Account_B $ AWS_PROFILE=Account_B terraform apply
...

aws_s3_bucket.bucket: Creating...
...
aws_s3_bucket.bucket: Creation complete (ID: CrossAccountBucket)
data.aws_iam_policy_document.b: Refreshing state...
aws_s3_bucket_policy.b: Creating...
...
aws_s3_bucket_policy.b: Creation complete (ID: CrossAccountBucket)

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

>>> I found something strange like this, just in case you are experiencing it.
>>> ![ ARN_is_not_correct.png ](/images/aws-kinesis-firehose-cross-account-bucket/ARN_is_not_correct.png "ARN is not correct")
>>> I'm trying to reproduce it again, but no luck.
>>> To fix it, type `terraform apply` again.

## Orchestrate Account A

And move to `Account_A` and run the script.
```
Account_A $ AWS_PROFILE=Account_A terraform plan
...

+ aws_iam_role.firehose_role
    ...
+ aws_iam_role_policy.firehose_role_policy
    ...
+ aws_kinesis_firehose_delivery_stream.delivery_stream
    ...

Plan: 3 to add, 0 to change, 0 to destroy.
```

After we are sure, let's apply.

>> REMEMBER : always do plan before apply

```
Account_A $ AWS_PROFILE=Account_A terraform apply
...

aws_iam_role.firehose_role: Creating...
...
aws_iam_role.firehose_role: Creation complete (ID: CrossAccountRole)
aws_iam_role_policy.firehose_role_policy: Creating...
...
aws_kinesis_firehose_delivery_stream.delivery_stream: Creating...
...

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```

## Run

```
$ aws firehose put-record --delivery-stream-name CrossAccountStream --record '{"Data":"{\"foo\":\"bar\"}\n"}' --profile=Account_A
{
    "RecordId": "+GiNY3nhqcDEyUYRTSvDK7w3UuOuBIWpkli/y71WHsXu0wfa3IA90t586c/bi4pZct57f59VO4JbCFM3pcb5UZzvUMtmywPCeldScYFEiVRSanIZsdvSm5oXd25fuoQQcz3G+cIOXeCyiTuLMeIQdl29vgQiWDi0uwc/1ttQG2mw7uUHWswHwmK2XV3Bu4N+EniMNFwJSS+5uBsDnXxCyIrM0srsxPXs"
}
```

And let's check on Account B S3 Bucket, wait for 60 seconds or more.

![ Result_in_S3_bucket_Account_B.png ](/images/aws-kinesis-firehose-cross-account-bucket/Result_in_S3_bucket_Account_B.png "Result in S3 Bucket Account B")

## Destroying Infra

You need to empty the S3 bucket first.

```
aws s3 rm s3://CrossAccountBucket/ --recursive --profile=Account_B
```

And after the bucket is empty, you can use terraform command on both Account.

```
$ AWS_PROFILE=Account_A terraform plan -destroy
$ AWS_PROFILE=Account_A terraform destroy
$ AWS_PROFILE=Account_B terraform plan -destroy
$ AWS_PROFILE=Account_B terraform destroy
```

## Conclusion

We achieve our goal. Just in case you want to scale your kinesis firehose I also have create [an article](/post/how-to-scaling-aws-kinesis-firehose/). Thanks to terraform, it is easy to automate our case.

## References

1. [http://docs.aws.amazon.com/firehose/latest/dev/controlling-access.html#cross-account-delivery](http://docs.aws.amazon.com/firehose/latest/dev/controlling-access.html#cross-account-delivery)
2. [http://docs.aws.amazon.com/firehose/latest/dev/controlling-access.html#using-iam-s3](http://docs.aws.amazon.com/firehose/latest/dev/controlling-access.html#using-iam-s3)
3. [www.mylinuxguru.com/?p=746](www.mylinuxguru.com/?p=746)
4. [https://www.terraform.io/docs/providers/aws/d/iam_policy_document.html](https://www.terraform.io/docs/providers/aws/d/iam_policy_document.html)
