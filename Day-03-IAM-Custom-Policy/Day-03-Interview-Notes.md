# Day 3 — IAM Custom Policy Interview Notes

## 🎯 Interview Preparation

### Project: EC2 → S3 Using a Custom IAM Policy

This document contains important interview questions and answers based on the Day 3 hands-on project.

---

# 1. What did you do in your Day 3 project?

**Answer:**

In Day 3, I created a **customer-managed IAM policy** with specific S3 permissions and attached it to an IAM Role used by an EC2 instance.

The policy allowed:

* `s3:ListBucket`
* `s3:GetObject`
* `s3:PutObject`

It did not allow:

* `s3:ListAllMyBuckets`
* `s3:DeleteObject`

I then tested these permissions from the EC2 instance using AWS CLI.

---

# 2. What is an IAM Policy?

An IAM Policy is a JSON document that defines **what actions an identity is allowed or denied to perform on AWS resources**.

Example:

```text
IAM Policy
     ↓
Defines permissions
     ↓
Who can do what
     ↓
On which AWS resource
```

---

# 3. What type of policy did you create?

I created a **Customer Managed Policy**.

Policy name:

```text
Day3-EC2-S3-Custom-Policy
```

A customer-managed policy is created and managed by the AWS customer.

---

# 4. Why did you create a custom policy instead of using AmazonS3FullAccess?

Because `AmazonS3FullAccess` provides much broader permissions than the application required.

I wanted to follow the **Principle of Least Privilege**.

The EC2 instance only needed:

```text
ListBucket
GetObject
PutObject
```

So I granted only those permissions.

---

# 5. What is the Principle of Least Privilege?

The Principle of Least Privilege means giving an identity **only the permissions it needs to perform its required task and nothing more**.

In my project:

```text
Required:
List → Read → Upload

Not required:
Delete → Denied
List all buckets → Denied
```

---

# 6. What was the name of your IAM policy?

```text
Day3-EC2-S3-Custom-Policy
```

---

# 7. Explain your IAM policy JSON.

My policy contains two statements.

The first statement allows:

```text
s3:ListBucket
```

on the specific S3 bucket.

The second statement allows:

```text
s3:GetObject
s3:PutObject
```

on objects inside that bucket.

---

# 8. What does `"Version": "2012-10-17"` mean?

It specifies the version of the IAM policy language syntax being used.

It does not mean that the policy was created in 2012.

---

# 9. What is `Effect` in an IAM Policy?

`Effect` specifies whether the statement:

```text
Allow
```

or

```text
Deny
```

the specified action.

Example:

```json
"Effect": "Allow"
```

---

# 10. What is `Action`?

`Action` specifies the AWS API operation that the policy allows or denies.

For example:

```text
s3:GetObject
s3:PutObject
s3:ListBucket
```

---

# 11. What is `Resource`?

`Resource` specifies the AWS resource on which the action can be performed.

In my project, the policy was restricted to one specific S3 bucket.

---

# 12. What is `Sid`?

`Sid` means **Statement ID**.

It is used to identify a particular statement within an IAM policy.

Example:

```text
ListBucket
ReadWriteObjects
```

---

# 13. Why are there two different S3 ARNs in your policy?

Because S3 permissions can apply at different resource levels.

Bucket-level ARN:

```text
arn:aws:s3:::sushma-day3-custom-policy-2026
```

Object-level ARN:

```text
arn:aws:s3:::sushma-day3-custom-policy-2026/*
```

---

# 14. What is the difference between a bucket ARN and an object ARN?

### Bucket ARN

```text
arn:aws:s3:::bucket-name
```

Identifies the S3 bucket itself.

Used for bucket-level actions such as:

```text
s3:ListBucket
```

### Object ARN

```text
arn:aws:s3:::bucket-name/*
```

Identifies objects inside the bucket.

Used for object-level actions such as:

```text
s3:GetObject
s3:PutObject
s3:DeleteObject
```

---

# 15. Why did you use `/*` in the object ARN?

The `/*` means the permissions apply to **objects inside the bucket**.

For example:

```text
arn:aws:s3:::my-bucket/*
```

means objects such as:

```text
my-bucket/file1.txt
my-bucket/file2.txt
my-bucket/images/photo.jpg
```

---

# 16. What is `s3:ListBucket`?

`s3:ListBucket` allows an identity to list objects within a specific S3 bucket.

It is a **bucket-level permission**.

---

# 17. What is `s3:GetObject`?

`s3:GetObject` allows an identity to read or download an object from S3.

It is an **object-level permission**.

---

# 18. What is `s3:PutObject`?

`s3:PutObject` allows an identity to upload an object to S3.

It is an **object-level permission**.

---

# 19. Why did you not add `s3:DeleteObject`?

Delete was not required for the project.

To demonstrate least privilege, I intentionally did not grant:

```text
s3:DeleteObject
```

When I tested deletion, AWS returned `AccessDenied`.

---

# 20. What is `s3:ListAllMyBuckets`?

`s3:ListAllMyBuckets` allows an identity to list the S3 buckets that it has access to in the account.

This permission was intentionally not included in my custom policy.

---

# 21. Why did `aws s3 ls` fail?

I ran:

```bash
aws s3 ls
```

Without specifying a bucket, this command attempts to list buckets.

The request requires:

```text
s3:ListAllMyBuckets
```

Since my policy did not grant that permission, AWS returned:

```text
AccessDenied
```

This was expected.

---

# 22. Why did this command work?

```bash
aws s3 ls s3://sushma-day3-custom-policy-2026
```

It worked because the policy grants:

```text
s3:ListBucket
```

for that specific bucket.

---

# 23. What is an IAM Role?

An IAM Role is an AWS identity that provides permissions through temporary credentials.

A role is commonly used by AWS services such as:

* EC2
* Lambda
* ECS
* EKS

---

# 24. What IAM Role did you create?

```text
Day3-EC2-S3-Custom-Role
```

The trusted entity was:

```text
EC2
```

---

# 25. Why did you use an IAM Role for EC2?

Because applications running on EC2 should not require hard-coded AWS access keys.

The IAM Role allows EC2 to obtain temporary credentials automatically.

---

# 26. Did you configure AWS access keys on EC2?

No.

I did not configure:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

Instead, the EC2 instance used its IAM Role.

---

# 27. How does EC2 get permissions from an IAM Role?

The basic flow is:

```text
EC2
 ↓
IAM Role
 ↓
Temporary Credentials
 ↓
AWS STS
 ↓
AWS Service
```

The EC2 instance can then make authorized AWS API requests.

---

# 28. What is AWS STS?

AWS STS stands for **AWS Security Token Service**.

It provides temporary security credentials for AWS resources and identities.

In my project, I used STS to verify which IAM Role the EC2 instance was using.

---

# 29. Which command did you use to verify the IAM Role?

```bash
aws sts get-caller-identity
```

This command displays information about the identity making the AWS API request.

---

# 30. What did `get-caller-identity` prove?

It proved that the EC2 instance was operating using:

```text
Day3-EC2-S3-Custom-Role
```

Therefore, the EC2 instance was successfully using the IAM Role.

---

# 31. What happens when EC2 calls S3?

The basic flow is:

```text
EC2
 ↓
IAM Role Credentials
 ↓
AWS API Request
 ↓
IAM Authorization
 ↓
S3
```

IAM checks whether the requested action is allowed for the requested resource.

---

# 32. What happens if the IAM policy does not allow an action?

AWS denies the request.

For example, my policy did not allow:

```text
s3:DeleteObject
```

So:

```bash
aws s3 rm s3://bucket/test.txt
```

returned:

```text
AccessDenied
```

---

# 33. Did upload work?

Yes.

Command:

```bash
aws s3 cp test.txt s3://sushma-day3-custom-policy-2026/
```

The upload succeeded because the policy allows:

```text
s3:PutObject
```

---

# 34. Did download work?

Yes.

Command:

```bash
aws s3 cp s3://sushma-day3-custom-policy-2026/test.txt downloaded.txt
```

The download succeeded because the policy allows:

```text
s3:GetObject
```

---

# 35. Did delete work?

No.

The delete operation returned:

```text
AccessDenied
```

because the policy does not include:

```text
s3:DeleteObject
```

---

# 36. How did you verify that the delete was actually denied?

After the failed delete operation, I ran:

```bash
aws s3 ls s3://sushma-day3-custom-policy-2026
```

The object:

```text
test.txt
```

was still present.

This confirmed that deletion was denied.

---

# 37. What test file did you create?

I created:

```text
test.txt
```

with the content:

```text
Day 3 Custom IAM Policy Test
```

---

# 38. How did you verify the downloaded file?

I used:

```bash
cat downloaded.txt
```

The output was:

```text
Day 3 Custom IAM Policy Test
```

This confirmed that the download was successful.

---

# 39. What is the difference between IAM Role and IAM Policy?

### IAM Role

An IAM Role is an identity that can be assumed by a trusted entity.

### IAM Policy

An IAM Policy defines what permissions are allowed or denied.

Simple way to remember:

```text
Role = Identity
Policy = Permissions
```

---

# 40. What is the difference between Trust Policy and Permissions Policy?

### Trust Policy

Defines **who can assume the role**.

In my project:

```text
EC2 → Can assume the role
```

### Permissions Policy

Defines **what the role is allowed to do**.

In my project:

```text
Role → List / Read / Upload to S3
```

Memory trick:

```text
Trust Policy
     ↓
Who can assume?

Permissions Policy
     ↓
What can they do?
```

---

# 41. What is a trusted entity?

A trusted entity is the AWS principal that is allowed to assume an IAM Role.

In my project:

```text
Trusted Entity = EC2
```

---

# 42. Why is the custom policy called least privilege?

Because it grants only the permissions required for the task.

```text
Allowed:
s3:ListBucket
s3:GetObject
s3:PutObject

Not allowed:
s3:ListAllMyBuckets
s3:DeleteObject
```

The permissions are also restricted to one specific bucket.

---

# 43. Why is resource restriction important?

Suppose an EC2 instance is compromised.

If the role has access to every S3 bucket, an attacker may be able to access many resources.

If access is restricted to one bucket, the potential impact is reduced.

Therefore, resource-level restrictions improve security.

---

# 44. What is a Customer Managed Policy?

A Customer Managed Policy is an IAM policy created and managed by the AWS customer.

It can be:

* Customized
* Attached to multiple identities
* Updated when requirements change
* Versioned

---

# 45. Customer Managed Policy vs AWS Managed Policy?

### AWS Managed Policy

Created and maintained by AWS.

Example:

```text
AmazonS3ReadOnlyAccess
```

### Customer Managed Policy

Created and managed by the customer.

Example:

```text
Day3-EC2-S3-Custom-Policy
```

---

# 46. Why is a custom policy useful in production?

A custom policy allows an organization to define exactly what an application needs.

For example:

```text
Application A
     ↓
Only Bucket A
     ↓
Read + Write
```

instead of giving:

```text
All S3
Full Access
```

---

# 47. What would happen if you added `s3:DeleteObject`?

The EC2 instance would be authorized to delete objects covered by the resource ARN.

For example:

```text
s3:DeleteObject
```

would allow the delete command to succeed, assuming no other policy or resource-based control denies it.

---

# 48. What would happen if you removed `s3:GetObject`?

The EC2 instance would no longer be authorized to download/read objects from the bucket.

The upload operation could still work because `s3:PutObject` would remain allowed.

---

# 49. What would happen if you removed `s3:PutObject`?

The EC2 instance would no longer be able to upload objects.

The download operation could still work if `s3:GetObject` remained allowed.

---

# 50. What would happen if you removed `s3:ListBucket`?

The EC2 instance would not be able to list the objects inside the bucket.

However, if `s3:GetObject` permission remained, it could potentially access a known object directly.

---

# 51. What is the difference between authentication and authorization?

### Authentication

Authentication answers:

> Who are you?

Example:

```text
EC2 assumes IAM Role
```

### Authorization

Authorization answers:

> What are you allowed to do?

Example:

```text
Can EC2 upload to S3?
Yes → s3:PutObject
```

Simple memory:

```text
Authentication = Who?
Authorization = What can you do?
```

---

# 52. What does AccessDenied mean?

`AccessDenied` means AWS received the request but the identity does not have the required authorization for that operation, or another applicable policy/control prevents it.

In my project, `AccessDenied` was expected for:

```text
s3:ListAllMyBuckets
s3:DeleteObject
```

---

# 53. Why is AccessDenied useful in your project?

It proves that the custom policy is restricting permissions correctly.

For example:

```text
Upload → Allowed
Download → Allowed
Delete → Denied
```

This demonstrates that the policy is not giving unnecessary permissions.

---

# 54. How would you troubleshoot an AccessDenied error?

I would check:

1. Which IAM identity is making the request.
2. Whether the correct IAM Role is attached.
3. Whether the required action exists in the policy.
4. Whether the resource ARN is correct.
5. Whether there is an explicit Deny.
6. Whether a bucket policy affects the request.
7. Whether an SCP or permissions boundary restricts the request.

I would also use:

```bash
aws sts get-caller-identity
```

to verify the identity.

---

# 55. What is an explicit Deny?

An explicit `Deny` statement specifically denies an action.

An explicit Deny generally overrides an Allow.

Example:

```json
{
  "Effect": "Deny",
  "Action": "s3:DeleteObject",
  "Resource": "*"
}
```

---

# 56. What is an implicit deny?

AWS follows a default-deny model.

If there is no applicable Allow for an action, the action is denied.

In my project:

```text
s3:DeleteObject
```

was not present in the policy, so there was no Allow for that action.

Therefore, the request was denied.

---

# 57. Explain your complete Day 3 access flow.

```text
EC2 Instance
      ↓
Assumes IAM Role
      ↓
Day3-EC2-S3-Custom-Role
      ↓
Custom IAM Policy
      ↓
IAM Authorization
      ↓
Amazon S3
```

Allowed:

```text
ListBucket
GetObject
PutObject
```

Denied:

```text
ListAllMyBuckets
DeleteObject
```

---

# 58. What security concept did you demonstrate?

The main security concept was:

```text
Principle of Least Privilege
```

I granted only the permissions required by the EC2 instance and restricted those permissions to a specific S3 bucket.

---

# 59. How would you improve this project for production?

For a production environment, I would:

* Follow least privilege.
* Avoid long-term access keys on EC2.
* Use IAM Roles.
* Restrict resources using specific ARNs.
* Enable S3 Block Public Access.
* Enable S3 encryption.
* Enable CloudTrail for auditing.
* Use IAM Access Analyzer.
* Regularly review unused permissions.
* Monitor CloudWatch and CloudTrail logs.
* Avoid unnecessary broad managed policies.

---

# 60. Give a 30-second interview explanation.

> "In my Day 3 project, I created a customer-managed IAM policy following the Principle of Least Privilege. The policy allowed an EC2 instance to list objects in a specific S3 bucket, upload objects, and download objects. I attached the policy to an EC2 IAM Role and assigned the role to an Amazon Linux instance. I verified the role using AWS STS and tested the permissions using AWS CLI. Upload, download, and specific bucket listing succeeded, while listing all buckets and deleting objects were denied because those permissions were not granted."

---

# ⭐ Most Important Interview Points

These are the points I should remember first:

### 1. IAM Policy

```text
Defines permissions.
```

### 2. IAM Role

```text
Identity that can be assumed by AWS services or other trusted principals.
```

### 3. Trust Policy

```text
Defines who can assume the role.
```

### 4. Permissions Policy

```text
Defines what the role can do.
```

### 5. Least Privilege

```text
Give only required permissions.
```

### 6. Bucket-Level Permission

```text
s3:ListBucket
```

Uses:

```text
arn:aws:s3:::bucket-name
```

### 7. Object-Level Permissions

```text
s3:GetObject
s3:PutObject
s3:DeleteObject
```

Usually use:

```text
arn:aws:s3:::bucket-name/*
```

### 8. STS

```text
AWS Security Token Service
```

Provides temporary credentials.

### 9. EC2 Role Flow

```text
EC2
 ↓
IAM Role
 ↓
Temporary Credentials
 ↓
AWS Service
```

### 10. `aws sts get-caller-identity`

Used to verify the identity making the AWS request.

### 11. `aws s3 ls`

Without a bucket:

```text
Requires ListAllMyBuckets
```

### 12. Specific bucket listing

```bash
aws s3 ls s3://bucket-name
```

Requires:

```text
s3:ListBucket
```

### 13. Upload

```text
s3:PutObject
```

### 14. Download

```text
s3:GetObject
```

### 15. Delete

```text
s3:DeleteObject
```

If it is not allowed → `AccessDenied`.

---

# 🧠 Day 3 Memory Map

```text
                    IAM
                     │
          ┌──────────┴──────────┐
          │                     │
        Role                  Policy
          │                     │
     Who can assume?       What can they do?
          │                     │
         EC2          List + Read + Upload
          │                     │
          └──────────┬──────────┘
                     │
                     ▼
                    S3
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      List          Read         Upload
   ListBucket     GetObject     PutObject
        │            │            │
        └────────────┼────────────┘
                     │
                     ▼
              Least Privilege
```

---

# 📌 Day 3 Key Takeaway

The most important lesson from Day 3 is:

> **Don't give an AWS identity more permissions than it needs.**

Instead of:

```text
EC2
 ↓
AmazonS3FullAccess
```

I implemented:

```text
EC2
 ↓
IAM Role
 ↓
Custom IAM Policy
 ↓
Specific S3 Bucket
 ↓
Only Required Permissions
```

This is the practical application of the **Principle of Least Privilege**.

---

# ✅ Day 3 Interview Preparation Status

**Concepts Covered:**

* IAM Policy
* Customer Managed Policy
* IAM Role
* Trust Policy
* Permissions Policy
* S3 Bucket ARN
* S3 Object ARN
* `s3:ListBucket`
* `s3:GetObject`
* `s3:PutObject`
* `s3:DeleteObject`
* `s3:ListAllMyBuckets`
* AWS STS
* Temporary Credentials
* Authentication
* Authorization
* AccessDenied
* Explicit Deny
* Implicit Deny
* Least Privilege
* EC2 → S3 Access
* IAM Troubleshooting
* Production Security

**Day 3 Interview Notes — Completed ✅**

