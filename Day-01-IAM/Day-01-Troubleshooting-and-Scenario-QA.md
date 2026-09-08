# Day 1 — IAM + S3 Basics

# Troubleshooting & Scenario-Based Interview Questions

---

## PART 1 — TROUBLESHOOTING QUESTIONS

### Q1. An IAM user cannot access S3. How would you troubleshoot it?

**Answer:**

First, I would identify the IAM user making the request.

Then I would check:

1. Policies attached directly to the user.
2. IAM groups and policies inherited from those groups.
3. Required S3 action.
4. Resource ARN.
5. S3 bucket policy.
6. Any explicit Deny.
7. Permissions boundary or SCP if applicable.

Then I would test the permission again.

---

### Q2. The IAM user can read an S3 object but cannot upload it. Why?

**Answer:**

Reading and uploading require different permissions.

Reading requires:

```text
s3:GetObject
```

Uploading requires:

```text
s3:PutObject
```

Therefore, the user can read an object without necessarily having permission to upload one.

---

### Q3. The user can upload an object but cannot delete it. What would you check?

**Answer:**

I would check whether the user has:

```text
s3:DeleteObject
```

permission.

If the policy contains `s3:PutObject` but not `s3:DeleteObject`, upload will work but deletion will fail with `AccessDenied`.

---

### Q4. The user can access one S3 bucket but not another. Why?

**Answer:**

The IAM policy may be restricted to a specific bucket.

For example:

```text
arn:aws:s3:::bucket-a/*
```

allows object access to bucket A, but does not automatically allow access to bucket B.

---

### Q5. The IAM policy contains Allow, but the request still returns AccessDenied. What do you check?

**Answer:**

I would check for an explicit Deny.

I would also check:

* S3 bucket policy
* Permissions boundary
* SCP
* Resource ARN
* Conditions
* Other applicable policies

An explicit Deny overrides an Allow.

---

### Q6. A user was added to an IAM group but still cannot access S3. What do you check?

**Answer:**

I would verify:

1. The user is actually a member of the group.
2. The correct policy is attached to the group.
3. The policy contains the required S3 action.
4. The resource ARN is correct.
5. There is no explicit Deny.

---

### Q7. S3 upload returns AccessDenied. What is your troubleshooting approach?

**Answer:**

I would identify:

```text
WHO → Which IAM identity?
WHAT → Which S3 action?
WHERE → Which S3 resource?
POLICY → Is the action allowed?
DENY → Is there an explicit Deny?
```

For an upload, I would specifically check:

```text
s3:PutObject
```

---

### Q8. S3 download returns AccessDenied. What would you check?

**Answer:**

I would check:

```text
s3:GetObject
```

I would also verify that the policy's resource matches the object ARN.

---

### Q9. S3 delete returns AccessDenied. What would you check?

**Answer:**

I would check whether:

```text
s3:DeleteObject
```

is allowed.

I would also check for an explicit Deny.

---

### Q10. IAM permissions worked yesterday but stopped working today. What would you investigate?

**Answer:**

I would investigate what changed.

I would check:

* IAM policies
* Group membership
* S3 bucket policy
* Explicit Deny
* Permissions boundary
* SCP
* Recent IAM changes

I could also use CloudTrail to investigate recent API or configuration changes.

---

## PART 2 — SCENARIO-BASED QUESTIONS

### Scenario 1. Application only needs to download files from S3. What permissions would you give?

**Answer:**

I would give only the required read permission:

```text
s3:GetObject
```

If the application also needs to list objects, I would consider:

```text
s3:ListBucket
```

I would not provide upload or delete permissions unless required.

---

### Scenario 2. Application needs to upload files but never delete them.

**Answer:**

I would grant:

```text
s3:PutObject
```

and avoid granting:

```text
s3:DeleteObject
```

This follows the principle of least privilege.

---

### Scenario 3. Developer asks for AmazonS3FullAccess because the application gets AccessDenied. What would you do?

**Answer:**

I would not immediately grant full S3 access.

First, I would identify the exact operation that failed.

For example, if upload failed, I would investigate:

```text
s3:PutObject
```

Then I would grant only the required permission for the required resource.

---

### Scenario 4. Your company has 100 S3 buckets, but an application should access only one. How would you design the policy?

**Answer:**

I would restrict the policy's Resource to the required bucket.

I would not give access to all S3 buckets.

This follows least privilege.

---

### Scenario 5. Developers should upload files but must never delete them.

**Answer:**

I would allow:

```text
s3:PutObject
```

but not:

```text
s3:DeleteObject
```

Then I would test both operations to confirm the intended behavior.

---

### Scenario 6. An employee says, "I have S3 access." What does that actually mean?

**Answer:**

"S3 access" is not specific enough.

I would determine:

* Which S3 actions?
* Which buckets?
* Which objects?
* Which conditions?

For example, a user may have read access but not write or delete access.

---

### Scenario 7. A user can upload to bucket A but cannot upload to bucket B.

**Answer:**

I would compare the policy resources.

The policy may allow:

```text
arn:aws:s3:::bucket-a/*
```

but not:

```text
arn:aws:s3:::bucket-b/*
```

Therefore, access to bucket A does not automatically mean access to bucket B.

---

### Scenario 8. Security asks you to follow least privilege. What would you do?

**Answer:**

I would identify the exact actions and resources required by the application and grant only those permissions.

I would avoid broad policies such as full administrator or full S3 access unless there is a genuine requirement.

---

## PART 3 — REAL TROUBLESHOOTING MINDSET

When an AWS permission problem occurs, I would think:

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
ALLOW?
    ↓
Is the action allowed?
    ↓
DENY?
    ↓
Is there an explicit Deny?
    ↓
TEST AGAIN
```

---

## PART 4 — MOST IMPORTANT DAY 1 QUESTIONS

1. What is IAM?
2. What is an IAM user?
3. What is an IAM group?
4. What is an IAM policy?
5. What is an IAM role?
6. What is least privilege?
7. What is Allow?
8. What is Deny?
9. What is explicit Deny?
10. Why can a user read but not upload?
11. Why can a user upload but not delete?
12. How do you troubleshoot AccessDenied?
13. Why should you avoid AdministratorAccess for normal users?
14. Why should you avoid AmazonS3FullAccess when only one action is required?
15. How do you restrict access to one S3 bucket?

---

## KEY INTERVIEW RULE

Never answer only:

> "I will check IAM."

Instead say:

> "First I would identify the IAM identity, then check the failed action, resource ARN, applicable policies and any explicit Deny. After that I would test the request again."

This demonstrates troubleshooting ability rather than just memorization.

---

## DAY 1 KEY TAKEAWAY

**IAM controls who can perform an action on which AWS resource.**

For S3 troubleshooting, always think:

```text
WHO → ACTION → RESOURCE → ALLOW/DENY
```

