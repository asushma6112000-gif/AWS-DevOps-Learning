# AWS IAM + S3 — Day 1 Mini Project

## Interview Preparation Notes

### 1. What is IAM?

IAM stands for **Identity and Access Management**.

AWS IAM is used to control:

* **Who can access AWS resources**
* **What actions they are allowed to perform**

Simple understanding:

```text
Authentication → Who are you?
Authorization  → What can you do?
```

---

### 2. IAM User

An IAM user represents an identity in AWS.

In my project, I created:

```text
iam-test-user
```

The user was used to test access to Amazon S3.

---

### 3. IAM Group

An IAM group is a collection of IAM users.

I created:

```text
s3-readers
```

Then I added:

```text
iam-test-user
       ↓
   s3-readers
```

The advantage of using a group is that permissions can be managed centrally for multiple users.

---

### 4. IAM Policy

An IAM policy is a JSON document that defines what actions are allowed or denied on AWS resources.

I attached the AWS-managed policy:

```text
AmazonS3ReadOnlyAccess
```

This provided read-only access to S3.

---

### 5. S3 Bucket

I created the following S3 bucket:

```text
sushma-iam-day1-test-2026
```

I uploaded a test file:

```text
test.txt
```

---

### 6. Complete Permission Flow

```text
IAM User
iam-test-user
      ↓
IAM Group
s3-readers
      ↓
IAM Policy
AmazonS3ReadOnlyAccess
      ↓
Amazon S3
sushma-iam-day1-test-2026
```

---

### 7. AWS CLI Configuration

I configured the AWS CLI using:

```bash
aws configure
```

I configured the AWS region as:

```text
ap-south-1
```

---

### 8. Verify IAM Identity

I used AWS STS to verify which identity was being used by the AWS CLI.

Command:

```bash
aws sts get-caller-identity
```

This confirmed that the AWS CLI was using the IAM user credentials.

---

### 9. Test S3 Read Access

I listed all S3 buckets:

```bash
aws s3 ls
```

Result:

```text
Allowed ✅
```

Then I listed objects inside my bucket:

```bash
aws s3 ls s3://sushma-iam-day1-test-2026
```

The object was displayed:

```text
test.txt
```

Result:

```text
Allowed ✅
```

This proved that the IAM user had read/list permissions.

---

### 10. Test S3 Upload Permission

I tried to upload a file:

```bash
aws s3 cp upload-test.txt s3://sushma-iam-day1-test-2026/
```

Result:

```text
AccessDenied ❌
```

The reason is that uploading an object requires:

```text
s3:PutObject
```

The read-only policy does not provide this permission.

Therefore:

```text
Upload → ❌ Denied
```

---

### 11. Test S3 Delete Permission

I tried to delete the test object:

```bash
aws s3 rm s3://sushma-iam-day1-test-2026/test.txt
```

Result:

```text
AccessDenied ❌
```

Deleting an object requires:

```text
s3:DeleteObject
```

The read-only policy does not provide this permission.

Therefore:

```text
Delete → ❌ Denied
```

---

### 12. Final Permission Testing

```text
S3 Operation             Result

List buckets              ✅ Allowed
List objects              ✅ Allowed
Read objects              ✅ Allowed
Upload objects            ❌ Denied
Delete objects            ❌ Denied
```

---

### 13. Principle of Least Privilege

The main security concept demonstrated in this project is:

**Principle of Least Privilege**

It means:

> Give a user only the permissions they actually need.

In my project, the user needed read access to S3, so I used a read-only policy instead of giving full S3 permissions.

```text
Read / List → ✅ Allowed
Upload      → ❌ Denied
Delete      → ❌ Denied
```

This reduces the risk of unauthorized changes or deletion.

---

# 🎯 Interview Explanation

If an interviewer asks:

### "Explain your IAM project."

I can say:

> "I created a hands-on AWS IAM and S3 access-control project. I created an IAM user called iam-test-user and added it to an IAM group called s3-readers. I attached the AmazonS3ReadOnlyAccess AWS-managed policy to the group. I then configured the AWS CLI and used AWS STS to verify the IAM identity. After that, I tested different S3 operations. Listing the bucket and objects was allowed, but uploading and deleting objects returned AccessDenied because the user did not have s3:PutObject and s3:DeleteObject permissions. This project helped me understand IAM users, groups, policies, AWS CLI, STS, S3 permissions, and the Principle of Least Privilege."

---

# ⭐ Important Interview Points

### IAM

```text
IAM = Identity and Access Management
```

### Authentication

```text
Authentication = Who are you?
```

### Authorization

```text
Authorization = What are you allowed to do?
```

### User

```text
IAM User = Identity
```

### Group

```text
IAM Group = Collection of users
```

### Policy

```text
IAM Policy = Defines permissions
```

### Least Privilege

```text
Give only required permissions.
```

### AccessDenied

```text
AccessDenied means the request was not authorized
by the applicable permissions.
```

### S3 Upload

```text
s3:PutObject
```

### S3 Delete

```text
s3:DeleteObject
```

### STS

```text
AWS STS can provide temporary security credentials
and is also used to retrieve caller identity.
```

---

# 📌 Project Summary

```text
Project:
AWS IAM + S3 Access Control

IAM User:
iam-test-user

IAM Group:
s3-readers

IAM Policy:
AmazonS3ReadOnlyAccess

S3 Bucket:
sushma-iam-day1-test-2026

Region:
ap-south-1

Tools:
AWS CLI + AWS STS

Main Security Concept:
Principle of Least Privilege
```

### Project Status

```text
Day 1 → Completed ✅
IAM + S3 → Completed ✅
AWS CLI → Practiced ✅
STS → Practiced ✅
Permission Testing → Completed ✅
Least Privilege → Understood ✅
```

