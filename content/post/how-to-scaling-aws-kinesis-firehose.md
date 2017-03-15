+++
title = "How to Scaling AWS Kinesis Firehose"
author = "Fajri Abdillah"
share = true
tags = ["aws", "kinesis", "firehose"]
draft = false
menu = ""
image = "images/matt-popovich-604362.jpg"
slug = "how-to-scaling-aws-kinesis-firehose"
date = "2017-03-14T00:25:00+07:00"
comments = true
+++

I explore how to scale aws kinesis firehose.

<!--more-->

## Background

I was given task to create unlimited log pipeline that can scale easily. So the plan is using aws kinesis firehose and S3 as the destination.

Here is overview what we are going to built.

![ scaling_firehose_diagram.png ](/images/how-to-scaling-aws-kinesis-firehose/scaling_firehose_diagram.png "Scaling Firehose Diagram")

The producer, can be any application. But remember, kinesis firehose is not available yet in every region. So, if your application is in Asia, there will be latency to US / EU around 180 - 250ms.

One big question in my mind is, "How can I scale this firehose after see the limitations?". Technically, 5000 records/second is not a small number. But right now, data can increasing without notice to us first. Especially when we want to save log of application. So, my choice is using S3 to save the log. I skipped on using object lifecycle, you can read more about object lifecycle [aws version](http://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html) and [terraform version](https://www.terraform.io/docs/providers/aws/r/s3_bucket.html).

## Limitation

Amazon Kinesis Firehose has the following limits.

- By default, each account can have up to 20 Firehose delivery streams per region. This limit can be increased using the Amazon Kinesis Firehose Limits form.
- By default, each Firehose delivery stream can accept a maximum of 2,000 transactions/second, 5,000 records/second, and 5 MB/second. You can submit a limit increase request using the Amazon Kinesis Firehose Limits form. The three limits scale proportionally. For example, if you increase the throughput limit to 10MB/second, the other two limits increase to 4,000 transactions/second and 10,000 records/second.

>>> Important  
>>> If the increased limit is much higher than the running traffic, this causes very small delivery batches to destinations, which is inefficient and can be costly. Be sure to increase the limit only to match current running traffic, and increase the limit further if traffic increases.

- Each Firehose delivery stream stores data records for up to 24 hours in case the delivery destination is unavailable.
- The maximum size of a record sent to Firehose, before base64-encoding, is 1000 KB.
- The PutRecordBatch operation can take up to 500 records per call or 4 MB per call, whichever is smaller. This limit cannot be changed.
- The following operations can provide up to 5 transactions per second CreateDeliveryStream, DeleteDeliveryStream, DescribeDeliveryStream, ListDeliveryStreams, and UpdateDestination.
- The buffer sizes hints range from 1 MB to 128 MB for Amazon S3 delivery, and 1 MB to 100 MB for Amazon Elasticsearch Service delivery. The size threshold is applied to the buffer before compression.
- The buffer interval hints range from 60 seconds to 900 seconds.
- For Firehose to Amazon Redshift delivery, only publicly accessible Amazon Redshift clusters are supported.
- The retry duration range is from 0 seconds to 7200 seconds for Amazon Redshift and Amazon ES delivery.

## Aws CLI Up & Running

```
 $ aws configure
AWS Access Key ID [********************]:
AWS Secret Access Key [********************]:
Default region name [us-east-1]: us-west-2
Default output format [None]:
```

## Terraform Up & Running

I use mac os version, so to install it just type :

```
brew install terraform
```

And my terraform version is `terraform-0.8.8`.

For another OS, you can refer [here](https://www.terraform.io/downloads.html).

This is list of variables that we want to use on our terraform project. Here I name it, `terraform.tfvars`. More information on this, [https://www.terraform.io/docs/configuration/variables.html](https://www.terraform.io/docs/configuration/variables.html).

```
#--------------------------------------------------------------
# General
#--------------------------------------------------------------
region            = "us-west-2"
firehose_count    = 2
log_group_name    = "scaling-firehose"
log_stream_name   = "scaling-firehose"
```

This is list of resource that we want to use on our terraform project. We plan to add 1 S3 buckets, 1 new rule and 2 kinesis firehose delivey stream with same S3 destination. Here I name it, `scaling_firehose.tf`. More information on this, [https://www.terraform.io/docs/configuration/resources.html](https://www.terraform.io/docs/configuration/resources.html)

```
variable "region"            { }
variable "firehose_count"    { }
variable "log_group_name"    { }
variable "log_stream_name"   { }

provider "aws" {
  region = "${var.region}"
}

resource "aws_s3_bucket" "bucket" {
    bucket = "scaling-firehose"
}

resource "aws_iam_role" "firehose_role" {
   name = "firehose_example_role"
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
      "Sid": ""
    }
  ]
}
EOF
}

resource "aws_iam_role_policy" "firehose_role_policy" {
   name = "firehose_example_role_policy"
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
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::scaling-firehose",
        "arn:aws:s3:::scaling-firehose/*",
        "arn:aws:s3:::%FIREHOSE_BUCKET_NAME%",
        "arn:aws:s3:::%FIREHOSE_BUCKET_NAME%/*"
      ]
    }
  ]
}
EOF
}

resource "aws_cloudwatch_log_group" "log_group" {
  name = "${var.log_group_name}"
}

resource "aws_cloudwatch_log_stream" "log_stream" {
  name           = "${var.log_stream_name}"
  log_group_name = "${aws_cloudwatch_log_group.log_group.name}"
}

resource "aws_kinesis_firehose_delivery_stream" "firehose_stream" {
  count = "${var.firehose_count}"
  name = "scaling-firehose-${count.index + 1}"
  destination = "s3"
  s3_configuration {
    role_arn = "${aws_iam_role.firehose_role.arn}"
    bucket_arn = "${aws_s3_bucket.bucket.arn}"
    buffer_size = 5
    buffer_interval = 60

    cloudwatch_logging_options {
      enabled = "true"
      log_group_name = "${aws_cloudwatch_log_group.log_group.name}"
      log_stream_name = "${aws_cloudwatch_log_stream.log_stream.name}"
    }
  }
}
```

And after we are done with the configuration, we can try `terraform plan`.

```
$ terraform plan
...

+ aws_cloudwatch_log_group.log_group
...

+ aws_cloudwatch_log_stream.log_stream
...

+ aws_iam_role.firehose_role
...

+ aws_iam_role_policy.firehose_role_policy
...

+ aws_kinesis_firehose_delivery_stream.firehose_stream.0
...

+ aws_kinesis_firehose_delivery_stream.firehose_stream.1
...

+ aws_s3_bucket.bucket
...


Plan: 7 to add, 0 to change, 0 to destroy.
```

Then if we are confident with the plan, we can try to apply the plan with command `terraform apply`.

```
$ terraform apply
aws_cloudwatch_log_group.log_group: Creating...
...

aws_iam_role.firehose_role: Creating...
...

aws_s3_bucket.bucket: Creating...
...

aws_cloudwatch_log_group.log_group: Creation complete
aws_cloudwatch_log_stream.log_stream: Creating...
...
aws_iam_role.firehose_role: Creation complete
aws_iam_role_policy.firehose_role_policy: Creating...
...
aws_cloudwatch_log_stream.log_stream: Creation complete
aws_iam_role_policy.firehose_role_policy: Creation complete
aws_s3_bucket.bucket: Still creating... (10s elapsed)
aws_s3_bucket.bucket: Creation complete
aws_kinesis_firehose_delivery_stream.firehose_stream.0: Creating...
aws_kinesis_firehose_delivery_stream.firehose_stream.1: Creating...
aws_kinesis_firehose_delivery_stream.firehose_stream.0: Still creating... (3m0s elapsed)
aws_kinesis_firehose_delivery_stream.firehose_stream.0: Creation complete

Apply complete! Resources: 7 added, 0 changed, 0 destroyed.

The state of your infrastructure has been saved to the path
below. This state is required to modify and destroy your
infrastructure, so keep it safe. To inspect the complete state
use the `terraform show` command.

State path: terraform.tfstate
```

Go to aws console, and this is what we get.

![ aws_console.png ](/images/how-to-scaling-aws-kinesis-firehose/aws_console.png "2 Firehose Delivery Stream")

## Test using AWS CLI

```
 $ aws firehose put-record --delivery-stream-name scaling-firehose-1 --record '{"Data":"{\"foo\":\"bar\"}"}'
{
    "RecordId": "PoKUUQ4805aidLc3CUH..."
}
 $ aws firehose put-record --delivery-stream-name scaling-firehose-2 --record '{"Data":"{\"foo2\":\"bar2\"}"}'
{
    "RecordId": "kjoYUT2YJt3VNcxCrSH..."
}
```

And this is what we get

![ S3_result.png ](/images/how-to-scaling-aws-kinesis-firehose/S3_result.png "S3 Result")

## Test using AWS Python SDK

This is the python code, we will test using 1 delivery stream first.

```
import boto3

client = boto3.client('firehose', region_name='us-west-2')

message = '{"foo":"bar"}' + "\n"
response = client.put_record(
    DeliveryStreamName='scaling-firehose-1',
    Record={
        'Data': message
    }
)
```

And this is what we get

![ S3_result_python_1.png ](/images/how-to-scaling-aws-kinesis-firehose/S3_result_python_1.png "S3 Result")

After we are confident with the code, we can try this

```
import boto3
import random

client = boto3.client('firehose', region_name='us-west-2')

for x in xrange(1,100):
    delivery_streams = ['scaling-firehose-1','scaling-firehose-2']
    delivery_stream = random.choice(delivery_streams)

    print delivery_stream

    message = '{"foo":"bar"}' + "\n"
    response = client.put_record(
        DeliveryStreamName=delivery_stream,
        Record={
            'Data': message
        }
    )
```

```
$ python firehose.py
scaling-firehose-2
scaling-firehose-2
scaling-firehose-2
scaling-firehose-1
scaling-firehose-1
scaling-firehose-1
...
```

As you can see, we are randomizing the delivery stream, let's checkout the S3.

![ S3_result_python_2.png ](/images/how-to-scaling-aws-kinesis-firehose/S3_result_python_2.png "S3 Result")

## Destroying Infra

You need to empty the S3 bucket first.

```
aws s3 rm s3://scaling-firehose/ --recursive
```

And after the bucket is empty, you can use terraform command.

```
terraform plan -destroy
terraform destroy
```

## Conclusion

This is just a proof of concept, and if this is going well, when I want to add new delivery stream, what I need to do is edit `terraform.tfvars` and change the value of `firehose_count` into desired value. Type `terraform plan` check if it return error, and `terraform apply`. Then add list of current delivery stream to `firehose.py`. Creating delivery stream is taking time, so need to be careful about deploying your new code.
