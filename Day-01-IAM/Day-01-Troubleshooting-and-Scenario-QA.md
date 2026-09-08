# Day 1 — IAM + S3 Basics

# Troubleshooting & Scenario-Based Interview Questions

This document contains detailed troubleshooting and scenario-based interview questions for Day 1.

The purpose is to understand **how to troubleshoot IAM and S3 permission problems step by step**, instead of memorizing answers.

---

# PART 1 — TROUBLESHOOTING QUESTIONS

---

# Q1. An IAM user cannot access S3. How would you troubleshoot it?

## Answer

I would troubleshoot the problem step by step.

I would not immediately say "I will check IAM."

First, I would identify the IAM identity and then determine exactly what operation is failing.

---

## Step 1 — Identify WHO is making the request

First, I identify the IAM identity making the request.

For example:

```text
IAM User
    ↓
day1-user
```

The first question is:

```text
WHO is making the request?
```

The identity could be:

```text
IAM User
IAM Role
Federated Identity
```

For Day 1, our example is an IAM user.

---

## Step 2 — Identify WHAT the user is trying to do

"S3 access" is too general.

I need to know the exact operation.

For example:

```text
Download object → s3:GetObject

Upload object   → s3:PutObject

Delete object   → s3:DeleteObject

List objects    → s3:ListBucket
```

These are separate permissions.

For example:

```text
s3:GetObject
```

does not automatically give:

```text
s3:PutObject
```

---

## Step 3 — Identify WHERE the user is accessing

Next, I identify the exact S3 bucket and object.

For example:

```text
Bucket:
company-data

Object:
reports/file.txt
```

Now I know:

```text
WHO:
day1-user

WHAT:
s3:GetObject

WHERE:
company-data/reports/file.txt
```

---

## Step 4 — Check the Identity-Based Policy

An identity-based policy can be attached to:

```text
IAM User
IAM Group
IAM Role
```

For example:

```text
IAM User
    ↓
Identity Policy
    ↓
Allow s3:GetObject
```

I check whether the required action is allowed.

Example:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::company-data/*"
}
```

This allows the identity to read objects matching that resource.

---

## Step 5 — Check IAM Group Policies

If the user belongs to a group, I check the policies attached to the group.

Example:

```text
IAM User
    ↓
Developers Group
    ↓
IAM Policy
    ↓
S3 Permission
```

I verify:

1. Is the user actually a member of the group?
2. Is the correct policy attached?
3. Does the policy contain the required action?

---

## Step 6 — Check the Resource ARN

Next, I check whether the policy's Resource matches the resource being accessed.

For an S3 bucket:

```text
arn:aws:s3:::company-data
```

For objects inside the bucket:

```text
arn:aws:s3:::company-data/*
```

This distinction is important.

For example:

```text
s3:GetObject
```

normally requires an object resource.

---

## Step 7 — Check the S3 Bucket Policy

S3 can also have a resource-based policy called a bucket policy.

The bucket policy can control access to the bucket and its objects.

Conceptually:

```text
IAM Policy
    ↓
What can this identity do?

Bucket Policy
    ↓
Who can access this bucket?
```

I would check whether the bucket policy contains a restriction or explicit Deny affecting the request.

---

## Step 8 — Check Policy Conditions

A policy can contain a Condition.

For example:

```json
"Condition": {
  "StringEquals": {
    "aws:RequestedRegion": "ap-south-1"
  }
}
```

This means the policy statement has an additional condition.

I need to verify that the request satisfies the condition.

---

## Step 9 — Check Permissions Boundary

If the IAM user or role has a permissions boundary, I check it.

A permissions boundary defines the maximum permissions that the identity can have.

Important:

```text
Permissions Boundary
        ↓
Limits maximum permissions
```

It does not grant permissions by itself.

---

## Step 10 — Check SCP

If the AWS account belongs to AWS Organizations, I check the applicable SCP.

SCP means:

```text
Service Control Policy
```

An SCP acts as an organization-level guardrail.

It can limit what permissions are available to principals in the account.

Important:

```text
SCP
 ↓
Organization-level restriction
```

It does not grant permissions by itself.

---

## Step 11 — Check for Explicit Deny

This is extremely important.

I check whether any applicable policy contains:

```text
Effect: Deny
```

For example:

```text
Identity Policy:
Allow s3:GetObject

Another applicable policy:
Deny s3:GetObject
```

The result can be:

```text
AccessDenied
```

because:

```text
Explicit Deny
       ↓
Overrides
       ↓
Allow
```

---

## Step 12 — Test Again

After finding and fixing the problem, I test the operation again.

For example:

```bash
aws s3api get-object \
  --bucket company-data \
  --key reports/file.txt \
  downloaded.txt
```

---

## Interview Answer

> "First, I would identify the IAM identity making the request and the exact S3 action that is failing. Then I would identify the target resource and verify the Resource ARN. I would check identity-based policies, including direct user policies and group policies, and also check the S3 bucket policy. Then I would check policy conditions, permissions boundaries, and SCPs where applicable. Finally, I would look for an explicit Deny because an explicit Deny overrides an Allow. After fixing the issue, I would test the request again."

---

# Q2. The IAM user can read an S3 object but cannot upload it. Why?

## Answer

This happens because reading and uploading are different S3 actions.

---

## Step 1 — Identify the successful operation

The user can read/download.

The required permission is:

```text
s3:GetObject
```

So:

```text
Read
 ↓
s3:GetObject
```

---

## Step 2 — Identify the failed operation

The user cannot upload.

Uploading requires:

```text
s3:PutObject
```

So:

```text
Upload
 ↓
s3:PutObject
```

---

## Step 3 — Compare the permissions

Suppose the policy contains:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::company-data/*"
}
```

The user has:

```text
GetObject → YES
PutObject → NO
```

Therefore:

```text
Download → Works
Upload   → AccessDenied
```

---

## Step 4 — What would I check?

I would check:

1. `s3:PutObject`
2. Object Resource ARN
3. Bucket policy
4. Conditions
5. Explicit Deny
6. Permissions boundary
7. SCP, if applicable

---

## Interview Answer

> "Reading and uploading require different permissions. Reading an S3 object requires s3:GetObject, while uploading requires s3:PutObject. So the user can read successfully but receive AccessDenied when uploading if s3:PutObject is not allowed."

---

# Q3. The user can upload an object but cannot delete it. What would you check?

## Answer

I would first identify the two operations.

```text
Upload
 ↓
s3:PutObject

Delete
 ↓
s3:DeleteObject
```

---

## Step 1 — Check PutObject

The user can upload, so:

```text
s3:PutObject
```

is already allowed for that request.

---

## Step 2 — Check DeleteObject

The failed operation is delete.

Therefore I check:

```text
s3:DeleteObject
```

---

## Step 3 — Check the object ARN

The DeleteObject permission must apply to the object being deleted.

For example:

```text
arn:aws:s3:::company-data/*
```

---

## Step 4 — Check for restrictions

I would also check:

```text
Bucket policy
Explicit Deny
Permissions boundary
SCP
Conditions
```

---

## Example

If the policy contains:

```text
Allow:
s3:PutObject
```

but does not contain:

```text
Allow:
s3:DeleteObject
```

then:

```text
Upload → Success
Delete → AccessDenied
```

---

## Interview Answer

> "I would check whether s3:DeleteObject is allowed for the correct object ARN. s3:PutObject permission does not automatically provide delete permission. I would also check the bucket policy, conditions, permissions boundary, SCP where applicable, and any explicit Deny."

---

# Q4. The user can access one S3 bucket but not another. Why?

## Answer

Access to one S3 bucket does not automatically mean access to every S3 bucket.

---

## Step 1 — Identify the working bucket

Suppose the user can access:

```text
bucket-a
```

---

## Step 2 — Identify the failed bucket

But access to:

```text
bucket-b
```

fails.

---

## Step 3 — Compare Resource ARNs

The policy may contain:

```text
arn:aws:s3:::bucket-a/*
```

This applies to objects inside bucket A.

It does not automatically include:

```text
arn:aws:s3:::bucket-b/*
```

---

## Step 4 — Check the required action

For example:

```text
s3:GetObject
```

may be allowed for bucket A but not bucket B.

---

## Step 5 — Check other restrictions

I would also check:

```text
Bucket B policy
Explicit Deny
Conditions
Permissions boundary
SCP
```

---

## Interview Answer

> "The IAM policy may be restricted to a specific bucket ARN. For example, if the policy allows access to arn:aws:s3:::bucket-a/*, it does not automatically allow access to bucket B. I would compare the Resource ARNs and then check any resource policy or Deny affecting bucket B."

---

# Q5. The IAM policy contains Allow, but the request still returns AccessDenied. What do you check?

## Answer

This is a very important troubleshooting scenario.

I would not assume that an Allow statement means the request must succeed.

---

## Step 1 — Check the exact action

First:

```text
WHAT action failed?
```

For example:

```text
s3:PutObject
```

---

## Step 2 — Check the Resource

Verify that the policy applies to the correct resource.

For example:

```text
arn:aws:s3:::company-data/*
```

---

## Step 3 — Check Identity-Based Policies

I check:

```text
User policy
Group policy
Role policy
```

as applicable.

---

## Step 4 — Check Resource-Based Policy

For S3, I check the bucket policy.

It may contain:

```text
Deny
```

or another restriction.

---

## Step 5 — Check Conditions

The Allow statement may have a condition.

For example:

```text
aws:RequestedRegion
aws:SourceIp
aws:SecureTransport
```

If the condition is not satisfied, that Allow statement may not apply.

---

## Step 6 — Check Permissions Boundary

A permissions boundary can limit the maximum permissions of an IAM user or role.

---

## Step 7 — Check SCP

If the account is part of AWS Organizations, I check the applicable SCP.

---

## Step 8 — Check Explicit Deny

This is the most important point.

```text
Allow
 +
Explicit Deny
 ↓
DENIED
```

An explicit Deny overrides an applicable Allow.

---

## Interview Answer

> "If an Allow exists but I still receive AccessDenied, I would check the exact action and Resource ARN first. Then I would check all applicable identity-based and resource-based policies, conditions, permissions boundary, and SCP where applicable. Most importantly, I would check for an explicit Deny because an explicit Deny overrides an Allow."

---

# Q6. A user was added to an IAM group but still cannot access S3. What do you check?

## Answer

I would troubleshoot the group configuration step by step.

---

## Step 1 — Verify group membership

Check whether:

```text
User
 ↓
Actually belongs to group
```

---

## Step 2 — Check the group policy

The group may have a policy attached.

For example:

```text
Developers Group
       ↓
S3 Read Policy
```

I verify that the correct policy is attached.

---

## Step 3 — Check the required action

If the user is trying to download:

```text
s3:GetObject
```

If uploading:

```text
s3:PutObject
```

If deleting:

```text
s3:DeleteObject
```

---

## Step 4 — Check Resource ARN

Verify the policy points to the correct bucket/object.

---

## Step 5 — Check resource policy

For S3, check the bucket policy.

---

## Step 6 — Check restrictions

Check:

```text
Conditions
Permissions boundary
SCP
Explicit Deny
```

---

## Interview Answer

> "I would first verify that the user is actually a member of the group. Then I would check whether the correct policy is attached to the group and whether it contains the required S3 action and correct Resource ARN. I would also check the bucket policy, conditions, permissions boundary, SCP where applicable, and any explicit Deny."

---

# Q7. S3 upload returns AccessDenied. What is your troubleshooting approach?

## Answer

Because the operation is upload, the first permission I check is:

```text
s3:PutObject
```

---

## Step 1 — WHO?

Identify the caller.

```text
IAM User?
IAM Role?
```

---

## Step 2 — WHAT?

The operation is:

```text
Upload
```

Therefore:

```text
s3:PutObject
```

---

## Step 3 — WHERE?

Identify:

```text
Bucket
Object
```

For example:

```text
arn:aws:s3:::company-data/*
```

---

## Step 4 — Check identity policy

Does the applicable policy allow:

```text
s3:PutObject
```

for that object?

---

## Step 5 — Check bucket policy

The bucket policy could restrict the request.

---

## Step 6 — Check conditions

For example:

```text
Source IP
Secure Transport
Requested Region
```

depending on the policy.

---

## Step 7 — Check permissions boundary

Does the permissions boundary allow the required permission within its maximum?

---

## Step 8 — Check SCP

If applicable, verify that the organization-level guardrails permit the action.

---

## Step 9 — Check explicit Deny

Look for:

```text
Effect: Deny
```

---

## Step 10 — Test again

After fixing the issue, retry the upload.

---

## Interview Answer

> "For an S3 upload AccessDenied, I would identify the IAM identity, confirm that the failed action is s3:PutObject, verify the bucket and object ARN, and check identity-based and resource-based policies. I would then check conditions, permissions boundaries, SCPs where applicable, and explicit Deny. Finally, I would test the upload again."

---

# Q8. S3 download returns AccessDenied. What would you check?

## Answer

The first permission I check is:

```text
s3:GetObject
```

because downloading an object requires object-read permission.

---

## Step 1 — Identify the caller

```text
WHO?
```

For example:

```text
IAM User
```

---

## Step 2 — Identify the operation

```text
WHAT?
```

The operation is:

```text
Download
```

Required action:

```text
s3:GetObject
```

---

## Step 3 — Identify the resource

```text
WHERE?
```

For example:

```text
Bucket:
company-data

Object:
reports/file.txt
```

---

## Step 4 — Check the object ARN

Verify that the policy Resource matches the object.

Example:

```text
arn:aws:s3:::company-data/*
```

---

## Step 5 — Check policies

Check:

```text
Identity policy
Group policy
Bucket policy
```

as applicable.

---

## Step 6 — Check restrictions

Check:

```text
Conditions
Permissions boundary
SCP
Explicit Deny
```

---

## Interview Answer

> "For an S3 download AccessDenied, I would first verify s3:GetObject and confirm that the Resource ARN matches the object. Then I would check applicable identity-based and resource-based policies, conditions, permissions boundaries, SCPs where applicable, and explicit Deny."

---

# Q9. S3 delete returns AccessDenied. What would you check?

## Answer

The first permission I check is:

```text
s3:DeleteObject
```

---

## Step 1

Identify the IAM identity.

```text
WHO?
```

---

## Step 2

Identify the failed action.

```text
Delete
 ↓
s3:DeleteObject
```

---

## Step 3

Identify the object.

```text
WHERE?
```

Check that the Resource ARN applies to the object.

---

## Step 4

Check identity policies.

Look for:

```text
Allow s3:DeleteObject
```

---

## Step 5

Check bucket policy.

Look for restrictions or Deny statements.

---

## Step 6

Check permissions boundary.

A boundary can limit the maximum permissions of the IAM user or role.

---

## Step 7

Check SCP.

If applicable, check organization-level restrictions.

---

## Step 8

Check explicit Deny.

For example:

```text
Allow → s3:DeleteObject

Deny → s3:DeleteObject
```

Result:

```text
AccessDenied
```

---

## Interview Answer

> "I would first check whether s3:DeleteObject is allowed for the correct object ARN. Then I would check the applicable identity and bucket policies, conditions, permissions boundary, SCP where applicable, and any explicit Deny."

---

# Q10. IAM permissions worked yesterday but stopped working today. What would you investigate?

## Answer

If something worked yesterday but not today, I would think:

```text
WHAT CHANGED?
```

---

## Step 1 — Check IAM policy changes

Maybe someone modified:

```text
IAM policy
```

or removed a permission.

---

## Step 2 — Check group membership

Maybe the user was removed from a group.

---

## Step 3 — Check S3 bucket policy

Maybe the bucket policy changed.

---

## Step 4 — Check explicit Deny

A new Deny may have been introduced.

---

## Step 5 — Check Permissions Boundary

Maybe the boundary changed.

---

## Step 6 — Check SCP

Maybe an organization-level restriction was introduced.

---

## Step 7 — Check Conditions

Maybe a policy condition changed.

---

## Step 8 — Use CloudTrail

CloudTrail can help investigate AWS API activity and relevant changes.

For example, I could investigate:

```text
Who made the change?
When was it changed?
What AWS API operation occurred?
```

---

## Interview Answer

> "If permissions worked yesterday but stopped today, I would investigate what changed. I would check recent IAM policy changes, group membership, bucket policy changes, explicit Deny, permissions boundaries, SCPs, and conditions. I would also use CloudTrail to investigate relevant API activity and configuration changes."

---

# PART 2 — SCENARIO-BASED QUESTIONS

---

# Scenario 1. Application only needs to download files from S3. What permissions would you give?

## Requirement

The application needs:

```text
Download → YES
Upload   → NO
Delete   → NO
```

---

## Step 1 — Identify required action

Downloading an object requires:

```text
s3:GetObject
```

---

## Step 2 — Grant only that permission

Example:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::company-data/*"
}
```

---

## Step 3 — Do not grant unnecessary permissions

Do not automatically grant:

```text
s3:PutObject
s3:DeleteObject
```

---

## Step 4 — What if the application needs to list files?

Then it may also require:

```text
s3:ListBucket
```

Important:

```text
GetObject
 ↓
Read an object

ListBucket
 ↓
List objects in a bucket
```

These are different permissions.

---

## Interview Answer

> "I would grant s3:GetObject for downloading objects and restrict the Resource to the required bucket or objects. If the application also needs to list objects, I would add s3:ListBucket as required. I would not grant upload or delete permissions unless they are needed."

---

# Scenario 2. Application needs to upload files but never delete them.

## Requirement

```text
Upload → YES
Delete → NO
```

---

## Step 1

Upload requires:

```text
s3:PutObject
```

---

## Step 2

Do not grant:

```text
s3:DeleteObject
```

---

## Step 3

Restrict the Resource.

For example:

```text
arn:aws:s3:::company-data/uploads/*
```

if the application's required scope is limited to that prefix.

---

## Step 4 — Test

Test:

```text
Upload → Should work
Delete → Should fail
```

This proves least privilege is working.

---

## Interview Answer

> "I would grant s3:PutObject only for the required S3 resource or prefix and would not grant s3:DeleteObject. I would test both upload and delete to confirm the application has only the required permission."

---

# Scenario 3. Developer asks for AmazonS3FullAccess because the application gets AccessDenied. What would you do?

## Answer

I would not immediately grant:

```text
AmazonS3FullAccess
```

---

## Step 1 — Identify the failed operation

For example:

```text
Application
 ↓
Upload failed
```

---

## Step 2 — Identify required permission

Upload requires:

```text
s3:PutObject
```

---

## Step 3 — Identify the required resource

Maybe the application only needs:

```text
company-data/uploads/*
```

---

## Step 4 — Create least-privilege permission

Instead of full S3 access, grant the required action for the required resource.

---

## Step 5 — Test

Test the application again.

---

## Why?

Because:

```text
AccessDenied
       ↓
Does NOT automatically mean
       ↓
Give full access
```

Instead:

```text
AccessDenied
       ↓
Find failed action
       ↓
Find required resource
       ↓
Grant minimum permission
       ↓
Test
```

---

## Interview Answer

> "I would not immediately grant AmazonS3FullAccess. I would identify the exact operation that failed, determine the required action and resource, and grant only the minimum permission required. This follows least privilege and reduces security risk."

---

# Scenario 4. Your company has 100 S3 buckets, but an application should access only one. How would you design the policy?

## Requirement

There are:

```text
100 S3 buckets
```

But the application needs:

```text
Only bucket-A
```

---

## Step 1 — Identify required bucket

```text
bucket-A
```

---

## Step 2 — Identify required actions

For example:

```text
s3:GetObject
s3:PutObject
```

---

## Step 3 — Restrict Resource

For object access:

```text
arn:aws:s3:::bucket-A/*
```

---

## Step 4 — Do not grant all buckets

Avoid:

```text
Resource: "*"
```

when a specific resource can be used.

---

## Interview Answer

> "I would restrict the policy Resource to the specific bucket required by the application and grant only the required S3 actions. I would not provide access to all 100 buckets because that violates least privilege."

---

# Scenario 5. Developers should upload files but must never delete them.

## Requirement

```text
Upload → YES
Delete → NO
```

---

## Step 1

Grant:

```text
s3:PutObject
```

---

## Step 2

Do not grant:

```text
s3:DeleteObject
```

---

## Step 3

Test both operations.

```text
Upload
 ↓
Expected: Success

Delete
 ↓
Expected: AccessDenied
```

---

## Step 4

If delete still works unexpectedly, investigate:

```text
Other applicable policies
Resource policy
Permissions boundary
SCP
Explicit Deny/Allow interactions
```

This is important because permissions can come from multiple applicable policy mechanisms.

---

## Interview Answer

> "I would allow s3:PutObject but not s3:DeleteObject for the required resource. Then I would test both operations to verify that upload succeeds and delete is denied."

---

# Scenario 6. An employee says, "I have S3 access." What does that actually mean?

## Answer

"S3 access" is too general.

I need to ask:

```text
Which action?
Which bucket?
Which object?
Which conditions?
```

---

## Example

A user may have:

```text
s3:GetObject
```

but not:

```text
s3:PutObject
s3:DeleteObject
```

Therefore:

```text
Read → YES
Upload → NO
Delete → NO
```

---

## Interview Answer

> "'S3 access' is not specific enough. I would determine which actions, buckets, objects, and conditions the user has access to. For example, a user may have read access without having upload or delete access."

---

# Scenario 7. A user can upload to bucket A but cannot upload to bucket B.

## Step 1 — Identify the action

Both operations are uploads:

```text
s3:PutObject
```

---

## Step 2 — Compare resources

Maybe the policy allows:

```text
arn:aws:s3:::bucket-a/*
```

but does not allow:

```text
arn:aws:s3:::bucket-b/*
```

---

## Step 3 — Check bucket B policy

Bucket B may also have a resource-based policy or restriction.

---

## Step 4 — Check other restrictions

Check:

```text
Conditions
Permissions boundary
SCP
Explicit Deny
```

---

## Result

```text
Bucket A
 ↓
PutObject → Allowed

Bucket B
 ↓
PutObject → AccessDenied
```

---

## Interview Answer

> "I would compare the Resource ARNs in the policy. The user may have s3:PutObject permission for bucket A but not bucket B. I would also check bucket B's resource policy, conditions, permissions boundary, SCP where applicable, and explicit Deny."

---

# Scenario 8. Security asks you to follow least privilege. What would you do?

## Answer

Least privilege means:

```text
Give only the permissions actually required.
```

---

## Step 1 — Understand the application

I would ask:

```text
What does the application need to do?
```

---

## Step 2 — Identify actions

For example:

```text
Read
Upload
```

---

## Step 3 — Identify resources

For example:

```text
Only company-data bucket
```

---

## Step 4 — Build the smallest permission set

For example:

```text
s3:GetObject
s3:PutObject
```

for the required resource.

---

## Step 5 — Do not give unnecessary permissions

Avoid:

```text
AdministratorAccess
AmazonS3FullAccess
```

when they are not required.

---

## Step 6 — Test

Confirm:

```text
Required operations → Work
Unrequired operations → Do not work
```

---

## Interview Answer

> "I would identify the exact actions and resources required by the application and grant only those permissions. I would avoid broad policies such as AdministratorAccess or AmazonS3FullAccess unless there is a genuine requirement. I would then test the required and restricted operations."

---

# PART 3 — IAM POLICY TYPES YOU SHOULD KNOW

When troubleshooting IAM, you should understand these policy concepts.

---

# 1. Identity-Based Policy

Attached to:

```text
IAM User
IAM Group
IAM Role
```

It answers:

```text
What can this identity do?
```

Example:

```text
User
 ↓
Identity Policy
 ↓
Allow s3:GetObject
```

---

# 2. Resource-Based Policy

Attached to a resource.

For S3, this is commonly:

```text
S3 Bucket Policy
```

It answers:

```text
Who can access this resource?
```

Example:

```text
S3 Bucket
 ↓
Bucket Policy
 ↓
Principal
 ↓
Allow s3:GetObject
```

---

# 3. Permissions Boundary

Attached to:

```text
IAM User
IAM Role
```

It defines the maximum permissions that identity can have.

Important:

```text
Permissions Boundary
       ↓
Maximum permissions
```

It does not grant permissions by itself.

---

# 4. Service Control Policy — SCP

Used through:

```text
AWS Organizations
```

It acts as an organization-level guardrail.

Example:

```text
Organization
     ↓
OU
     ↓
AWS Account
     ↓
SCP
     ↓
IAM User/Role
```

Important:

```text
SCP
 ↓
Limits available permissions
```

It does not grant permissions by itself.

---

# 5. Policy Condition

A Condition adds an additional requirement to a policy statement.

Example:

```json
"Condition": {
  "StringEquals": {
    "aws:RequestedRegion": "ap-south-1"
  }
}
```

Think:

```text
Allow
 +
Condition must be satisfied
```

---

# 6. Principal

Principal means:

```text
Who is specified in a resource-based policy?
```

Example:

```json
"Principal": {
  "AWS": "arn:aws:iam::123456789012:role/MyRole"
}
```

---

# 7. Resource

Resource tells AWS:

```text
Which AWS resource?
```

Example:

```text
arn:aws:s3:::company-data/*
```

---

# 8. Action

Action tells AWS:

```text
What operation?
```

Examples:

```text
s3:GetObject
s3:PutObject
s3:DeleteObject
s3:ListBucket
```

---

# PART 4 — IDENTITY POLICY VS RESOURCE POLICY

This is very important for interviews.

## Identity-Based Policy

Think:

```text
IDENTITY
   ↓
What can I do?
```

Example:

```text
IAM User
   ↓
Allow s3:GetObject
   ↓
company-data
```

---

## Resource-Based Policy

Think:

```text
RESOURCE
   ↓
Who can access me?
```

Example:

```text
S3 Bucket
   ↓
Bucket Policy
   ↓
Principal: IAM Role
   ↓
Allow GetObject
```

---

# PART 5 — PERMISSIONS BOUNDARY VS SCP

Another very common interview question.

## Permissions Boundary

```text
Individual IAM User/Role
        ↓
Maximum permissions
```

## SCP

```text
Organization / OU / Account
        ↓
Organization-level guardrail
```

### Easy memory trick

```text
PB
 ↓
Person/Principal boundary

SCP
 ↓
Organization boundary
```

---

# PART 6 — CONDITION EXAMPLE

Suppose we have:

```text
IAM User
 ↓
Allow s3:GetObject
```

But we add:

```text
Condition:
aws:RequestedRegion = ap-south-1
```

Conceptually:

```text
Request
   ↓
GetObject?
   ↓
YES
   ↓
Condition satisfied?
   ↓
YES → Statement applies
NO  → Statement does not apply
```

Therefore Conditions provide more precise control over when a policy statement applies.

---

# PART 7 — RESOURCE ARN EXAMPLE

For S3:

### Bucket ARN

```text
arn:aws:s3:::company-data
```

### Object ARN

```text
arn:aws:s3:::company-data/*
```

Remember:

```text
Bucket
 ↓
arn:aws:s3:::company-data

Objects
 ↓
arn:aws:s3:::company-data/*
```

For example:

```text
s3:ListBucket
```

is a bucket-level action.

While:

```text
s3:GetObject
s3:PutObject
s3:DeleteObject
```

are object-level actions.

---

# PART 8 — EXPLICIT DENY

An explicit Deny looks like:

```json
{
  "Effect": "Deny",
  "Action": "s3:DeleteObject",
  "Resource": "arn:aws:s3:::company-data/*"
}
```

If another applicable policy says:

```text
Allow s3:DeleteObject
```

the explicit Deny wins.

Remember:

```text
EXPLICIT DENY
      ↓
OVERRIDES
      ↓
ALLOW
```

---

# PART 9 — COMPLETE ACCESSDENIED TROUBLESHOOTING FORMULA

Whenever I see:

```text
AccessDenied
```

I use this process:

```text
STEP 1
WHO?
↓
Which IAM identity?

STEP 2
WHAT?
↓
Which action failed?

STEP 3
WHERE?
↓
Which resource?

STEP 4
IDENTITY POLICY
↓
Is the action allowed?

STEP 5
RESOURCE POLICY
↓
Does the resource policy allow/restrict it?

STEP 6
RESOURCE ARN
↓
Does the ARN match?

STEP 7
CONDITIONS
↓
Are the conditions satisfied?

STEP 8
PERMISSIONS BOUNDARY
↓
Is the permission within the boundary?

STEP 9
SCP
↓
Is the action permitted by organization guardrails?

STEP 10
EXPLICIT DENY
↓
Is there an applicable Deny?

STEP 11
TEST AGAIN
↓
Did the operation succeed?
```

---

# PART 10 — REAL TROUBLESHOOTING MINDSET

Never think:

```text
AccessDenied
 ↓
Give AdministratorAccess
```

Instead think:

```text
AccessDenied
      ↓
WHO?
      ↓
WHAT?
      ↓
WHERE?
      ↓
POLICY?
      ↓
RESOURCE?
      ↓
CONDITION?
      ↓
BOUNDARY?
      ↓
SCP?
      ↓
EXPLICIT DENY?
      ↓
FIX
      ↓
TEST AGAIN
```

This demonstrates real troubleshooting ability.

---

# PART 11 — MOST IMPORTANT INTERVIEW QUESTIONS

## Q1. What is IAM?

> "IAM stands for Identity and Access Management. It allows me to control who can access AWS resources and what actions they can perform."

---

## Q2. What is an IAM user?

> "An IAM user is an AWS identity that can represent a person or application and can have permissions assigned to it."

---

## Q3. What is an IAM group?

> "An IAM group is a collection of IAM users. Policies can be attached to the group so that its users can receive those permissions."

---

## Q4. What is an IAM policy?

> "An IAM policy is a JSON document that defines permissions using elements such as Effect, Action, Resource, and optionally Condition."

---

## Q5. What is an IAM role?

> "An IAM role is an identity that trusted entities can assume to obtain permissions, typically using temporary credentials."

---

## Q6. What is least privilege?

> "Least privilege means granting only the permissions required to perform a specific task and avoiding unnecessary permissions."

---

## Q7. What is an identity-based policy?

> "An identity-based policy is attached to an IAM user, group, or role and defines what actions that identity can perform on resources."

---

## Q8. What is a resource-based policy?

> "A resource-based policy is attached to a resource, such as an S3 bucket, and defines which principals can access that resource and what actions they can perform."

---

## Q9. What is a permissions boundary?

> "A permissions boundary defines the maximum permissions an IAM user or role can have. It does not grant permissions by itself."

---

## Q10. What is an SCP?

> "An SCP, or Service Control Policy, is an AWS Organizations policy that acts as an organization-level guardrail and limits the maximum permissions available to principals in member accounts. It does not grant permissions by itself."

---

## Q11. What is a Condition?

> "A Condition adds additional requirements that must be satisfied for a policy statement to apply. For example, a policy can use conditions based on requested region, source IP, or secure transport."

---

## Q12. What is a Principal?

> "Principal identifies who is specified in a resource-based policy, such as an IAM user, role, AWS account, or service principal."

---

## Q13. What is an ARN?

> "ARN stands for Amazon Resource Name. It uniquely identifies an AWS resource."

---

## Q14. What is explicit Deny?

> "An explicit Deny is a policy statement with Effect set to Deny. An explicit Deny overrides an applicable Allow."

---

# PART 12 — KEY S3 PERMISSIONS

| Permission        | Meaning                  |
| ----------------- | ------------------------ |
| `s3:GetObject`    | Read/download an object  |
| `s3:PutObject`    | Upload an object         |
| `s3:DeleteObject` | Delete an object         |
| `s3:ListBucket`   | List objects in a bucket |

Remember:

```text
GetObject
 ↓
Read

PutObject
 ↓
Upload

DeleteObject
 ↓
Delete

ListBucket
 ↓
List
```

---

# PART 13 — FINAL INTERVIEW ANSWER

## Interviewer:

"How do you troubleshoot an AWS AccessDenied error?"

## Strong Answer:

> "First, I identify the IAM identity making the request. Then I identify the exact action that failed and the AWS resource being accessed. I verify the Resource ARN and check the applicable identity-based and resource-based policies. I also check policy conditions, permissions boundaries, and SCPs where applicable. Most importantly, I check for an explicit Deny because an explicit Deny overrides an Allow. After identifying the restriction, I fix the permission according to least privilege and test the request again."

---

# PART 14 — DAY 1 GOLDEN MEMORY FORMULA

Memorize this:

```text
WHO
 ↓
WHAT
 ↓
WHERE
 ↓
IDENTITY POLICY
 ↓
RESOURCE POLICY
 ↓
RESOURCE ARN
 ↓
CONDITION
 ↓
PERMISSIONS BOUNDARY
 ↓
SCP
 ↓
EXPLICIT DENY
 ↓
TEST AGAIN
```

And always remember:

```text
EXPLICIT DENY
      ↓
OVERRIDES
      ↓
ALLOW
```

And:

```text
LEAST PRIVILEGE
      ↓
ONLY GIVE WHAT IS REQUIRED
```

---

# DAY 1 FINAL TAKEAWAY

IAM is about controlling:

```text
WHO
 ↓
CAN PERFORM
 ↓
WHAT ACTION
 ↓
ON WHICH RESOURCE
 ↓
UNDER WHICH CONDITIONS
```

For S3 troubleshooting:

```text
WHO → ACTION → RESOURCE → POLICY → CONDITION → RESTRICTIONS → DENY → TEST
```

The goal of an AWS Cloud/DevOps Engineer is not to solve AccessDenied by giving broad permissions.

The goal is to:

```text
Identify the problem
       ↓
Understand the required permission
       ↓
Find the restriction
       ↓
Apply least privilege
       ↓
Test
       ↓
Verify
```

That is the correct IAM troubleshooting mindset.

