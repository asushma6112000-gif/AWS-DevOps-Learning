# AWS Labs — Actual Project Interview Explanation

## 📌 Purpose of This Document

This document contains the detailed interview explanation of the AWS hands-on labs I actually performed.

The labs were designed to understand AWS IAM, S3 access control, EC2 IAM roles, custom IAM policies, least privilege, IAM policy conditions, AWS CLI authentication, authorization, and troubleshooting.

The four labs are connected because each lab builds on the previous one.

```text
LAB 1
IAM User + IAM Group + S3
        ↓
Basic IAM permissions

LAB 2
EC2 + IAM Role + S3
        ↓
Access AWS services without access keys on EC2

LAB 3
EC2 + Custom IAM Policy + S3
        ↓
Least-privilege permissions

LAB 4
IAM User + Policy Condition + S3
        ↓
Conditional access control
```

---

# LAB 1 — IAM + S3 Basics

## 1. Lab Objective

The objective of Lab 1 was to understand the fundamentals of AWS IAM permissions.

I wanted to understand:

* What an IAM user is
* What an IAM group is
* How IAM policies provide permissions
* How permissions affect S3 operations
* How `Allow` and missing permissions affect access
* How to troubleshoot `AccessDenied`
* How least privilege works

---

# 2. AWS Resources Used

For Lab 1, I worked with:

```text
IAM
 ├── IAM Group
 └── IAM User

S3
 └── S3 Bucket
```

I created an IAM group and IAM user and used an S3 bucket to test permissions.

---

# 3. IAM Group

An IAM group is a collection of IAM users.

Instead of attaching the same policy individually to multiple users, we can attach a policy to a group and add users to that group.

Conceptually:

```text
IAM Group
    │
    ├── User 1
    ├── User 2
    └── User 3
```

The users can receive permissions through the group.

---

# 4. S3 Permission Testing

I tested different S3 operations and observed how IAM permissions affected them.

The important operations I tested were:

```text
s3:GetObject
s3:PutObject
s3:DeleteObject
```

I verified that when a permission was granted, the operation succeeded.

When the required permission was not granted, AWS returned:

```text
AccessDenied
```

---

# 5. What I Learned from Lab 1

The most important concept I learned was:

```text
IAM controls authorization.
```

For example:

```text
User
 ↓
IAM Policy
 ↓
Action
 ↓
Resource
 ↓
Allow / Deny
```

I learned that simply having an IAM user does not mean the user can access every AWS service.

Permissions must be explicitly provided.

---

# 6. Interview Explanation — Lab 1

### Interviewer:

**Tell me about your first AWS hands-on lab.**

### Answer:

"My first lab was focused on IAM and S3 fundamentals. I created an IAM group and IAM user and used an S3 bucket to test different permissions. I tested operations such as reading, uploading, and deleting objects. When the required permission was available, the operation succeeded, and when the permission was not available, AWS returned AccessDenied. Through this lab I understood how IAM policies control authorization and how to troubleshoot permission-related issues."

---

# 7. Possible Follow-up Question

### Interviewer:

**What did you learn from the AccessDenied error?**

### Answer:

"I learned that I should not immediately assume that AWS is malfunctioning. I need to identify the IAM principal, determine the exact API action being attempted, check the resource, review the applicable policies, and look for explicit denies or missing permissions."

My troubleshooting approach is:

```text
WHO?
 ↓
WHAT ACTION?
 ↓
WHICH RESOURCE?
 ↓
WHICH POLICY?
 ↓
ALLOW?
 ↓
EXPLICIT DENY?
```

---

# LAB 2 — EC2 IAM Role → S3

## 1. Lab Objective

The objective of Lab 2 was to understand how an EC2 instance can access AWS services securely using an IAM role.

Instead of storing AWS access keys directly on the EC2 instance, I attached an IAM role to the EC2 instance.

The role had:

```text
AmazonS3ReadOnlyAccess
```

---

# 2. Architecture

The basic architecture was:

```text
EC2 Instance
     │
     │ IAM Role
     ▼
EC2-S3-ReadOnly-Role
     │
     ▼
Amazon S3
```

The EC2 instance received temporary credentials through the IAM role.

---

# 3. IAM Role

I created:

```text
EC2-S3-ReadOnly-Role
```

The trusted entity was:

```text
EC2
```

This means EC2 was allowed to assume the role.

The role had:

```text
AmazonS3ReadOnlyAccess
```

---

# 4. Why Use an IAM Role?

The major reason is security.

A common bad practice would be:

```text
EC2
 ↓
Hard-coded AWS Access Key
 ↓
S3
```

Instead, I used:

```text
EC2
 ↓
IAM Role
 ↓
Temporary Credentials
 ↓
S3
```

This avoids putting long-term AWS access keys inside the EC2 instance.

---

# 5. EC2 Instance

I launched an EC2 instance:

```text
day2-ec2-s3-test
```

The IAM role was attached to the EC2 instance.

I then connected to the instance using EC2 Instance Connect.

---

# 6. STS Identity Test

Inside the EC2 instance, I ran:

```bash
aws sts get-caller-identity
```

The result showed that the AWS CLI was operating using the assumed IAM role.

This was important because it proved that the EC2 instance was receiving AWS permissions through the role.

---

# 7. S3 Testing

I tested S3 operations.

### List S3 resources

```bash
aws s3 ls
```

The command worked because the role had S3 read permissions.

### Download an object

The object download worked.

### Upload an object

Upload failed with:

```text
AccessDenied
```

because the role did not have:

```text
s3:PutObject
```

### Delete an object

Delete also failed because the role did not have:

```text
s3:DeleteObject
```

---

# 8. What Did Lab 2 Prove?

It proved that:

```text
EC2
 ↓
IAM Role
 ↓
Temporary AWS Credentials
 ↓
S3 Access
```

The EC2 instance did not need a manually configured long-term access key to access S3.

---

# 9. Interview Explanation — Lab 2

### Interviewer:

**Tell me about your second AWS lab.**

### Answer:

"In my second lab, I practiced accessing S3 from an EC2 instance using an IAM role. I created an IAM role trusted by EC2 and attached AmazonS3ReadOnlyAccess to the role. I launched an EC2 instance with that role attached and used AWS CLI commands inside the instance. I verified the identity using `aws sts get-caller-identity`. I was able to list and download S3 objects, while upload and delete operations returned AccessDenied because those permissions were not granted. This helped me understand IAM roles, temporary credentials, and secure service-to-service access."

---

# 10. Interview Follow-up

### Interviewer:

**Why is an IAM role better than storing access keys on EC2?**

### Answer:

"An IAM role provides temporary credentials to the EC2 instance and avoids storing long-term access keys on the server. This reduces the risk of credential exposure and makes credential rotation easier because AWS manages the temporary credentials."

---

# LAB 3 — EC2 → S3 Custom IAM Policy + Least Privilege

## 1. Lab Objective

The objective of Lab 3 was to move from a broad managed policy to a more restricted custom IAM policy.

I wanted to understand:

* Custom IAM policies
* Resource-level permissions
* S3 bucket ARN
* S3 object ARN
* Least privilege
* Difference between bucket-level and object-level permissions
* Permission testing

---

# 2. AWS Resources

I created:

```text
S3 Bucket:
sushma-day3-custom-policy-2026
```

IAM role:

```text
Day3-EC2-S3-Custom-Role
```

IAM policy:

```text
Day3-EC2-S3-Custom-Policy
```

EC2:

```text
day3-ec2-custom-policy
```

---

# 3. Custom Policy

The policy was:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::sushma-day3-custom-policy-2026"
    },
    {
      "Sid": "ReadWriteObjects",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::sushma-day3-custom-policy-2026/*"
    }
  ]
}
```

---

# 4. Policy Explanation

The policy had two statements.

## Statement 1 — ListBucket

```text
Action:
s3:ListBucket
```

Resource:

```text
arn:aws:s3:::sushma-day3-custom-policy-2026
```

This is the bucket ARN.

It allows listing objects in that specific bucket.

---

# 5. Statement 2 — Object Permissions

The actions were:

```text
s3:GetObject
s3:PutObject
```

The resource was:

```text
arn:aws:s3:::sushma-day3-custom-policy-2026/*
```

The `/*` represents objects inside the bucket.

Therefore:

```text
Bucket ARN
 ↓
s3:ListBucket

Object ARN
 ↓
s3:GetObject
s3:PutObject
```

---

# 6. What Was NOT Allowed?

The custom policy did not grant:

```text
s3:DeleteObject
```

It also did not grant access to unrelated buckets.

It did not grant:

```text
s3:ListAllMyBuckets
```

---

# 7. Testing

I tested:

```text
STS identity
 ↓
Specific bucket listing
 ↓
Upload
 ↓
Download
 ↓
Delete
```

The results were:

```text
Specific bucket listing → Success
Upload                 → Success
Download               → Success
Delete                 → AccessDenied
```

---

# 8. Important Test — `aws s3 ls`

I also tested:

```bash
aws s3 ls
```

This failed because the command requires the ability to list buckets across the account, which involves:

```text
s3:ListAllMyBuckets
```

My custom policy did not grant that permission.

However, listing the specific bucket was allowed because:

```text
s3:ListBucket
```

was granted for that bucket.

This helped me understand that:

```text
s3:ListAllMyBuckets
```

and:

```text
s3:ListBucket
```

are different permissions.

---

# 9. What Did Lab 3 Teach Me?

Lab 3 taught me that using a custom policy allows much more precise access control.

Instead of saying:

```text
Give EC2 broad S3 access
```

I could define:

```text
Specific bucket
+
Specific actions
+
Specific objects
```

This is the principle of least privilege.

---

# 10. Interview Explanation — Lab 3

### Interviewer:

**Tell me about your third AWS lab.**

### Answer:

"In my third lab, I practiced creating a custom least-privilege IAM policy for an EC2 instance to access S3. I created a custom policy that allowed `s3:ListBucket` only on a specific bucket and allowed `s3:GetObject` and `s3:PutObject` only on objects inside that bucket. I intentionally did not grant `s3:DeleteObject` or broad bucket-listing permissions. During testing, listing the specific bucket, uploading, and downloading worked, while deleting an object was denied. I also tested `aws s3 ls`, which failed because the policy did not grant `s3:ListAllMyBuckets`. This lab helped me understand custom IAM policies, resource ARNs, and least privilege."

---

# 11. Interview Follow-up

### Interviewer:

**Why do you need two different S3 resource ARNs?**

### Answer:

"S3 has bucket-level and object-level actions. `s3:ListBucket` is a bucket-level permission, so I used the bucket ARN without `/*`. `s3:GetObject` and `s3:PutObject` operate on objects, so I used the object ARN with `/*`."

---

# LAB 4 — IAM Policy Conditions

## 1. Lab Objective

The objective of Lab 4 was to understand IAM Policy Conditions.

I created an IAM user and attached a custom policy containing:

```text
aws:RequestedRegion
```

The policy allowed:

```text
s3:GetObject
```

with the condition:

```text
aws:RequestedRegion = ap-south-1
```

---

# 2. AWS Resources

IAM user:

```text
day4-policy-test-user
```

IAM policy:

```text
Day4-S3-Region-Condition-Policy
```

S3 bucket:

```text
sushma-day4-policy-conditions-2026
```

Region:

```text
ap-south-1
```

AWS CLI profile:

```text
day4
```

---

# 3. IAM Policy

The policy was:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnlyFromMumbaiRegion",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::sushma-day4-policy-conditions-2026/*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "ap-south-1"
        }
      }
    }
  ]
}
```

---

# 4. Policy Explanation

The policy allowed:

```text
s3:GetObject
```

on:

```text
sushma-day4-policy-conditions-2026/*
```

and included the condition:

```text
aws:RequestedRegion = ap-south-1
```

So the permission was more specific than a simple Allow statement.

---

# 5. AWS CLI Profile

I created a dedicated AWS CLI profile:

```text
day4
```

using:

```bash
aws configure --profile day4
```

I configured:

```text
Region:
ap-south-1
```

Then I verified the identity:

```bash
aws sts get-caller-identity --profile day4
```

The result confirmed that the CLI was using:

```text
day4-policy-test-user
```

---

# 6. Test File

I created:

```bash
echo "Day 4 IAM Policy Conditions Test" > day4-test.txt
```

Then verified it using:

```bash
cat day4-test.txt
```

---

# 7. ListBucket Test

I tested:

```bash
aws s3 ls s3://sushma-day4-policy-conditions-2026 --profile day4
```

The result was:

```text
AccessDenied
```

The reason was that the policy did not grant:

```text
s3:ListBucket
```

The policy only granted:

```text
s3:GetObject
```

---

# 8. GetObject Test

I downloaded the object using:

```bash
aws s3api get-object \
  --bucket sushma-day4-policy-conditions-2026 \
  --key day4-test.txt \
  day4-downloaded.txt \
  --region ap-south-1 \
  --profile day4
```

The download succeeded.

I then verified the file:

```bash
cat day4-downloaded.txt
```

The output was:

```text
Day 4 IAM Policy Conditions Test
```

---

# 9. What Did Lab 4 Teach Me?

Lab 4 taught me that IAM permissions can be controlled using additional conditions.

The basic permission model is:

```text
Principal
    ↓
Action
    ↓
Resource
    ↓
Condition
```

This allows more granular security controls.

I also learned how to troubleshoot AWS CLI credential problems using:

```bash
aws sts get-caller-identity
```

---

# 10. Interview Explanation — Lab 4

### Interviewer:

**Tell me about your fourth AWS lab.**

### Answer:

"In my fourth lab, I practiced IAM Policy Conditions. I created an IAM user and attached a custom policy that allowed `s3:GetObject` on objects in a specific S3 bucket, with a condition using `aws:RequestedRegion` set to `ap-south-1`. I created a separate AWS CLI profile for that user and verified the identity using STS. I tested the S3 permissions and confirmed that object retrieval worked, while bucket listing returned AccessDenied because `s3:ListBucket` was not granted. This lab helped me understand conditional IAM permissions, least privilege, AWS CLI profiles, and permission troubleshooting."

---

# 11. Important Interview Point

Do not say:

> "I tested the policy from another region and confirmed it was denied."

I did not perform that test in this lab.

The actual test I performed was:

```text
GetObject using ap-south-1 → Success
ListBucket → AccessDenied
```

The condition was configured as:

```text
aws:RequestedRegion = ap-south-1
```

This is important because interview answers should describe what I actually tested.

---

# COMPARING ALL 4 LABS

## Lab 1

```text
IAM User
+
IAM Group
+
S3
```

Main learning:

```text
Basic IAM permissions
```

---

## Lab 2

```text
EC2
+
IAM Role
+
S3
```

Main learning:

```text
Secure AWS service access using IAM roles
```

---

## Lab 3

```text
EC2
+
Custom IAM Policy
+
S3
```

Main learning:

```text
Least privilege
+
Resource-level permissions
```

---

## Lab 4

```text
IAM User
+
Custom Policy
+
Condition
+
S3
```

Main learning:

```text
Conditional access control
```

---

# HOW THE 4 LABS PROGRESSIVELY BUILD KNOWLEDGE

The progression was:

```text
LAB 1
Understand IAM permissions
        ↓
LAB 2
Understand IAM Roles
        ↓
LAB 3
Understand Custom Policies
        ↓
LAB 4
Understand Policy Conditions
```

So the labs progressed from basic IAM concepts to more granular access control.

---

# FINAL INTERVIEW EXPLANATION — ALL 4 LABS

## 30-Second Answer

"I have completed four AWS IAM hands-on labs. First, I practiced IAM users, groups, policies, and S3 permissions. Second, I accessed S3 from an EC2 instance using an IAM role instead of storing access keys on the server. Third, I created a custom least-privilege policy for EC2-to-S3 access and tested specific allowed and denied actions. Fourth, I practiced IAM Policy Conditions using `aws:RequestedRegion` and tested the policy through an AWS CLI profile. Across these labs, I practiced IAM authorization, least privilege, roles, policies, conditions, AWS CLI, and AccessDenied troubleshooting."

---

# FINAL INTERVIEW EXPLANATION — 1 MINUTE

"I have been building my AWS fundamentals through hands-on IAM and S3 labs.

In my first lab, I worked with IAM users, groups, policies, and S3 permissions and tested allowed and denied operations.

In my second lab, I created an IAM role trusted by EC2 and attached S3 read-only permissions. I launched an EC2 instance with the role and verified that the instance could access S3 without storing long-term access keys.

In my third lab, I created a custom IAM policy following least privilege. I allowed only specific S3 actions on a specific bucket and its objects, and verified that unnecessary actions such as deleting objects were denied.

In my fourth lab, I learned IAM Policy Conditions. I created a policy using `aws:RequestedRegion` and tested S3 object access using a dedicated AWS CLI profile. I also practiced troubleshooting invalid credentials and AccessDenied errors.

These labs helped me understand IAM authentication, authorization, roles, policies, resource-level permissions, conditions, and security best practices."

---

# FINAL INTERVIEW EXPLANATION — 2 MINUTES

"I have completed four practical AWS labs focused mainly on IAM, S3, EC2, and security.

The first lab was IAM and S3 fundamentals. I created an IAM group and user and tested different S3 operations. This helped me understand that AWS access is controlled through IAM policies and that missing permissions can result in AccessDenied.

The second lab focused on EC2 IAM roles. I created a role trusted by EC2 and attached AmazonS3ReadOnlyAccess. I launched an EC2 instance with the role and used AWS CLI from the instance. I verified the identity using STS and tested S3 access. Reading and downloading worked, while uploading and deleting failed because those permissions were not granted. This helped me understand temporary credentials and why IAM roles are preferred over hard-coded access keys on EC2.

The third lab focused on custom IAM policies and least privilege. I created a custom policy that allowed `s3:ListBucket` only on a specific bucket and allowed `s3:GetObject` and `s3:PutObject` only on objects within that bucket. I intentionally did not grant delete permission or broad bucket-listing permission. I tested the policy and verified the expected allowed and denied operations.

The fourth lab focused on IAM Policy Conditions. I created an IAM user and custom policy with `aws:RequestedRegion` set to `ap-south-1`. I configured a dedicated AWS CLI profile and verified the identity using STS. I successfully retrieved an S3 object using the Mumbai region, while listing the bucket was denied because `s3:ListBucket` was not included.

Overall, these labs gave me practical experience with IAM users, groups, roles, policies, conditions, S3 permissions, EC2 service access, AWS CLI, STS identity verification, least privilege, and troubleshooting AccessDenied and credential errors."

---

# COMMON INTERVIEW FOLLOW-UP QUESTIONS

## Q1. Which lab taught you least privilege?

**Answer:**

Lab 3. I created a custom policy with only the required S3 actions and resources.

---

## Q2. Which lab taught you IAM Roles?

**Answer:**

Lab 2. I attached an IAM role to EC2 and used it to access S3 without storing long-term credentials on the instance.

---

## Q3. Which lab taught you IAM Conditions?

**Answer:**

Lab 4. I used `aws:RequestedRegion` as a condition in the S3 policy.

---

## Q4. Which lab taught you custom policies?

**Answer:**

Lab 3 and Lab 4.

Lab 3 focused on resource-level least privilege.

Lab 4 focused on conditions.

---

## Q5. How did you troubleshoot AccessDenied?

**Answer:**

I first identified the IAM identity, then checked the exact AWS action being performed, the resource, the attached policy, and whether the required permission was granted.

I also checked for explicit denies when applicable.

---

## Q6. What command do you use to identify the current AWS identity?

**Answer:**

```bash
aws sts get-caller-identity
```

If I am using a specific profile:

```bash
aws sts get-caller-identity --profile day4
```

---

## Q7. What is the difference between an IAM User and an IAM Role?

**Answer:**

An IAM user represents a long-term AWS identity that can have credentials.

An IAM role is an identity that can be assumed by trusted principals such as EC2, Lambda, or users and provides temporary credentials.

---

## Q8. Why shouldn't we put AWS access keys directly into application code?

**Answer:**

Because long-term credentials can be exposed through source code, logs, repositories, or configuration files.

For AWS workloads such as EC2, an IAM role is preferred because it provides temporary credentials.

---

## Q9. What is least privilege?

**Answer:**

Least privilege means giving an identity only the permissions required to perform its task and nothing more.

---

## Q10. What is the difference between `GetObject` and `ListBucket`?

**Answer:**

`GetObject` allows access to an object.

`ListBucket` allows listing objects in a bucket.

They are different permissions and use different resource scopes.

---

# MY AWS TROUBLESHOOTING APPROACH

When I receive an AWS permission error, I follow this approach:

```text
1. Identify the IAM identity
        ↓
2. Identify the exact API action
        ↓
3. Identify the resource
        ↓
4. Check attached policies
        ↓
5. Check whether the action is allowed
        ↓
6. Check conditions
        ↓
7. Check for explicit Deny
        ↓
8. Test again
```

Example:

```text
AccessDenied
     ↓
Who?
     ↓
What action?
     ↓
Which resource?
     ↓
Which policy?
     ↓
Condition?
     ↓
Explicit Deny?
     ↓
Retest
```

---

# KEY AWS COMMANDS I PRACTICED

## Check AWS Identity

```bash
aws sts get-caller-identity
```

## Check Identity with Profile

```bash
aws sts get-caller-identity --profile day4
```

## Configure Profile

```bash
aws configure --profile day4
```

## List S3 Buckets

```bash
aws s3 ls --profile day4
```

## List a Specific S3 Bucket

```bash
aws s3 ls s3://BUCKET-NAME --profile day4
```

## Download S3 Object

```bash
aws s3api get-object \
  --bucket BUCKET-NAME \
  --key OBJECT-NAME \
  downloaded-file.txt \
  --region ap-south-1 \
  --profile day4
```

---

# WHAT I CAN CONFIDENTLY SAY IN AN INTERVIEW

I can confidently say that I have hands-on experience with:

```text
✅ IAM Users
✅ IAM Groups
✅ IAM Policies
✅ IAM Roles
✅ EC2 IAM Roles
✅ S3 Permissions
✅ Custom IAM Policies
✅ Least Privilege
✅ Resource ARNs
✅ IAM Policy Conditions
✅ aws:RequestedRegion
✅ AWS CLI
✅ AWS CLI Profiles
✅ AWS STS
✅ AccessDenied Troubleshooting
✅ Credential Troubleshooting
```

---

# FINAL SUMMARY

My four labs demonstrate a progression from basic AWS identity and access management to more advanced permission control.

```text
                    AWS IAM LEARNING

LAB 1
IAM User + Group + S3
        │
        ▼
Basic Permissions

LAB 2
EC2 + IAM Role + S3
        │
        ▼
Temporary Credentials

LAB 3
Custom Policy + S3
        │
        ▼
Least Privilege

LAB 4
Policy Condition + S3
        │
        ▼
Conditional Access
```

The main concepts I gained from these labs are:

```text
Authentication
Authorization
IAM Users
IAM Groups
IAM Roles
IAM Policies
Custom Policies
Conditions
Resource ARNs
Least Privilege
S3 Permissions
EC2 Service Access
AWS CLI
STS
AccessDenied Troubleshooting
```

These four labs form the foundation for the security and IAM concepts I will use in later AWS DevOps work.

