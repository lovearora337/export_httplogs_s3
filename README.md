📘 AWS Logging Project – Apache → CloudWatch → S3


🚀 Project Overview

This project demonstrates how to configure an end-to-end logging pipeline on AWS:

Collect Apache server logs on Amazon Linux

Push logs to Amazon CloudWatch Logs

Export logs to Amazon S3

Apply S3 lifecycle rules for long-term archiving (e.g., Glacier)

This setup provides centralized log management, cost optimization, and observability in a real-world cloud environment.


🛠️ Architecture
Apache Server → CloudWatch Logs → Amazon S3 → Lifecycle Policy → Glacier (Archive)

⚡ Steps Implemented

1️⃣ Apache Log Collection

Installed & configured Apache on Amazon Linux

Enabled access/error logs

2️⃣ CloudWatch Logs Integration
create cloudwatch agent and configure with ec2

Created a CloudWatch Log Group (apache-access-logs)

Configured Apache logs to stream into CloudWatch

3️⃣ Export Logs to S3

Created an export task with AWS CLI:

START_TIME=$(date -u -d '24 hours ago' +%s)000
END_TIME=$(date -u +%s)000

aws logs create-export-task \
  --task-name "export-apache-logs" \
  --log-group-name "apache-access-logs" \
  --from $START_TIME \
  --to $END_TIME \
  --destination "http-logs-cw-s3" \
  --destination-prefix "AWSLogs/<account-id>"

4️⃣ S3 Bucket Policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowGetBucketAcl",
      "Effect": "Allow",
      "Principal": { "Service": "logs.ap-south-1.amazonaws.com" },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::http-logs-cw-s3"
    },
    {
      "Sid": "AllowPutObject",
      "Effect": "Allow",
      "Principal": { "Service": "logs.ap-south-1.amazonaws.com" },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::http-logs-cw-s3/AWSLogs/<account-id>/*",
      "Condition": {
        "StringEquals": { "s3:x-amz-acl": "bucket-owner-full-control" }
      }
    }
  ]
}

5️⃣ S3 Lifecycle Rule

Move logs to Glacier after 30 days

Delete logs after 365 days

Example rule:

{
  "Rules": [
    {
      "ID": "MoveToGlacier",
      "Status": "Enabled",
      "Filter": { "Prefix": "" },
      "Transitions": [
        { "Days": 30, "StorageClass": "GLACIER" }
      ],
      "Expiration": { "Days": 365 }
    }
  ]
}

📂 Sample Log Export

Logs stored in S3:

s3://http-logs-cw-s3/AWSLogs/<account-id>/apache-access-logs/2025/09/15/000000.gz


To read:

aws s3 cp s3://http-logs-cw-s3/AWSLogs/<account-id>/apache-access-logs/2025/09/15/000000.gz ./logs.gz
gunzip logs.gz
cat logs

✅ Skills Gained

AWS CloudWatch Logs

S3 bucket policies & lifecycle rules

IAM permissions debugging

Log pipeline design

Observability best practices
