# Day 2 — EC2 IAM Role → S3

# Troubleshooting & Scenario-Based Interview Questions

---

## PART 1 — TROUBLESHOOTING QUESTIONS

### Q1. EC2 cannot access S3 even though an IAM role exists. What do you check?

**Answer:**

First, I would verify that the IAM role is actually attached to the EC2 instance.

Then I would run:

```bash
aws sts get-caller-identity
```

This helps verify which identity the EC2 instance is using.

Then I would check the permissions attached to the role.

---

### Q2. How do you verify which IAM role EC2 is using?

**Answer:**

I can run:

```bash
aws sts get-caller-identity
```

The response should show an assumed-role ARN.

I would verify that the role name is the expected role.

---

### Q3. `aws sts get-caller-identity` shows a different role. What would you check?

**Answer:**

I would check:

1. IAM role attached to the EC2 instance.
2. Instance profile.
3. AWS CLI credential configuration.
4. Environment credentials.
5. Whether another credential source is being used.

---

### Q4. EC2 has an IAM role, but AWS CLI says credentials are unavailable. What would you check?

**Answer:**

I would verify that the IAM role is properly attached through the EC2 instance profile.

I would also check whether EC2 can obtain temporary credentials from the instance metadata service.

---

### Q5. EC2 can list S3 objects but cannot upload.

**Answer:**

I would check whether the role has:

```text
s3:PutObject
```

Listing and uploading are different S3 permissions.

---

### Q6. EC2 can download from S3 but cannot delete.

**Answer:**

I would check:

```text
s3:GetObject
s3:DeleteObject
```

The role may have `GetObject` without `DeleteObject`.

---

### Q7. EC2 can upload but cannot download.

**Answer:**

I would check whether:

```text
s3:GetObject
```

is allowed.

`PutObject` does not automatically provide `GetObject`.

---

### Q8. EC2 role permissions look correct but S3 returns AccessDenied. What would you investigate?

**Answer:**

I would check:

* IAM role
* IAM policy
* S3 bucket policy
* Resource ARN
* Explicit Deny
* Permissions boundary
* SCP
* KMS permissions if encryption is involved

---

### Q9. EC2 worked yesterday but cannot access S3 today.

**Answer:**

I would investigate recent changes.

I would check:

* EC2 IAM role
* IAM policy
* Instance profile
* S3 bucket policy
* Explicit Deny
* SCP
* KMS configuration
* Network configuration

CloudTrail can help identify changes made to AWS resources.

---

### Q10. How do you prove that an EC2 instance is using an IAM role rather than an IAM user?

**Answer:**

I would run:

```bash
aws sts get-caller-identity
```

If the ARN contains an assumed role, I can identify the role being used.

---

### Q11. Why can EC2 access S3 without storing an AWS access key in the server?

**Answer:**

Because EC2 can use an IAM role attached through an instance profile.

AWS provides temporary credentials associated with that role.

The AWS CLI or SDK can use those credentials automatically.

---

### Q12. EC2 has the correct role but cannot reach S3. How do you know whether it is an IAM or network problem?

**Answer:**

I would examine the error.

For example:

```text
AccessDenied
```

usually points toward authorization.

A:

```text
timeout
connection error
```

could indicate a networking problem.

I would troubleshoot IAM and networking separately.

---

## PART 2 — SCENARIO-BASED QUESTIONS

### Scenario 1. Your application on EC2 needs to upload files to S3. How would you give it access?

**Answer:**

I would create an IAM role for EC2 and attach a least-privilege policy that allows the required S3 action.

For upload, I would grant:

```text
s3:PutObject
```

for the required S3 resource.

I would avoid storing long-term access keys on the EC2 instance.

---

### Scenario 2. Why would you use an IAM role instead of access keys on EC2?

**Answer:**

IAM roles provide temporary credentials and avoid storing long-term access keys on the server.

This is a more secure approach for AWS workloads running on EC2.

---

### Scenario 3. You have 20 EC2 instances running the same application. Would you create 20 IAM users?

**Answer:**

No.

I would use an IAM role for the EC2 workload.

The same appropriate IAM role can be associated with the required EC2 instances.

---

### Scenario 4. Developer asks you to put an AWS access key in `/home/ec2-user/.aws/credentials`. What would you recommend?

**Answer:**

I would avoid storing long-term credentials on the EC2 instance.

I would recommend using an IAM role attached to EC2 and granting only the required permissions.

---

### Scenario 5. EC2 needs read-only access to S3.

**Answer:**

I would attach an IAM role with read-only permissions.

I would not provide upload or delete permissions unless required.

---

### Scenario 6. EC2 needs upload and download but must never delete.

**Answer:**

I would grant:

```text
s3:PutObject
s3:GetObject
```

but omit:

```text
s3:DeleteObject
```

This is least privilege.

---

### Scenario 7. The application suddenly starts using the wrong IAM role.

**Answer:**

I would first run:

```bash
aws sts get-caller-identity
```

Then I would inspect the EC2 instance's IAM role and instance profile.

I would also check whether environment or configuration credentials are overriding the expected credential source.

---

### Scenario 8. EC2 can access S3 from one subnet but not another.

**Answer:**

I would determine whether the problem is authorization or connectivity.

I would compare:

* IAM role
* Route tables
* NAT Gateway
* S3 VPC endpoint
* Security Groups
* NACLs

If one subnet has no network path to S3, IAM permissions alone will not solve the problem.

---

### Scenario 9. Private EC2 cannot access S3.

**Answer:**

I would check:

1. Route table.
2. NAT Gateway, if NAT is being used.
3. S3 VPC endpoint, if configured.
4. Security Group outbound rules.
5. NACL.
6. IAM permissions.

I would first determine whether the failure is an authorization failure or connectivity failure.

---

### Scenario 10. The application receives AccessDenied from S3. What would you do first?

**Answer:**

I would identify the IAM identity.

On EC2, I would run:

```bash
aws sts get-caller-identity
```

Then I would identify the failed S3 action and resource and check the role permissions.

---

### Scenario 11. The application receives a timeout while accessing S3.

**Answer:**

A timeout suggests I should investigate networking.

I would check:

```text
Route Table
↓
NAT Gateway / S3 VPC Endpoint
↓
Security Group
↓
NACL
```

I would also verify IAM after confirming connectivity.

---

### Scenario 12. Security asks you to remove all long-term AWS credentials from EC2.

**Answer:**

I would use IAM roles for EC2.

The application would obtain temporary credentials through the role instead of using permanent access keys.

---

## PART 3 — EC2 → S3 ACCESS FLOW

The basic flow is:

```text
EC2 Instance
      ↓
IAM Role
      ↓
Temporary Credentials
      ↓
AWS STS
      ↓
S3 API Request
      ↓
IAM Policy Evaluation
      ↓
S3
```

---

## PART 4 — TROUBLESHOOTING DECISION TREE

```text
EC2 cannot access S3
        ↓
Run:
aws sts get-caller-identity
        ↓
Correct role?
   ↓             ↓
 NO              YES
 ↓                ↓
Fix role      Check permissions
                  ↓
             Correct action?
                  ↓
             Correct resource?
                  ↓
             Explicit Deny?
                  ↓
             Bucket policy?
                  ↓
             Network issue?
                  ↓
               Test again
```

---

## PART 5 — IMPORTANT COMMANDS

### Check current AWS identity

```bash
aws sts get-caller-identity
```

### List buckets

```bash
aws s3 ls
```

### List a specific bucket

```bash
aws s3 ls s3://bucket-name
```

### Upload

```bash
aws s3 cp test.txt s3://bucket-name/
```

### Download

```bash
aws s3 cp s3://bucket-name/test.txt downloaded.txt
```

### Delete

```bash
aws s3 rm s3://bucket-name/test.txt
```

---

## PART 6 — MOST IMPORTANT DAY 2 QUESTIONS

1. What is an IAM role?
2. Why use IAM roles with EC2?
3. What is an instance profile?
4. How does EC2 obtain temporary credentials?
5. How do you verify the role used by EC2?
6. What does `aws sts get-caller-identity` do?
7. Why avoid access keys on EC2?
8. EC2 has a role but S3 returns AccessDenied. What do you check?
9. EC2 can download but cannot upload. Why?
10. EC2 can upload but cannot delete. Why?
11. Private EC2 cannot access S3. What do you check?
12. How do you distinguish IAM failure from network failure?
13. What happens if the wrong role is attached?
14. How would you secure EC2 → S3 in production?
15. Why is least privilege important for EC2 roles?

---

## DAY 2 KEY TAKEAWAY

When an EC2 application needs AWS access:

```text
EC2
 ↓
IAM ROLE
 ↓
TEMPORARY CREDENTIALS
 ↓
AWS SERVICE
```

Do not think:

> "EC2 needs an access key."

Think:

> "EC2 should use an IAM role with least-privilege permissions."

