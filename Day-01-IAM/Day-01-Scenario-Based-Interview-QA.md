Absolutely ❤️ Here is the **copy-paste-ready version**, separated into **3 Markdown files** exactly according to your Day 1, Day 2, and Day 3 labs.

### 📄 File 1 — `Day-01-Troubleshooting-and-Scenario-QA.md`

# Day 1 — IAM User, Group, Policy & S3

## Troubleshooting and Scenario-Based Interview Q&A

# 🔧 PART 1 — TROUBLESHOOTING INTERVIEW Q&A

### 1. IAM user cannot access S3. What do you check?

**Answer:**

I would check:

1. Which IAM user is making the request.
2. Policies attached to the user.
3. Policies inherited through the IAM group.
4. Required S3 action.
5. Resource ARN.
6. S3 bucket policy.
7. Explicit Deny.
8. Permissions boundary or SCP if applicable.

Then I would test the permission again.

---

### 2. User can read S3 objects but cannot upload.

**Answer:**

I would check whether `s3:PutObject` is allowed.

Reading requires:

```text
s3:GetObject
```

Uploading requires:

```text
s3:PutObject
```

So read access does not automatically mean upload access.

---

### 3. User can upload but cannot delete.

**Answer:**

I would check:

```text
s3:DeleteObject
```

If `DeleteObject` is not allowed, deletion will return `AccessDenied`.

---

### 4. User can access one bucket but not another.

**Answer:**

I would compare the resource permissions.

The policy may allow access only to a specific bucket ARN, for example:

```text
arn:aws:s3:::bucket-A/*
```

It would not automatically grant access to bucket B.

---

### 5. IAM policy says Allow, but access is denied. Why?

**Answer:**

I would check for an explicit Deny.

In AWS IAM evaluation, an explicit Deny overrides an Allow.

I would also check bucket policies, permissions boundaries, SCPs, and other applicable policy controls.

---

### 6. User says they were added to an IAM group but still cannot access S3.

**Answer:**

I would verify:

* User is actually a member of the group.
* Correct policy is attached to the group.
* Policy contains the required action.
* Resource ARN is correct.
* There is no explicit Deny.

Then I would test again.

---

### 7. IAM user suddenly loses access to S3.

**Answer:**

I would investigate what changed.

I would check:

* User policies.
* Group membership.
* Group policies.
* Bucket policy.
* Explicit Deny.
* Permissions boundary.
* SCP, if applicable.
* Recent IAM changes.

CloudTrail can also help identify who changed IAM permissions.

---

### 8. S3 upload gives AccessDenied. What is your troubleshooting process?

**Answer:**

I would identify:

```text
WHO → IAM identity
WHAT → s3:PutObject
WHERE → S3 object ARN
POLICY → Allow/Deny
```

Then I would check whether the user has `s3:PutObject` permission for the correct bucket/object.

---

### 9. S3 download gives AccessDenied.

**Answer:**

I would check:

```text
s3:GetObject
```

and verify that the resource ARN points to the object rather than only the bucket.

---

### 10. S3 delete gives AccessDenied.

**Answer:**

I would check:

```text
s3:DeleteObject
```

and look for an explicit Deny.

# 🎯 PART 2 — SCENARIO-BASED INTERVIEW Q&A

## Scenario 1 — IAM User Needs S3 Access

**Interviewer:**

> An IAM user needs access to S3. How would you give access?

**Answer:**

I would first identify exactly what the user needs to do, such as read, upload, or delete.

Then I would attach the minimum required permissions through an IAM policy or IAM group.

I would avoid giving unnecessary permissions.

---

## Scenario 2 — Read-Only User

**Interviewer:**

> A user only needs to download S3 files. What permissions would you give?

**Answer:**

I would grant only the required read permissions, primarily:

```text
s3:GetObject
```

and `s3:ListBucket` only if the user needs to list objects.

I would not grant upload or delete permissions.

---

## Scenario 3 — Upload Only

**Interviewer:**

> A user needs to upload files but never download or delete them.

**Answer:**

I would grant:

```text
s3:PutObject
```

for the required bucket/object path.

I would not grant:

```text
s3:GetObject
s3:DeleteObject
```

unless required.

---

## Scenario 4 — Upload + Download

**Interviewer:**

> A user needs to upload and download files but must never delete them.

**Answer:**

I would grant:

```text
s3:PutObject
s3:GetObject
```

but not:

```text
s3:DeleteObject
```

This follows least privilege.

---

## Scenario 5 — Developer Requests Full S3 Access

**Interviewer:**

> A developer asks for full S3 access because their application gets AccessDenied. What do you do?

**Answer:**

I would not immediately grant full access.

First, I would identify the exact API operation that failed and grant the minimum required permission.

For example, if upload failed, I would investigate:

```text
s3:PutObject
```

---

## Scenario 6 — One Application, One Bucket

**Interviewer:**

> Your company has 100 S3 buckets, but a user should access only one. How would you design it?

**Answer:**

I would restrict the IAM policy's `Resource` to the required bucket and its objects.

I would not grant access to all S3 buckets.

---

## Scenario 7 — Delete Protection

**Interviewer:**

> Users should be able to upload application files but never delete them.

**Answer:**

I would grant:

```text
s3:PutObject
```

and omit:

```text
s3:DeleteObject
```

I would test the upload and delete operations separately.

---

## Scenario 8 — `aws s3 ls`

**Interviewer:**

> A user can access a bucket, but `aws s3 ls` returns AccessDenied. Is the IAM configuration necessarily broken?

**Answer:**

No.

`aws s3 ls` without a bucket requires permission to list buckets.

I would test:

```bash
aws s3 ls s3://my-bucket
```

If that works, the user can access the specific bucket even though they cannot list all buckets.

---

## Scenario 9 — AccessDenied After Policy Change

**Interviewer:**

> You changed an IAM policy but the user still gets AccessDenied. What do you check?

**Answer:**

I would verify:

1. Correct user.
2. Correct policy.
3. Correct action.
4. Correct resource ARN.
5. Group policies.
6. Bucket policy.
7. Explicit Deny.
8. Permissions boundary.
9. SCP if applicable.

Then I would test again.

---

## Scenario 10 — Security Review

**Interviewer:**

> Security asks you to follow least privilege for S3 access. What would you do?

**Answer:**

I would identify the exact actions and resources required and grant only those permissions.

For example, if a user only needs to upload files, I would grant `s3:PutObject` to the required S3 object path instead of granting full S3 access.

# 🧠 PART 3 — HARDER SCENARIOS

### Scenario 11

> A user has read access to S3 but suddenly needs upload access. What would you change?

**Answer:**

I would first verify the requirement.

If upload is genuinely required, I would add the minimum required:

```text
s3:PutObject
```

I would not grant full S3 access.

---

### Scenario 12

> A user can upload to S3 but cannot delete. They say the permissions are broken. What do you say?

**Answer:**

The permissions may be working correctly.

Upload and delete are separate permissions.

I would check whether:

```text
s3:DeleteObject
```

was intentionally omitted.

In a least-privilege design, this may be expected behavior.

---

### Scenario 13

> Why is giving `AdministratorAccess` to solve an AccessDenied problem bad troubleshooting?

**Answer:**

Because it hides the actual permission problem and violates least privilege.

A proper solution is to identify the exact missing permission and grant only what is required.

---

# ⭐ MOST IMPORTANT DAY 1 QUESTIONS

1. Why is S3 AccessDenied happening?
2. Why can I download but not upload?
3. Why can I upload but not delete?
4. How do IAM users receive permissions?
5. What is an IAM group?
6. Why use groups for IAM users?
7. What is an IAM policy?
8. What is an Allow?
9. What is an explicit Deny?
10. What is least privilege?
11. What is an S3 bucket ARN?
12. What is an S3 object ARN?
13. Why does `GetObject` require an object ARN?
14. How do you troubleshoot S3 AccessDenied?
15. Why should you avoid giving full S3 access when only one action is required?

# 🧩 DAY 1 TROUBLESHOOTING FORMULA

```text
              S3 PROBLEM
                   ↓
                 WHO?
                   ↓
              IAM USER
                   ↓
                 WHAT?
                   ↓
           Which action failed?
                   ↓
                WHERE?
                   ↓
            Which resource?
                   ↓
              POLICY?
                   ↓
             Allow / Deny
                   ↓
           EXPLICIT DENY?
                   ↓
              TEST AGAIN
```

**Interview mindset:**

Don't just say:

> "I will check IAM."

Say:

> "I will identify the IAM user, determine the exact failed S3 action, verify the resource ARN, check the applicable policies and explicit denies, and then test the request again."

# ✅ DAY 1 STATUS

* IAM User troubleshooting covered
* IAM Group troubleshooting covered
* IAM Policy troubleshooting covered
* S3 permission troubleshooting covered
* AccessDenied scenarios covered
* Least privilege scenarios covered
* Interview preparation covered

### 📄 File 2 — `Day-02-Troubleshooting-and-Scenario-QA.md`

# Day 2 — EC2 IAM Role → S3

## Troubleshooting and Scenario-Based Interview Q&A

# 🖥️ PART 1 — TROUBLESHOOTING INTERVIEW Q&A

### 1. EC2 cannot access S3 even though the IAM role exists.

**Answer:**

I would first verify that the role is actually attached to the EC2 instance.

Then I would run:

```bash
aws sts get-caller-identity
```

This tells me which identity EC2 is 

using.

Then I would check the role's permissions.

---

### 2. How do you verify which IAM role EC2 is using?

**Answer:**

I would run:

```bash
aws sts get-caller-identity
```

The returned ARN should show the expected assumed role.

---

### 3. `get-caller-identity` shows a different role.

**Answer:**

I would check:

1. IAM role attached to EC2.
2. Instance profile.
3. Environment credentials.
4. AWS CLI credential configuration.
5. Whether another credential source is taking precedence.

---

### 4. EC2 has an IAM role but AWS CLI says credentials are missing.

**Answer:**

I would check whether the IAM role is correctly attached through the EC2 instance profile.

I would also verify that the instance can obtain credentials from the EC2 Instance Metadata Service.

---

### 5. EC2 can list S3 but cannot upload.

**Answer:**

I would check whether the role has:

```text
s3:PutObject
```

Listing and uploading are different permissions.

---

### 6. EC2 can download but cannot delete.

**Answer:**

I would check:

```text
s3:GetObject
s3:DeleteObject
```

The role may have `GetObject` without `DeleteObject`.

---

### 7. EC2 can upload but cannot download.

**Answer:**

I would check whether:

```text
s3:GetObject
```

is allowed.

`PutObject` does not automatically provide `GetObject`.

---

### 8. EC2 role permissions look correct but S3 still returns AccessDenied.

**Answer:**

I would check:

* IAM role.
* Identity policy.
* S3 bucket policy.
* Explicit Deny.
* Resource ARN.
* Permissions boundary.
* SCP.
* S3 encryption/KMS permissions if applicable.

---

### 9. Why would you not put AWS access keys directly on EC2?

**Answer:**

Long-term access keys increase security risk.

For EC2 workloads, I would prefer an IAM role because it provides temporary credentials and avoids storing permanent credentials on the server.

---

### 10. EC2 was working yesterday but S3 access stopped today.

**Answer:**

I would check what changed:

* IAM role.
* IAM policy.
* S3 bucket policy.
* Instance profile.
* SCP.
* KMS permissions if encryption changed.
* Network configuration if the error is connectivity-related.

I would use CloudTrail to investigate recent AWS API changes.

# 🎯 PART 2 — SCENARIO-BASED INTERVIEW Q&A

## Scenario 1 — Production EC2

**Interviewer:**

> You have an EC2 application that needs to upload files to S3. How would you give it access?

**Answer:**

I would create an IAM role for EC2 and attach a least-privilege policy allowing the required S3 actions.

I would avoid storing long-term access keys on the EC2 instance.

---

## Scenario 2 — EC2 Has No Access Keys

**Interviewer:**

> You connect to EC2 and notice there are no AWS access keys configured. How can AWS CLI still work?

**Answer:**

If the EC2 instance has an IAM role attached, the AWS SDK/CLI can obtain temporary credentials associated with that role.

I can verify the identity with:

```bash
aws sts get-caller-identity
```

---

## Scenario 3 — Wrong Role

**Interviewer:**

> Your EC2 application is using the wrong IAM role. What would you do?

**Answer:**

I would verify the current identity using:

```bash
aws sts get-caller-identity
```

Then I would check the EC2 instance's IAM role/instance profile and correct the attachment.

---

## Scenario 4 — Multiple EC2 Instances

**Interviewer:**

> You have 20 EC2 instances running the same application. Do you create 20 IAM users?

**Answer:**

No.

I would use an IAM role that can be attached to the EC2 instances and grant the required permissions through the role.

---

## Scenario 5 — EC2 → S3 Security

**Interviewer:**

> How would you securely allow an EC2 application to access S3?

**Answer:**

I would use an IAM role attached to EC2.

I would create a least-privilege policy that allows only the required S3 actions on the required resources.

I would avoid storing long-term AWS access keys on the server.

---

## Scenario 6 — AccessDenied After Deployment

**Interviewer:**

> Your application worked in testing but gets AccessDenied after deployment. How do you troubleshoot?

**Answer:**

I would compare the identities and permissions between the environments.

I would check:

1. IAM role.
2. `get-caller-identity`.
3. IAM policy.
4. Resource ARN.
5. Bucket policy.
6. Explicit Deny.
7. KMS permissions if encryption is involved.

---

## Scenario 7 — Private EC2

**Interviewer:**

> Your application can access S3 from a public EC2 instance but not from a private EC2 instance. What would you check?

**Answer:**

I would separate the problem into authorization and connectivity.

I would verify IAM first.

If IAM is correct, I would investigate the private subnet's network path, such as NAT Gateway or an S3 VPC endpoint, along with Security Groups and NACLs.

---

## Scenario 8 — Production Credentials

**Interviewer:**

> Security asks you to remove all long-term AWS credentials from your EC2 servers. What would you use?

**Answer:**

I would use IAM roles for EC2 and temporary credentials instead of storing long-term access keys.

---

## Scenario 9 — Troubleshooting Method

**Interviewer:**

> How do you approach an AWS permission issue on EC2?

**Answer:**

I don't immediately add more permissions.

I follow:

```text
WHO?
 ↓
WHAT ACTION?
 ↓
WHICH RESOURCE?
 ↓
IS IT ALLOWED?
 ↓
IS THERE AN EXPLICIT DENY?
 ↓
IS THERE A NETWORK PROBLEM?
 ↓
TEST AGAIN
```

This helps me troubleshoot systematically.

---

# 🧠 PART 3 — HARDER SCENARIOS

### Scenario 10

> An EC2 instance has `AmazonS3ReadOnlyAccess`, but the application needs to upload files. What would you change?

**Answer:**

The read-only policy does not provide upload permission.

I would replace or supplement it with a carefully scoped policy containing:

```text
s3:PutObject
```

rather than giving unnecessary full S3 access.

---

### Scenario 11

> An EC2 instance can upload to S3 but cannot delete. The developer says the role is broken. What do you say?

**Answer:**

The role may be working correctly.

Upload and delete are separate permissions.

I would check whether:

```text
s3:DeleteObject
```

was intentionally omitted.

In a least-privilege design, this may be expected behavior.

---

### Scenario 12

> Your application can access S3 from a public EC2 instance but not from a private EC2 instance.

**Answer:**

I would separate the problem into authorization and connectivity.

I would verify IAM first.

If IAM is correct, I would investigate the private subnet's network path, such as NAT Gateway or an S3 VPC endpoint, along with Security Groups and NACLs.

---

### Scenario 13

> You changed an IAM policy and the application still gets AccessDenied.

**Answer:**

I would not assume the policy change fixed everything.

I would verify:

```text
Correct role?
Correct action?
Correct resource?
Correct policy attached?
Explicit Deny?
Bucket policy?
Permissions boundary?
SCP?
```

Then I would test again.

---

# ⭐ MOST IMPORTANT DAY 2 QUESTIONS

1. How does EC2 access AWS services without access keys?
2. What is an IAM role?
3. How do you attach an IAM role to EC2?
4. What is an instance profile?
5. What does `aws sts get-caller-identity` do?
6. How do you verify which role EC2 is using?
7. Why use an IAM role instead of access keys?
8. What happens if the wrong role is attached?
9. Why can EC2 read but not upload?
10. Why can EC2 upload but not delete?
11. Why can EC2 upload but not download?
12. What causes AccessDenied even when a role exists?
13. How do you troubleshoot EC2 → S3 access?
14. How do you distinguish IAM from networking?
15. How would you secure EC2 → S3 in production?

# 🧩 DAY 2 TROUBLESHOOTING FORMULA

```text
             EC2 → S3 PROBLEM
                     ↓
                   WHO?
                     ↓
            Which IAM role?
                     ↓
          aws sts get-caller-identity
                     ↓
                  WHAT?
                     ↓
          Which S3 action failed?
                     ↓
                  WHERE?
                     ↓
             Which S3 resource?
                     ↓
                IAM POLICY
                     ↓
             Allow / Deny
                     ↓
               NETWORK?
                     ↓
             Test again
```

**Interview mindset:**

Don't just say:

> "I will check the IAM role."

Say:

> "First I will verify the role attached to EC2 using `aws sts get-caller-identity`. Then I will identify the failed S3 action, check the role policy and resource ARN, check for explicit denies, and if the error is a timeout rather than AccessDenied, I will investigate networking."

# ✅ DAY 2 STATUS

* EC2 IAM Role troubleshooting covered
* EC2 → S3 access covered
* STS identity verification covered
* Access key vs IAM role covered
* AccessDenied scenarios covered
* Production security scenarios covered
* Network vs IAM troubleshooting covered

### 📄 File 3 — `Day-03-Troubleshooting-and-Scenario-QA.md`

# Day 3 — Custom IAM Policy + Least Privilege

## Troubleshooting and Scenario-Based Interview Q&A

# 🔐 PART 1 — TROUBLESHOOTING INTERVIEW Q&A

### 1. Why does `aws s3 ls` fail in your Day 3 lab?

**Answer:**

Because the custom policy does not grant:

```text
s3:ListAllMyBuckets
```

The policy only grants `s3:ListBucket` for the specific bucket.

---

### 2. Why does this work?

```bash
aws s3 ls s3://sushma-day3-custom-policy-2026
```

**Answer:**

Because this operation requires permission to list that specific bucket:

```text
s3:ListBucket
```

The custom policy grants that permission.

---

### 3. Upload works but delete fails in Day 3. Why?

**Answer:**

Because the policy grants:

```text
s3:PutObject
```

but does not grant:

```text
s3:DeleteObject
```

This demonstrates least privilege.

---

### 4. Download works but upload fails. What does that tell you?

**Answer:**

It suggests the role has:

```text
s3:GetObject
```

but does not have the required:

```text
s3:PutObject
```

permission, or another policy is denying it.

---

### 5. Specific S3 bucket access works but other buckets don't.

**Answer:**

That is expected if the policy restricts the `Resource` to one bucket.

This is an example of least privilege.

---

### 6. What happens if the bucket ARN is wrong?

**Answer:**

The policy won't match the requested resource, so the required action can result in `AccessDenied`.

I would verify the exact ARN.

---

### 7. What happens if you use the bucket ARN for `GetObject`?

**Answer:**

Object-level actions such as `GetObject` operate on object resources.

Therefore, I would normally use:

```text
arn:aws:s3:::bucket-name/*
```

for objects.

---

### 8. What is the purpose of `/*` in the S3 ARN?

**Answer:**

It represents objects inside the bucket.

For example:

```text
arn:aws:s3:::my-bucket/*
```

means objects within `my-bucket`.

---

### 9. Why does `s3:ListBucket` use the bucket ARN?

**Answer:**

Because `ListBucket` is a bucket-level permission.

Example:

```text
arn:aws:s3:::my-bucket
```

---

### 10. Why does `GetObject` use the object ARN?

**Answer:**

Because `GetObject` operates on individual S3 objects.

Example:

```text
arn:aws:s3:::my-bucket/*
```

# 🌐 PART 2 — NETWORK-RELATED TROUBLESHOOTING

### 11. EC2 cannot access S3. How do you determine whether it is IAM or networking?

**Answer:**

I would first look at the error.

If I receive:

```text
AccessDenied
```

I would investigate IAM/S3 authorization.

If I receive:

```text
timeout
connection error
```

I would investigate networking.

---

### 12. Private EC2 cannot access S3.

**Answer:**

I would check:

1. Route table.
2. NAT Gateway if using NAT.
3. S3 VPC endpoint if using private S3 connectivity.
4. Security Group outbound rules.
5. Network ACL.
6. IAM permissions.

---

### 13. EC2 has correct IAM permissions but S3 connection times out.

**Answer:**

That points more toward a networking problem.

I would check the subnet route, NAT Gateway or S3 VPC endpoint, Security Group, and NACL.

---

### 14. EC2 receives AccessDenied, but networking is working.

**Answer:**

I would focus on authorization:

```text
IAM Role
↓
IAM Policy
↓
S3 Bucket Policy
↓
Resource ARN
↓
Explicit Deny
```

# 🔒 PART 3 — IAM POLICY TROUBLESHOOTING

### 15. How do you troubleshoot an IAM policy that isn't working?

**Answer:**

I follow:

```text
Identity
↓
Action
↓
Resource
↓
Condition
↓
Allow/Deny
↓
Other policy controls
```

I verify each part instead of randomly changing permissions.

---

### 16. Policy has correct Action but wrong Resource. What happens?

**Answer:**

The policy won't authorize the requested resource.

For example, if the policy allows access to bucket A but the application accesses bucket B, the request can be denied.

---

### 17. Policy has correct Resource but wrong Action.

**Answer:**

The requested operation won't be allowed.

For example:

```text
GetObject
```

permission does not automatically allow:

```text
PutObject
```

---

### 18. What is the first thing you check during an AccessDenied problem?

**Answer:**

I identify **who is making the request**.

For EC2 I can use:

```bash
aws sts get-caller-identity
```

Then I determine the exact failed action and resource.

# 🎯 PART 4 — SCENARIO-BASED INTERVIEW Q&A

## Scenario 1 — Upload Only

**Interviewer:**

> An application needs to upload files but never download or delete them. How would you design the policy?

**Answer:**

I would grant:

```text
s3:PutObject
```

for the required bucket/object path.

I would not grant:

```text
s3:GetObject
s3:DeleteObject
```

unless required.

---

## Scenario 2 — Upload + Download

**Interviewer:**

> The application needs to upload and download files but must never delete them.

**Answer:**

I would grant:

```text
s3:PutObject
s3:GetObject
```

but not:

```text
s3:DeleteObject
```

This follows least privilege.

---

## Scenario 3 — Developer Requests Full Access

**Interviewer:**

> A developer asks for `AmazonS3FullAccess` because their application gets AccessDenied. What do you do?

**Answer:**

I would not immediately grant full access.

First, I would identify the exact API operation that failed and grant the minimum required permission.

For example, if upload failed, I would investigate:

```text
s3:PutObject
```

---

## Scenario 4 — Bucket Restriction

**Interviewer:**

> Your company has 100 S3 buckets, but an application should access only one. How would you design it?

**Answer:**

I would restrict the IAM policy's `Resource` to the required bucket and its objects.

I would not grant access to all S3 buckets.

---

## Scenario 5 — Delete Protection

**Interviewer:**

> Developers should be able to upload application files but never delete them.

**Answer:**

I would grant:

```text
s3:PutObject
```

and omit:

```text
s3:DeleteObject
```

I would test the upload and delete operations separately.

---

## Scenario 6 — `aws s3 ls`

**Interviewer:**

> Your EC2 can access the bucket, but `aws s3 ls` returns AccessDenied. Is the IAM configuration necessarily broken?

**Answer:**

No.

`aws s3 ls` without a bucket requires the ability to list buckets.

I would test:

```bash
aws s3 ls s3://my-bucket
```

If that works, the role can access the specific bucket even though it cannot list all buckets.

---

## Scenario 7 — Least Privilege

**Interviewer:**

> Explain how your Day 3 project demonstrates least privilege.

**Answer:**

My EC2 role was given a custom policy that allowed only:

```text
s3:ListBucket
s3:GetObject
s3:PutObject
```

for one specific S3 bucket.

I intentionally did not grant:

```text
s3:DeleteObject
```

or:

```text
s3:ListAllMyBuckets
```

This restricted both the actions and resources.

---

## Scenario 8 — AccessDenied After Deployment

**Interviewer:**

> Your application worked in testing but gets AccessDenied after deployment. How do you troubleshoot?

**Answer:**

I would compare the identities and permissions between the environments.

I would check:

1. IAM role.
2. `get-caller-identity`.
3. IAM policy.
4. Resource ARN.
5. Bucket policy.
6. Explicit Deny.
7. KMS permissions if encryption is involved.

---

## Scenario 9 — One Application, One Bucket

**Interviewer:**

> An application should access only `app-data` and nothing else. What is your approach?

**Answer:**

I would create an IAM role with a custom least-privilege policy restricted to the `app-data` bucket and only the required S3 actions.

---

## Scenario 10 — Troubleshooting Method

**Interviewer:**

> How do you approach an AWS permission issue?

**Answer:**

I don't immediately add more permissions.

I follow:

```text
WHO?
 ↓
WHAT ACTION?
 ↓
WHICH RESOURCE?
 ↓
IS IT ALLOWED?
 ↓
IS THERE AN EXPLICIT DENY?
 ↓
IS THERE A NETWORK PROBLEM?
 ↓
TEST AGAIN
```

This helps me troubleshoot systematically.

# 🧠 PART 5 — HARDER SCENARIOS

### Scenario 11

> An EC2 instance has `AmazonS3ReadOnlyAccess`, but the application needs to upload files. What would you change?

**Answer:**

The read-only policy does not provide upload permission.

I would replace or supplement it with a carefully scoped policy containing:

```text
s3:PutObject
```

rather than giving unnecessary full S3 access.

---

### Scenario 12

> An EC2 instance can upload to S3 but cannot delete. The developer says the role is broken. What do you say?

**Answer:**

The role may be working correctly.

Upload and delete are separate permissions.

I would check whether:

```text
s3:DeleteObject
```

was intentionally omitted.

In a least-privilege design, this may be expected behavior.

---

### Scenario 13

> Your application can access S3 from a public EC2 instance but not from a private EC2 instance.

**Answer:**

I would separate the problem into authorization and connectivity.

I would verify IAM first.

If IAM is correct, I would investigate the private subnet's network path, such as NAT Gateway or an S3 VPC endpoint, along with Security Groups and NACLs.

---

### Scenario 14

> You changed an IAM policy and the application still gets AccessDenied.

**Answer:**

I would not assume the policy change fixed everything.

I would verify:

```text
Correct role?
Correct action?
Correct resource?
Correct policy attached?
Explicit Deny?
Bucket policy?
Permissions boundary?
SCP?
```

Then test again.

---

### Scenario 15

> A user says "I have S3 access." How do you determine exactly what they can do?

**Answer:**

I would identify the user's effective permissions by examining their attached policies, group policies, resource policies and applicable restrictions.

I would focus on the actual actions and resources rather than saying simply:

> "They have S3 access."

---

### Scenario 16

> Why is giving `AdministratorAccess` to solve an AccessDenied problem bad troubleshooting?

**Answer:**

Because it hides the actual permission problem and violates least privilege.

A proper solution is to identify the exact missing permission and grant only what is required.

# ⭐ PART 6 — MOST IMPORTANT QUESTIONS TO MEMORIZE

For your **Day 1–3 interview**, definitely know these:

```text
1. Why is S3 AccessDenied happening?

2. Why can I download but not upload?

3. Why can I upload but not delete?

4. Why does aws s3 ls fail?

5. Why does aws s3 ls s3://bucket work?

6. How do you verify the EC2 IAM role?

7. What does aws sts get-caller-identity do?

8. Why use IAM role instead of access keys?

9. Why does EC2 have a role but still get AccessDenied?

10. Bucket ARN vs object ARN?

11. What does /* mean?

12. What is s3:ListBucket?

13. What is s3:ListAllMyBuckets?

14. What is least privilege?

15. What is explicit Deny?

16. How do you troubleshoot AccessDenied?

17. How do you distinguish IAM problems from networking problems?

18. What happens when the wrong IAM role is attached?

19. Why should you avoid AmazonS3FullAccess when only one action is required?

20. How would you secure EC2 → S3 in production?
```

# 🧩 PART 7 — INTERVIEW TROUBLESHOOTING FORMULA

Memorize this — **not the individual answers**:

```text
              AWS PROBLEM
                   ↓
                WHO?
                   ↓
        Which IAM identity?
                   ↓
                WHAT?
                   ↓
        Which action failed?
                   ↓
                WHERE?
                   ↓
        Which resource?
                   ↓
              PERMISSION?
                   ↓
             Allow / Deny
                   ↓
           EXPLICIT DENY?
                   ↓
               NETWORK?
                   ↓
           Test the request
```

## 🎯 The Interview Mindset

Don't just say:

> "I will check IAM."

Say:

> "First, I will identify who is making the request. Then I will determine the exact action that failed and the resource being accessed. I will verify the applicable policies, check for explicit denies and other policy controls, determine whether the issue is authorization or networking, and then test the request again."

This demonstrates **real troubleshooting ability**, rather than memorized definitions.

# ✅ DAY 3 STATUS

* Custom IAM policy troubleshooting covered
* `ListBucket` vs `ListAllMyBuckets` covered
* Bucket ARN vs object ARN covered
* `/*` covered
* Least privilege covered
* AccessDenied troubleshooting covered
* IAM vs networking covered
* Scenario-based questions covered
* Harder interview scenarios covered
* Production security covered
* Final interview formula covered

