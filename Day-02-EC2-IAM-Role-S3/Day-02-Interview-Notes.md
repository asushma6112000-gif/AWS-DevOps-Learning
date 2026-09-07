# Day 2 — EC2 IAM Role + S3

## Interview Questions & Answers

---

# 1. Day 2 Project Overview

### Q1. What did you learn in Day 2?

**Answer:**

In Day 2, I learned how to securely allow an EC2 instance to access an S3 bucket using an IAM Role.

I learned:

* IAM Roles
* Trust Policies
* Permissions Policies
* IAM Policies
* EC2 IAM Role attachment
* Amazon S3
* AWS CLI
* AWS STS
* Temporary credentials
* Principle of Least Privilege
* Testing allowed and denied permissions
* EC2-to-S3 access

My main project flow was:

```text
EC2
 ↓
IAM Role
 ↓
IAM Permission Policy
 ↓
Amazon S3
```

---

# 2. What is IAM?

### Q2. What is IAM?

**Answer:**

IAM stands for **Identity and Access Management**.

It is an AWS service used to control:

* Who can access AWS resources
* What actions they can perform
* Which resources they can access

IAM mainly includes:

```text
IAM
├── Users
├── Groups
├── Roles
└── Policies
```

---

# 3. What is an IAM Role?

### Q3. What is an IAM Role?

**Answer:**

An IAM Role is an AWS identity that provides permissions to a trusted entity such as EC2, Lambda, or another AWS service.

In my project, I created:

```text
EC2-S3-ReadOnly-Role
```

I attached this role to my EC2 instance so that EC2 could access S3 without storing long-term AWS access keys on the server.

---

# 4. Why did you use an IAM Role?

### Q4. Why did you use an IAM Role instead of an access key?

**Answer:**

I used an IAM Role because it is more secure.

If I store an AWS access key and secret access key on an EC2 instance, those long-term credentials could be exposed if the server is compromised.

With an IAM Role, EC2 can obtain temporary credentials automatically.

Therefore:

```text
Access Keys
→ Long-term credentials
→ Need to manage securely

IAM Role
→ Temporary credentials
→ Automatically available to EC2
→ More secure
```

---

# 5. What was your IAM Role?

### Q5. What IAM Role did you create?

**Answer:**

I created:

```text
EC2-S3-ReadOnly-Role
```

The role was configured so that EC2 could assume the role.

I attached:

```text
AmazonS3ReadOnlyAccess
```

to the role.

---

# 6. What is a Trust Policy?

### Q6. What is a Trust Policy?

**Answer:**

A Trust Policy defines **who is allowed to assume an IAM Role**.

In my project, the trusted entity was EC2.

So:

```text
Trust Policy
      ↓
EC2 is allowed to assume the role
```

An easy way to remember:

```text
Trust Policy = WHO
```

---

# 7. What is a Permissions Policy?

### Q7. What is a Permissions Policy?

**Answer:**

A Permissions Policy defines **what actions the identity is allowed or denied to perform on AWS resources**.

In my project, I attached:

```text
AmazonS3ReadOnlyAccess
```

This allowed the EC2 role to perform read-related operations on S3.

Easy way to remember:

```text
Permissions Policy = WHAT
```

---

# 8. Trust Policy vs Permissions Policy

### Q8. What is the difference between a Trust Policy and a Permissions Policy?

**Answer:**

The Trust Policy determines **who can assume the role**.

The Permissions Policy determines **what the role can do after it is assumed**.

Example from my project:

```text
Trust Policy:
EC2 → allowed to assume role

Permissions Policy:
Role → allowed to read S3
```

Simple memory trick:

```text
Trust = WHO
Permissions = WHAT
```

---

# 9. What is the Principle of Least Privilege?

### Q9. What is the Principle of Least Privilege?

**Answer:**

The Principle of Least Privilege means giving an identity only the permissions required to perform its job and no unnecessary permissions.

In my project, EC2 only needed to read objects from S3.

Therefore:

```text
Read     → Allowed
Upload   → Denied
Delete   → Denied
```

This reduced unnecessary access.

---

# 10. What is S3?

### Q10. What is Amazon S3?

**Answer:**

S3 stands for **Simple Storage Service**.

It is an AWS object storage service used to store and retrieve data such as:

* Images
* Videos
* Documents
* Backups
* Logs
* Application files

---

# 11. What is an S3 Bucket?

### Q11. What is an S3 Bucket?

**Answer:**

An S3 bucket is a container used to store objects.

In my project, I created the bucket:

```text
sushma-day2-ec2-s3-2026-01
```

Inside the bucket, I stored:

```text
test.txt
```

---

# 12. What is an S3 Object?

### Q12. What is an S3 Object?

**Answer:**

An object is the actual data stored inside an S3 bucket.

For example:

```text
Bucket
 ↓
test.txt
```

Here:

```text
Bucket = container
test.txt = object
```

---

# 13. What was your EC2 instance?

### Q13. What EC2 instance did you use?

**Answer:**

I created an EC2 instance for the Day 2 lab:

```text
Name: day2-ec2-s3-test
```

I used Amazon Linux 2023 and attached the IAM Role:

```text
EC2-S3-ReadOnly-Role
```

---

# 14. How does EC2 access S3?

### Q14. Explain how EC2 accessed S3 in your project.

**Answer:**

The EC2 instance had an IAM Role attached to it.

The role had S3 read permissions.

When I ran an AWS CLI command from EC2, AWS used the role credentials to authenticate the request.

The flow was:

```text
EC2
 ↓
Attached IAM Role
 ↓
Temporary Credentials
 ↓
AWS API Request
 ↓
IAM Permission Evaluation
 ↓
Amazon S3
```

If the requested action was allowed by the IAM policy, S3 returned the result.

---

# 15. Did you store AWS access keys on EC2?

### Q15. Did you configure an AWS access key and secret key on the EC2 instance?

**Answer:**

No.

I did not store long-term AWS access keys on the EC2 instance.

Instead, I attached an IAM Role to the EC2 instance.

The EC2 instance obtained temporary credentials through the role.

This follows AWS security best practices.

---

# 16. What is AWS STS?

### Q16. What is AWS STS?

**Answer:**

STS stands for **AWS Security Token Service**.

It provides temporary security credentials that can be used to access AWS resources.

IAM Roles commonly use temporary credentials rather than long-term access keys.

---

# 17. How did you verify the IAM Role?

### Q17. How did you verify which AWS identity EC2 was using?

**Answer:**

I ran:

```bash
aws sts get-caller-identity
```

This command returned information about the AWS identity being used by the AWS CLI.

It helped me verify that the EC2 instance was using the IAM Role instead of manually configured access keys.

---

# 18. What does `aws sts get-caller-identity` do?

### Q18. What does `aws sts get-caller-identity` do?

**Answer:**

It returns information about the IAM identity that is making the AWS API request.

It can show information such as:

```text
Account
UserId
Arn
```

I used it to verify the identity being used by my EC2 instance.

---

# 19. How did you check the S3 bucket?

### Q19. Which AWS CLI command did you use to list S3 buckets?

**Answer:**

I used:

```bash
aws s3 ls
```

This lists the S3 buckets that the current AWS identity has permission to access.

---

# 20. How did you check objects inside the bucket?

### Q20. Which command did you use to list objects inside your S3 bucket?

**Answer:**

I used:

```bash
aws s3 ls s3://sushma-day2-ec2-s3-2026-01
```

This successfully showed:

```text
test.txt
```

This confirmed that the EC2 role had permission to list/read the bucket contents.

---

# 21. How did you download the S3 object?

### Q21. How did you download `test.txt` from S3?

**Answer:**

I used:

```bash
aws s3 cp s3://sushma-day2-ec2-s3-2026-01/test.txt .
```

The command successfully downloaded the file from S3 to the EC2 instance.

I then verified the contents using:

```bash
cat test.txt
```

The output was:

```text
Day 2 EC2 IAM ROLE S3 Test
```

---

# 22. How did you test upload permission?

### Q22. How did you verify that EC2 could not upload objects to S3?

**Answer:**

I created an upload test file and attempted to upload it:

```bash
aws s3 cp upload-test.txt s3://sushma-day2-ec2-s3-2026-01/
```

The request returned:

```text
AccessDenied
```

The reason was that the IAM Role did not have the required S3 upload permission.

The relevant permission would be:

```text
s3:PutObject
```

Therefore:

```text
Upload → DENIED
```

---

# 23. How did you test delete permission?

### Q23. How did you verify that EC2 could not delete the S3 object?

**Answer:**

I attempted to delete `test.txt`:

```bash
aws s3 rm s3://sushma-day2-ec2-s3-2026-01/test.txt
```

The request returned:

```text
AccessDenied
```

The role did not have the required:

```text
s3:DeleteObject
```

permission.

Therefore:

```text
Delete → DENIED
```

---

# 24. Why were upload and delete denied?

### Q24. Why did upload and delete fail?

**Answer:**

They failed because the IAM Role was configured for read-only S3 access.

The role did not have permissions such as:

```text
s3:PutObject
s3:DeleteObject
```

Therefore AWS denied those requests.

This demonstrated that the IAM policy was working as expected.

---

# 25. Why is AccessDenied a good result in this lab?

### Q25. Isn't AccessDenied an error?

**Answer:**

In this test, AccessDenied was expected and actually confirmed that the security configuration was working correctly.

My requirement was:

```text
Read → Allowed
Write → Denied
Delete → Denied
```

The results matched the intended permissions.

Therefore, the AccessDenied responses demonstrated that the Principle of Least Privilege was working.

---

# 26. What commands did you use in the lab?

### Q26. What important AWS CLI commands did you use?

**Answer:**

I used:

### Check current AWS identity

```bash
aws sts get-caller-identity
```

### List S3 buckets

```bash
aws s3 ls
```

### List objects in a bucket

```bash
aws s3 ls s3://BUCKET-NAME
```

### Download an object

```bash
aws s3 cp s3://BUCKET-NAME/test.txt .
```

### Test upload

```bash
aws s3 cp upload-test.txt s3://BUCKET-NAME/
```

### Test deletion

```bash
aws s3 rm s3://BUCKET-NAME/test.txt
```

### Verify downloaded file

```bash
cat test.txt
```

---

# 27. What happened when you ran `aws s3 ls`?

### Q27. What happened when you ran `aws s3 ls` from EC2?

**Answer:**

The command successfully listed the S3 bucket because the EC2 instance had an IAM Role with appropriate S3 permissions.

This demonstrated that EC2 could authenticate to AWS without manually configuring access keys.

---

# 28. What happens if the IAM Role is removed from EC2?

### Q28. What happens if you remove the IAM Role from EC2?

**Answer:**

The EC2 instance would no longer have the role credentials available through its instance role.

AWS CLI commands requiring those permissions would fail unless another valid credential source was configured.

For example, S3 operations could return an authentication or authorization error.

---

# 29. What happens if the IAM policy is removed?

### Q29. What happens if you remove the S3 permissions from the role?

**Answer:**

The EC2 instance may still have the role attached, but it would no longer have permission to perform the S3 actions that were removed.

For example:

```text
Role exists
      ↓
No S3 read permission
      ↓
S3 read request
      ↓
AccessDenied
```

---

# 30. What happens if you give the role AdministratorAccess?

### Q30. Why shouldn't you simply give EC2 AdministratorAccess?

**Answer:**

AdministratorAccess provides extremely broad permissions.

It would violate the Principle of Least Privilege for a workload that only needs to read S3 objects.

Instead, I should provide only the permissions required by the application.

In my lab, I used:

```text
AmazonS3ReadOnlyAccess
```

because the requirement was read-only S3 access.

---

# 31. What is the difference between IAM User and IAM Role?

### Q31. What is the difference between an IAM User and an IAM Role?

**Answer:**

An IAM User generally represents a person or application identity that can have long-term credentials.

An IAM Role is intended to be assumed by trusted entities and provides temporary credentials.

For example:

```text
IAM User
→ Human identity

IAM Role
→ EC2 / Lambda / AWS service / trusted entity
```

In my project, I used an IAM Role because EC2 needed AWS permissions.

---

# 32. What is the difference between Role and Policy?

### Q32. What is the difference between an IAM Role and an IAM Policy?

**Answer:**

A Role is an identity that can be assumed by a trusted entity.

A Policy is a document that defines permissions.

In my project:

```text
EC2
 ↓
IAM Role
 ↓
S3 Permission Policy
```

The role provides the identity, while the policy defines what it can do.

---

# 33. Why did you attach the role to EC2?

### Q33. Why must the IAM Role be attached to EC2?

**Answer:**

The role must be associated with the EC2 instance so that applications and AWS CLI commands running on the instance can obtain temporary credentials associated with that role.

This allows the instance to make authorized AWS API calls.

---

# 34. Explain the complete request flow.

### Q34. Explain what happens when you run the S3 command from EC2.

**Answer:**

Suppose I run:

```bash
aws s3 ls s3://sushma-day2-ec2-s3-2026-01
```

The flow is:

```text
1. AWS CLI runs on EC2
          ↓
2. CLI obtains credentials from the EC2 IAM Role
          ↓
3. CLI sends an authenticated request to AWS
          ↓
4. AWS evaluates the IAM permissions
          ↓
5. The role allows the required S3 action
          ↓
6. S3 returns the bucket/object information
```

If the action is not allowed:

```text
IAM permission check
        ↓
AccessDenied
```

---

# 35. What security concept did you demonstrate?

### Q35. What security principle did your project demonstrate?

**Answer:**

The main security principle was **Least Privilege**.

I gave EC2 only the S3 permissions required for the lab.

I verified:

```text
Read     → Allowed
Upload   → Denied
Delete   → Denied
```

This demonstrated controlled access rather than giving EC2 unrestricted permissions.

---

# 36. What was the purpose of the `test.txt` file?

### Q36. Why did you create `test.txt`?

**Answer:**

I created `test.txt` as a test object in the S3 bucket.

It allowed me to verify that EC2 could:

```text
1. Find the object
2. Read the object
3. Download the object
4. Verify its contents
```

I also attempted unauthorized operations against the bucket to test the IAM restrictions.

---

# 37. How would you improve this project in production?

### Q37. If this were a production environment, what would you improve?

**Answer:**

I would follow stricter least-privilege practices.

Instead of using a broad managed policy such as:

```text
AmazonS3ReadOnlyAccess
```

I could create a customer-managed or inline policy allowing only the required actions on a specific S3 bucket or specific object paths.

For example, if the application only needs to read from one bucket, I would restrict access to that bucket rather than all S3 resources.

I would also:

* Enable S3 Block Public Access
* Enable appropriate S3 encryption
* Enable CloudTrail monitoring where required
* Use IAM Roles instead of long-term access keys
* Restrict permissions to only required resources and actions

---

# 38. What did you learn from the denied tests?

### Q38. What did the upload and delete tests teach you?

**Answer:**

They showed me that IAM is not just about granting access; it is also about restricting unauthorized actions.

The tests proved:

```text
IAM Role
   ↓
Read permission
   ↓
Allowed

IAM Role
   ↓
Write/Delete not permitted
   ↓
AccessDenied
```

This helped me understand how IAM permissions are enforced in AWS.

---

# 39. Explain your project in 30 seconds.

### Q39. Give a short explanation of your Day 2 project.

**Answer:**

> In Day 2, I created an EC2 instance and an S3 bucket and securely connected them using an IAM Role. I created an `EC2-S3-ReadOnly-Role`, configured EC2 as the trusted entity, and attached S3 read-only permissions. I verified the role using `aws sts get-caller-identity`, successfully listed and downloaded an S3 object, and then tested upload and delete operations. Both were denied with AccessDenied, proving that the least-privilege permissions were working correctly.

---

# 40. Explain your project step by step.

### Q40. Tell me the complete implementation process.

**Answer:**

### Step 1 — Create S3 Bucket

I created an S3 bucket:

```text
sushma-day2-ec2-s3-2026-01
```

and uploaded:

```text
test.txt
```

---

### Step 2 — Create IAM Role

I created:

```text
EC2-S3-ReadOnly-Role
```

and selected EC2 as the trusted entity.

---

### Step 3 — Attach Permission

I attached:

```text
AmazonS3ReadOnlyAccess
```

to the role.

---

### Step 4 — Launch EC2

I launched:

```text
day2-ec2-s3-test
```

using Amazon Linux 2023.

---

### Step 5 — Attach IAM Role

I attached:

```text
EC2-S3-ReadOnly-Role
```

to the EC2 instance.

---

### Step 6 — Connect to EC2

I connected to the EC2 instance using EC2 Instance Connect.

---

### Step 7 — Verify Identity

I ran:

```bash
aws sts get-caller-identity
```

to verify the AWS identity being used.

---

### Step 8 — Test S3 Access

I ran:

```bash
aws s3 ls
```

and:

```bash
aws s3 ls s3://sushma-day2-ec2-s3-2026-01
```

The bucket and object were accessible.

---

### Step 9 — Download Object

I ran:

```bash
aws s3 cp s3://sushma-day2-ec2-s3-2026-01/test.txt .
```

The download succeeded.

---

### Step 10 — Verify File

I ran:

```bash
cat test.txt
```

and verified the expected contents.

---

### Step 11 — Test Unauthorized Upload

I attempted:

```bash
aws s3 cp upload-test.txt s3://sushma-day2-ec2-s3-2026-01/
```

The operation returned:

```text
AccessDenied
```

---

### Step 12 — Test Unauthorized Delete

I attempted:

```bash
aws s3 rm s3://sushma-day2-ec2-s3-2026-01/test.txt
```

The operation returned:

```text
AccessDenied
```

---

### Step 13 — Final Verification

The original object remained in S3.

Therefore:

```text
Read     → SUCCESS
Upload   → DENIED
Delete   → DENIED
```

The IAM Role was working as intended.

---

# 41. What are the key takeaways from Day 2?

### Q41. What are your key learnings?

**Answer:**

My main learnings were:

1. IAM controls access to AWS resources.
2. IAM Roles provide permissions to trusted entities.
3. EC2 can use IAM Roles to access AWS services.
4. IAM Roles avoid storing long-term access keys on EC2.
5. Trust Policies define who can assume a role.
6. Permission Policies define what actions are allowed.
7. S3 is AWS object storage.
8. AWS CLI can interact with S3 from EC2.
9. STS provides temporary security credentials.
10. Least Privilege means granting only required permissions.
11. AccessDenied can be an expected result when testing restricted permissions.
12. IAM permissions should be tested with both allowed and unauthorized operations.

---

# 42. Important Interview Memory Map

Remember this:

```text
                    DAY 2
                      │
          EC2 + IAM ROLE + S3
                      │
        ┌─────────────┴─────────────┐
        │                           │
    IAM ROLE                       S3
        │                           │
   ┌────┴────┐                 Bucket/Object
   │         │
Trust     Permissions
Policy      Policy
   │         │
WHO        WHAT
   │         │
EC2       Read S3
             │
             ▼
       Temporary Credentials
             │
             ▼
            STS
             │
             ▼
          AWS CLI
             │
             ▼
             S3
```

---

# 43. Most Important Interview Questions

Before attending an interview, make sure you can explain these without looking at your notes:

```text
1. What is IAM?
2. What is an IAM Role?
3. Why use an IAM Role instead of access keys?
4. What is a Trust Policy?
5. What is a Permissions Policy?
6. Trust Policy vs Permissions Policy?
7. What is Least Privilege?
8. What is S3?
9. What is an S3 Bucket?
10. What is an S3 Object?
11. How does EC2 access S3?
12. What is STS?
13. What does aws sts get-caller-identity do?
14. How did you verify the IAM Role?
15. How did you test S3 read access?
16. Why did upload return AccessDenied?
17. Why did delete return AccessDenied?
18. What is s3:PutObject?
19. What is s3:DeleteObject?
20. IAM User vs IAM Role?
21. IAM Role vs IAM Policy?
22. Explain your complete Day 2 project.
23. Explain your project in 30 seconds.
24. How would you improve this project for production?
25. What did you learn from this project?
```

---

# Final Day 2 Summary

The most important concept I learned is:

```text
EC2
 ↓
IAM Role
 ↓
Temporary Credentials
 ↓
IAM Permission Evaluation
 ↓
S3
```

My project demonstrated secure AWS service access without storing long-term credentials on EC2.

I also demonstrated the Principle of Least Privilege by allowing S3 read operations while preventing unauthorized upload and delete operations.

## Day 2 Status

**Completed — EC2 IAM Role + S3 Hands-on Lab**

Key skills demonstrated:

* EC2
* IAM Roles
* IAM Policies
* Trust Policies
* S3
* AWS CLI
* STS
* Temporary Credentials
* Least Privilege
* Access Testing
* AWS Security Fundamentals

