# Day 3 — IAM Custom Policy + Least Privilege

# Troubleshooting & Scenario-Based Interview Questions

---

## PART 1 — TROUBLESHOOTING QUESTIONS

### Q1. Why does `aws s3 ls` return AccessDenied in the Day 3 lab?

**Answer:**

Because the custom policy does not grant:

```text
s3:ListAllMyBuckets
```

The policy grants:

```text
s3:ListBucket
```

for the specific bucket.

Therefore, listing all buckets is denied.

---

### Q2. Why does this command work?

```bash
aws s3 ls s3://sushma-day3-custom-policy-2026
```

**Answer:**

Because this operation requires permission to list that specific bucket:

```text
s3:ListBucket
```

The custom policy grants that permission for the specific bucket.

---

### Q3. Upload works but delete fails. Why?

**Answer:**

The policy contains:

```text
s3:PutObject
```

but does not contain:

```text
s3:DeleteObject
```

Therefore:

```text
Upload → Allowed
Delete → Denied
```

This demonstrates least privilege.

---

### Q4. Download works but upload fails. What does that indicate?

**Answer:**

It indicates that `s3:GetObject` may be allowed while `s3:PutObject` is not.

I would verify the policy and resource ARN.

---

### Q5. Specific bucket access works but other buckets do not. Why?

**Answer:**

Because the custom policy restricts the Resource to a specific S3 bucket.

This prevents unnecessary access to other buckets.

---

### Q6. What happens if the bucket ARN is incorrect?

**Answer:**

The policy may not match the requested resource.

As a result, the request can fail with `AccessDenied`.

I would verify the exact bucket ARN.

---

### Q7. What happens if you use the bucket ARN for `GetObject`?

**Answer:**

`GetObject` is an object-level action.

Therefore, I would use an object resource such as:

```text
arn:aws:s3:::bucket-name/*
```

rather than only:

```text
arn:aws:s3:::bucket-name
```

---

### Q8. What does `/*` mean in an S3 ARN?

**Answer:**

It represents objects inside the bucket.

For example:

```text
arn:aws:s3:::my-bucket/*
```

represents objects within `my-bucket`.

---

### Q9. Why does `s3:ListBucket` use the bucket ARN?

**Answer:**

Because `ListBucket` is a bucket-level action.

Example:

```text
arn:aws:s3:::my-bucket
```

---

### Q10. Why does `GetObject` use an object ARN?

**Answer:**

Because `GetObject` operates on S3 objects.

Example:

```text
arn:aws:s3:::my-bucket/*
```

---

### Q11. The custom policy looks correct but EC2 gets AccessDenied. What do you check?

**Answer:**

I would check:

1. Correct IAM role attached to EC2.
2. Correct policy attached to the role.
3. Correct Action.
4. Correct Resource.
5. Bucket policy.
6. Explicit Deny.
7. Permissions boundary.
8. SCP.
9. KMS permissions if encryption is involved.

---

### Q12. How would you troubleshoot a custom IAM policy?

**Answer:**

I would break the problem into:

```text
WHO?
↓
IAM identity

WHAT?
↓
Requested AWS action

WHERE?
↓
Requested resource

POLICY?
↓
Allow/Deny

CONDITION?
↓
Does the condition match?

OTHER DENY?
↓
Bucket policy / SCP / boundary
```

Then I would test again.

---

### Q13. The policy contains the correct Action but the wrong Resource. What happens?

**Answer:**

The policy will not authorize the requested resource.

For example, if `PutObject` is allowed only for bucket A but the application uploads to bucket B, the request can be denied.

---

### Q14. The policy contains the correct Resource but the wrong Action. What happens?

**Answer:**

The requested operation will not be allowed.

For example:

```text
s3:GetObject
```

does not automatically allow:

```text
s3:PutObject
```

---

## PART 2 — SCENARIO-BASED QUESTIONS

### Scenario 1. Explain your Day 3 project to an interviewer.

**Answer:**

In Day 3, I created a custom IAM policy for an EC2 instance to access a specific S3 bucket.

The policy allowed:

```text
s3:ListBucket
s3:GetObject
s3:PutObject
```

for the required bucket and objects.

I intentionally did not allow:

```text
s3:DeleteObject
```

and I did not grant permission to list all buckets.

This demonstrated custom IAM policies and the principle of least privilege.

---

### Scenario 2. Why did you create a custom policy instead of using AmazonS3FullAccess?

**Answer:**

Because I wanted to follow least privilege.

`AmazonS3FullAccess` provides much broader access than the application requires.

My custom policy grants only the required actions against the required resource.

---

### Scenario 3. An application needs only upload access to one S3 bucket. What would you give it?

**Answer:**

I would grant:

```text
s3:PutObject
```

and restrict the Resource to the required bucket/object path.

I would not grant full S3 access.

---

### Scenario 4. Application needs upload and download but never delete.

**Answer:**

I would grant:

```text
s3:PutObject
s3:GetObject
```

and omit:

```text
s3:DeleteObject
```

---

### Scenario 5. Application should access one bucket but cannot list all buckets.

**Answer:**

I would allow:

```text
s3:ListBucket
```

for the specific bucket.

I would not grant:

```text
s3:ListAllMyBuckets
```

unless there is a real requirement.

---

### Scenario 6. An interviewer asks: "Why does `aws s3 ls` fail but `aws s3 ls s3://bucket-name` work?"

**Answer:**

Because they require different permissions.

The command:

```bash
aws s3 ls
```

lists buckets and requires the ability to list all buckets.

The command:

```bash
aws s3 ls s3://bucket-name
```

lists objects within a specific bucket and uses:

```text
s3:ListBucket
```

So the second command can work even when the first one is denied.

---

### Scenario 7. Security asks you to prevent developers from deleting S3 objects.

**Answer:**

I would create a policy that allows the required read/write actions but does not grant:

```text
s3:DeleteObject
```

I would test deletion to confirm that it is denied.

---

### Scenario 8. Developer asks for full S3 permissions.

**Answer:**

I would first understand the application's actual requirements.

Then I would identify the exact actions and resources needed and create a least-privilege policy.

I would avoid giving full S3 access simply to fix an AccessDenied error.

---

### Scenario 9. Your custom policy allows `PutObject`, but upload still fails.

**Answer:**

I would check:

1. Correct IAM role.
2. Correct policy attachment.
3. Exact bucket/object ARN.
4. Bucket policy.
5. Explicit Deny.
6. Permissions boundary.
7. SCP.
8. KMS permissions if the object uses SSE-KMS.
9. Conditions in the policy.

---

### Scenario 10. EC2 can access S3 but only from one region. How could you restrict this?

**Answer:**

I could use IAM policy conditions such as an AWS global condition key appropriate for the requirement, for example restricting requests based on AWS region where supported.

The exact condition should be tested carefully because S3 API operations and AWS global condition keys have operation-specific behavior.

---

### Scenario 11. How would you prove your Day 3 policy is actually least privilege?

**Answer:**

I would test both allowed and denied operations.

For example:

```text
List specific bucket → SUCCESS
Upload object → SUCCESS
Download object → SUCCESS
Delete object → ACCESS DENIED
List all buckets → ACCESS DENIED
```

The denied operations demonstrate that unnecessary permissions were not granted.

---

### Scenario 12. Your manager says: "Just attach AdministratorAccess so the application works." What would you say?

**Answer:**

I would explain that AdministratorAccess is unnecessarily broad.

I would first identify the exact permission required and grant only that permission.

This reduces security risk and follows least privilege.

---

## PART 3 — DAY 3 REAL TESTING FLOW

The Day 3 test can be explained like this:

```text
EC2
 ↓
IAM Role
 ↓
Custom IAM Policy
 ↓
S3
```

### Step 1 — Verify identity

```bash
aws sts get-caller-identity
```

Expected:

```text
assumed-role/Day3-EC2-S3-Custom-Role/...
```

---

### Step 2 — Test listing all buckets

```bash
aws s3 ls
```

Expected:

```text
AccessDenied
```

Reason:

```text
s3:ListAllMyBuckets
```

is not allowed.

---

### Step 3 — Test specific bucket

```bash
aws s3 ls s3://sushma-day3-custom-policy-2026
```

Expected:

```text
SUCCESS
```

Reason:

```text
s3:ListBucket
```

is allowed.

---

### Step 4 — Test upload

```bash
echo "Day 3 Custom IAM Policy Test" > test.txt
aws s3 cp test.txt s3://sushma-day3-custom-policy-2026/
```

Expected:

```text
SUCCESS
```

Reason:

```text
s3:PutObject
```

is allowed.

---

### Step 5 — Test download

```bash
aws s3 cp s3://sushma-day3-custom-policy-2026/test.txt downloaded.txt
```

Expected:

```text
SUCCESS
```

Reason:

```text
s3:GetObject
```

is allowed.

---

### Step 6 — Verify content

```bash
cat downloaded.txt
```

Expected:

```text
Day 3 Custom IAM Policy Test
```

---

### Step 7 — Test delete

```bash
aws s3 rm s3://sushma-day3-custom-policy-2026/test.txt
```

Expected:

```text
AccessDenied
```

Reason:

```text
s3:DeleteObject
```

was intentionally not granted.

---

### Step 8 — Confirm object still exists

```bash
aws s3 ls s3://sushma-day3-custom-policy-2026
```

The object should still be present.

---

## PART 4 — IMPORTANT POLICY CONCEPTS FOR TROUBLESHOOTING

### `s3:ListBucket`

Bucket-level permission.

Example resource:

```text
arn:aws:s3:::my-bucket
```

---

### `s3:ListAllMyBuckets`

Permission to list buckets available to the identity.

This is why:

```bash
aws s3 ls
```

can fail even though access to a specific bucket works.

---

### `s3:GetObject`

Allows reading/downloading an object.

Example:

```text
arn:aws:s3:::my-bucket/*
```

---

### `s3:PutObject`

Allows uploading an object.

Example:

```text
arn:aws:s3:::my-bucket/*
```

---

### `s3:DeleteObject`

Allows deleting an object.

It must be explicitly allowed if the application needs deletion.

---

## PART 5 — BUCKET ARN VS OBJECT ARN

### Bucket ARN

```text
arn:aws:s3:::my-bucket
```

Used for bucket-level actions such as:

```text
s3:ListBucket
```

### Object ARN

```text
arn:aws:s3:::my-bucket/*
```

Used for object-level actions such as:

```text
s3:GetObject
s3:PutObject
s3:DeleteObject
```

---

## PART 6 — MOST IMPORTANT DAY 3 QUESTIONS

1. What is a custom IAM policy?
2. Why create a custom policy?
3. What is least privilege?
4. Why not use AmazonS3FullAccess?
5. What is `s3:ListBucket`?
6. What is `s3:ListAllMyBuckets`?
7. What is `s3:GetObject`?
8. What is `s3:PutObject`?
9. What is `s3:DeleteObject`?
10. Why does `aws s3 ls` fail?
11. Why does specific bucket listing work?
12. Bucket ARN vs object ARN?
13. What does `/*` mean?
14. Why can upload work while delete fails?
15. How do you troubleshoot AccessDenied?
16. What is an explicit Deny?
17. Why is resource restriction important?
18. How do you prove least privilege?
19. Why use IAM roles instead of access keys?
20. How would you secure EC2 → S3 in production?

---

## PART 7 — MASTER TROUBLESHOOTING FORMULA

For any IAM/S3 AccessDenied problem, think:

```text
                PROBLEM
                   ↓
                 WHO?
                   ↓
          IAM User / IAM Role
                   ↓
                 WHAT?
                   ↓
          AWS API / S3 Action
                   ↓
                WHERE?
                   ↓
          Bucket / Object ARN
                   ↓
              IS IT ALLOWED?
                   ↓
             Explicit Deny?
                   ↓
        Bucket Policy / SCP /
        Permissions Boundary
                   ↓
               NETWORK?
                   ↓
                TEST
```

---

## PART 8 — HOW TO ANSWER SCENARIO QUESTIONS

Do not simply say:

> "I will check IAM."

Use this structure:

```text
1. What I would check
2. Why I would check it
3. What result I expect
4. What I would do next
```

### Example

**Interviewer:**

> EC2 cannot upload to S3. What will you do?

**Strong answer:**

> First, I would verify which IAM identity the EC2 instance is using with `aws sts get-caller-identity`. Then I would check whether the attached role allows `s3:PutObject` on the correct S3 object ARN. I would also check the bucket policy and any explicit Deny. If authorization looks correct but the request times out, I would investigate networking such as NAT Gateway or an S3 VPC endpoint. Finally, I would test the upload again.

This answer shows actual troubleshooting thinking.

---

# DAY 3 FINAL TAKEAWAY

The goal of IAM troubleshooting is not:

> "Give more permissions until it works."

The goal is:

> **Find the exact identity, action, resource and restriction causing the failure, then grant only what is required.**

That is the AWS **Principle of Least Privilege**.

