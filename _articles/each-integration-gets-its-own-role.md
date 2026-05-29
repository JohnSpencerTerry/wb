---
layout: default
title: "Each integration gets its own role."
date: 2026-05-29
category: Software Engineering
draft: false
---

## The shared `svc-integrations` role worked until the auditor asked a question

StartupTechCo runs a fleet of vendor integrations: two flat-file vendors it pulls from over SFTP, a third that pushes files into an SFTP endpoint it hosts, a payments vendor it pulls from over SSH, and half a dozen HTTPS API integrations. All of it runs on a couple of Fargate workers. All of those workers assume the same task role, `svc-integrations`. The role's policy covers read/write on every integration's S3 prefix and `secretsmanager:GetSecretValue` on the secrets holding the SFTP keys and API keys.

The setup had two problems.

The first was attribution. An auditor asked which integration accessed `vendor-a-incoming/` at a specific timestamp. CloudTrail showed `svc-integrations` for every S3 call. There was no way to map the call to a specific integration.

The second was blast radius. A refactor in the vendor-B integration introduced a paginated `ListObjectsV2` call with the wrong prefix variable: it listed `vendor-a-incoming/` instead of `vendor-b-incoming/`. The change deployed. It ran on every vendor-B cron tick for two weeks. Vendor-B's downstream processor filtered by filename pattern and silently skipped everything that didn't match, so nothing broke. The call returned normally each time, because `svc-integrations` had access to both prefixes. A per-integration role would have failed at the first execution with `AccessDenied`.

## The blast radius of a shared credential is the whole fleet

Least privilege doesn't get you isolation when every integration uses the same identity. The vendor-B bug listed vendor-A's prefix because the IAM role the workload ran under had access to that prefix. Tightening the policy doesn't help: every integration runs under the same policy, so the only way to deny vendor-B access to vendor-A's prefix is to deny everyone, which breaks vendor-A's own integration.

The auditor's question and the cross-integration bug are the same property in different shapes. Shared identity collapses attribution and isolation into one credential. There is no policy you can attach to a single IAM role that recovers either.

## Per-integration isolation means one IAM role per compute unit

The shortcut is to keep the shared role and scope per-integration access through `StringEquals` conditions on a session tag or principal tag. This adds complexity to the policy without changing the blast radius. The fix is to give each integration its own deploy target with its own role.

At StartupTechCo, this meant two changes that come together: each integration moved onto its own compute (Lambda for the HTTPS APIs, Fargate for the long-lived SFTP/SSH transfers), and each got its own IAM role. The role's policy mentions only the resources the integration touches: its S3 prefix, the one Secrets Manager secret holding its credential, and the KMS key that encrypts that secret.

The vendor-A Fargate task role looks like this:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VendorAIncomingObjects",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::integrations-prod/vendor-a-incoming/*"
    },
    {
      "Sid": "VendorAIncomingList",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::integrations-prod",
      "Condition": {
        "StringLike": {"s3:prefix": ["vendor-a-incoming/*"]}
      }
    },
    {
      "Sid": "VendorASFTPSecret",
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:integrations/vendor-a/sftp-key-*"
    },
    {
      "Sid": "VendorASecretKMS",
      "Effect": "Allow",
      "Action": "kms:Decrypt",
      "Resource": "arn:aws:kms:us-east-1:123456789012:key/11111111-2222-3333-4444-555555555555"
    }
  ]
}
```

Vendor-A's Fargate task pulls files from the vendor's SFTP endpoint, writes them to `s3://integrations-prod/vendor-a-incoming/`, and reads the SFTP private key from one named secret. The `kms:Decrypt` statement is in the policy because the secret uses a customer-managed KMS key; with the default AWS-managed `aws/secretsmanager` key, that statement wouldn't be needed. Vendor-B's task role doesn't reference `vendor-a-incoming/` anywhere. The accidental `ListObjectsV2` against that prefix returns `AccessDenied` on the first execution.

## The compute boundary is the credential boundary

Each integration deals with two credentials. The first is the AWS identity the workload runs as: the Fargate task role or Lambda execution role. The AWS SDK handles this automatically; the integration code never sees an AWS access key. The second is the vendor credential: the SFTP private key, the API key. The integration code uses the first to fetch the second from Secrets Manager at runtime.

For the Lambda API integrations, the execution role is attached at deploy time. The function calls `GetSecretValue` at start to pull the vendor API key, holds it in memory for the lifetime of the execution environment, and passes it into the outbound HTTP client (`requests`, `httpx`, or equivalent).

For the Fargate SFTP tasks, the task role is attached when the task launches. The task pulls the SFTP private key from Secrets Manager, hands the key material straight to the SFTP client (`paramiko` or equivalent), and never writes it to disk. When the task exits, the key material is gone with the task memory.

The integration code only handles application logic: pull files, push files, call the API, parse the response. The credential management is between the AWS SDK and the role. IAM enforces the boundary.

## CloudTrail attributes each call to a specific integration

The same CloudTrail field that returned `svc-integrations` for every S3 call now returns the per-integration role. The audit question (which integration accessed `vendor-a-incoming/` at a specific timestamp) has a one-word answer.

```json
{
  "eventName": "GetObject",
  "eventTime": "2026-03-12T03:14:22Z",
  "userIdentity": {
    "type": "AssumedRole",
    "arn": "arn:aws:sts::123456789012:assumed-role/integration-vendor-a-task-role/abcd1234",
    "sessionContext": {
      "sessionIssuer": {
        "arn": "arn:aws:iam::123456789012:role/integration-vendor-a-task-role"
      }
    }
  },
  "resources": [
    {
      "ARN": "arn:aws:s3:::integrations-prod/vendor-a-incoming/batch-20260312.csv"
    }
  ]
}
```

Attribution is a property of the deploy. The role name in `sessionIssuer.arn` resolves directly to the service that ran the call.

Adding a new vendor is a deploy of a new Fargate task or Lambda function with its own role and its own secret. It doesn't touch any other integration's permissions. The blast radius of the new integration is whatever its own role permits. Other integrations are unaffected.

Removing a vendor is symmetric. Delete the service, delete the role, delete the secret. There is no shared credential left holding lingering access. The "did we remember to revoke that" question doesn't come up.

The auditor's question has an answer. The cross-integration bug fails on the first call.

## Smaller systems are cheaper to audit

Per-integration services are the same code, split into separate deploy targets, with an IAM role attached to each. The total complexity is the same as the shared-role version.

Smaller systems are cheaper to audit because they're cheaper to reason about. An auditor reviewing one integration sees one role, one secret, one S3 prefix. The blast radius they have to think about ends at that role's policy. The same is true for an engineer adding a vendor or debugging a permission error.

The `svc-integrations` role was a scope problem. Tightening the policy would not have helped. One identity still served every integration. The fix was to split the scope.

