---
layout: default
title: "Each integration gets its own role."
date: 2026-05-26
category: Software Engineering
draft: true
---

<!--
OUTLINE (remove before publishing)

1. The shared `svc-integrations` user worked until the auditor asked a question.
   - The fleet at StartupTechCo: two clinical vendors dropping files over SFTP, a payments vendor pulled over SSH, half a dozen HTTPS API integrations.
   - The credential setup: one IAM user (`svc-integrations`) with read/write to several S3 prefixes; SFTP private keys in one shared Secrets Manager entry; vendor API keys in separate entries, all readable by the same user.
   - The audit moment: "show me which integration accessed `vendor-a-incoming/` at 03:14 UTC." CloudTrail showed `svc-integrations` on every call. No answer.
   - The corroborating bug: a refactor in the vendor-B integration accidentally listed `vendor-a-incoming/`. The shared user had access, so AWS allowed the call. Discovered in code review.

2. The blast radius of a shared credential is the whole fleet.
   - Least privilege can't be applied meaningfully to a shared identity: every integration runs under the same policy, so scoping the policy down breaks the integrations that legitimately need that access.
   - The auditor's question and the bug are the same property in different costumes: shared identity collapses both attribution and isolation.

3. Per-integration isolation means one IAM role per compute unit.
   - The wrong fix: keep the shared user, add a giant policy with `StringEquals` conditions to scope by integration name. Adds complexity without changing the blast radius.
   - The right fix: each integration is its own deploy target — a Lambda function for the HTTPS API integrations, a Fargate task for the SFTP/SSH file transfers. Each gets its own execution/task role.
   - The role's policy mentions only the resources that integration touches: its S3 prefix, its one Secrets Manager secret ARN, its KMS key. No wildcards across vendors.
   - Show a short policy snippet for `integration-vendor-a` Fargate task role — scoped to `s3://integrations-prod/vendor-a/*` and the one secret ARN.

4. The compute boundary is the credential boundary.
   - Lambda for API integrations: execution role attached; vendor API key fetched from Secrets Manager at cold start; no secret material on disk, no env vars at deploy time.
   - Fargate for SFTP/SSH transfers: task role attached; SFTP private key fetched into ephemeral task memory; key never written to a mounted volume, never baked into the image.
   - The SDK handles AWS auth via the role. The integration code only deals with application logic. IAM enforces the boundary.

5. CloudTrail attributes each call to a specific integration.
   - CloudTrail now answers "who accessed vendor-A's prefix at 03:14 UTC" with `integration-vendor-a-task-role`. Attribution is a property of the deploy.
   - Adding a new vendor is a deploy of a new service plus a new role. It does not touch any other integration's permissions.
   - Removing a vendor is symmetric: delete the service, delete the role, the secret is unreachable. No "did we remember to revoke that" lingering risk.
   - The auditor's question has an answer. The cross-integration bug fails on the first call.

6. Smaller systems are cheaper to audit.
   - Per-integration services aren't more complex than a shared integrations service. They're the same complexity, partitioned differently.
   - Smaller systems are cheaper to audit because they're cheaper to reason about.
   - Closing argument back to the opening: the `svc-integrations` user was a scope problem. The fix was to split the scope.
-->

## The shared `svc-integrations` user worked until the auditor asked a question.

StartupTechCo runs a fleet of vendor integrations: two flat-file vendors it pulls from over SFTP, a third that pushes files into an SFTP endpoint it hosts, a payments vendor it pulls from over SSH, and half a dozen HTTPS API integrations. All of it runs on a couple of Fargate workers. All of it authenticates as a single IAM user, `svc-integrations`. The user's policy covers read/write on every integration's S3 prefix and `secretsmanager:GetSecretValue` on the secrets holding the SFTP keys and API keys.

The setup had two problems.

The first was attribution. An auditor asked which integration accessed `vendor-a-incoming/` at a specific timestamp. CloudTrail showed `svc-integrations` for every S3 call. There was no way to map the call to a specific integration.

The second was blast radius. A refactor in the vendor-B integration introduced a paginated `ListObjectsV2` call with the wrong prefix variable: it listed `vendor-a-incoming/` instead of `vendor-b-incoming/`. The change deployed. It ran on every vendor-B cron tick for two weeks. Vendor-B's downstream processor filtered by filename pattern and silently skipped everything that didn't match, so nothing broke. The call returned normally each time, because `svc-integrations` had access to both prefixes. A per-integration role would have failed at the first execution with `AccessDenied`.

## The blast radius of a shared credential is the whole fleet.

Least privilege doesn't get you isolation when every integration uses the same identity. The vendor-B bug listed vendor-A's prefix because the IAM user the workload ran as had access to that prefix. Tightening the policy doesn't help: every integration runs under the same policy, so the only way to deny vendor-B access to vendor-A's prefix is to deny everyone, which breaks vendor-A's own integration.

The auditor's question and the cross-integration bug are the same property in different shapes. Shared identity collapses attribution and isolation into one credential. There is no policy you can attach to a single IAM user that recovers either.

## Per-integration isolation means one IAM role per compute unit.

The shortcut is to keep the shared user and scope per-integration access through `StringEquals` conditions on a tag or username. This adds complexity to the policy without changing the blast radius. The fix is to give each integration its own deploy target with its own role.

At StartupTechCo this meant splitting `svc-integrations` into one role per integration, attached to the compute that runs that integration. Each integration is its own Fargate task or Lambda function. Each task or function has its own IAM role. The role's policy mentions only the resources the integration touches: its S3 prefix, the one Secrets Manager secret holding its credential, and the KMS key that encrypts that secret.

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

Vendor-A's Fargate task pulls files from the vendor's SFTP endpoint, writes them to `s3://integrations-prod/vendor-a-incoming/`, and reads the SFTP private key from one named secret. Vendor-B's task role doesn't reference `vendor-a-incoming/` anywhere. The accidental `ListObjectsV2` against that prefix returns `AccessDenied` on the first execution.

## The compute boundary is the credential boundary.

Each integration deals with two credentials. The first is the AWS identity the workload runs as: the Fargate task role or Lambda execution role. The AWS SDK handles this automatically; the integration code never sees an AWS access key. The second is the vendor credential: the SFTP private key, the API key. The integration code uses the first to fetch the second from Secrets Manager at runtime.

For the Lambda API integrations, the execution role is attached at deploy time. The function calls `GetSecretValue` at start to pull the vendor API key, holds it in memory for the lifetime of the container, and passes it into the outbound HTTP client (`requests`, `httpx`, or equivalent).

For the Fargate SFTP tasks, the task role is attached when the task launches. The task pulls the SFTP private key from Secrets Manager, hands the key material straight to the SFTP client (`paramiko` or equivalent), and never writes it to disk. When the task exits, the key material is gone with the task memory.

The integration code only handles application logic: pull files, push files, call the API, parse the response. The credential management is between the AWS SDK and the role. IAM enforces the boundary.

## CloudTrail attributes each call to a specific integration.

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

Removing a vendor is symmetric. Delete the service, delete the role, the secret is unreachable. There is no shared credential left holding lingering access. The "did we remember to revoke that" question doesn't come up.

The auditor's question has an answer. The cross-integration bug fails on the first call.

## Smaller systems are cheaper to audit.

Per-integration services are the same code, split into separate deploy targets, with an IAM role attached to each. The total complexity is the same as the shared-user version.

Smaller systems are cheaper to audit because they're cheaper to reason about. An auditor reviewing one integration sees one role, one secret, one S3 prefix. The blast radius they have to think about ends at that role's policy. The same is true for an engineer adding a vendor or debugging a permission error.

The `svc-integrations` user was a scope problem. Tightening the policy would not have helped. One identity still served every integration. The fix was to split the scope.

