# Day 7 — KMS Troubleshooting and Scenario-Based Interview Q&A

# 🖥️ PART 1 — TROUBLESHOOTING INTERVIEW Q&A

### 1. Application gets `AccessDenied` when trying to use a KMS key.

**Answer:**

I would check:

1. Which IAM user or role is making the request.
2. Whether the identity has the required KMS permissions.
3. Whether the KMS key policy allows the identity.
4. Whether there is an explicit Deny.
5. Whether the correct KMS key is being used.

---

### 2. The user has IAM permissions but still cannot use the KMS key.

**Answer:**

I would check the KMS key policy.

KMS uses a key policy in addition to IAM permissions.

I would verify whether the user or role is allowed to perform the required operation such as:

```text
kms:Encrypt
kms:Decrypt
```

---

### 3. Encryption works but decryption fails.

**Answer:**

I would check whether the identity has:

```text
kms:Decrypt
```

permission.

I would also verify that the application is using the correct KMS key.

---

### 4. Decryption returns `AccessDenied`.

**Answer:**

I would check:

```text
Correct IAM identity?
        ↓
kms:Decrypt allowed?
        ↓
KMS key policy?
        ↓
Explicit Deny?
        ↓
Correct KMS key?
```

---

### 5. A developer says, "The IAM policy allows KMS, so why is access denied?"

**Answer:**

I would explain that KMS authorization can involve the IAM policy and the KMS key policy.

I would inspect both policies and look for explicit Denies.

---

### 6. The application uses the wrong KMS key.

**Answer:**

I would verify the KMS key ID or ARN configured in the application or AWS service configuration.

Then I would check whether the application identity has permission to use that key.

---

### 7. A KMS key exists but the application cannot use it.

**Answer:**

The existence of the key does not automatically grant permission.

I would check:

* IAM permissions
* KMS key policy
* Key state
* Correct key ID
* Explicit Deny

---

### 8. Why should you not immediately add `kms:*`?

**Answer:**

Because that violates least privilege.

I would identify the exact operation required and grant only the required permission.

For example:

```text
kms:Encrypt
```

instead of:

```text
kms:*
```

---

### 9. A KMS key was scheduled for deletion. Can the application continue using it?

**Answer:**

I would not rely on the key for production workloads once deletion is scheduled.

I would first verify the key state and cancel deletion if the key is still required.

---

### 10. How would you troubleshoot a KMS permission issue?

**Answer:**

I would follow:

```text
WHO?
 ↓
Which IAM user/role?
 ↓
WHAT?
 ↓
Which KMS action?
 ↓
WHICH KEY?
 ↓
Which KMS key?
 ↓
IAM POLICY
 ↓
KEY POLICY
 ↓
EXPLICIT DENY?
 ↓
KEY STATE?
 ↓
TEST AGAIN
```

---

# 🎯 PART 2 — SCENARIO-BASED INTERVIEW Q&A

## Scenario 1 — Application Cannot Encrypt

**Interviewer:**

> Your application needs to encrypt data using KMS, but it receives AccessDenied. What would you check?

**Answer:**

First I would identify which IAM role or user the application is using.

Then I would check whether it has the required KMS permission:

```text
kms:Encrypt
```

I would also check the KMS key policy, explicit Denies, and whether the correct key is being used.

---

## Scenario 2 — Application Can Encrypt but Cannot Decrypt

**Interviewer:**

> Encryption works, but decryption fails. What could be wrong?

**Answer:**

I would check whether the application's IAM role and KMS key policy allow:

```text
kms:Decrypt
```

I would also verify that the application is using the correct KMS key.

---

## Scenario 3 — IAM Allows KMS but Key Policy Does Not

**Interviewer:**

> An IAM role has `kms:Decrypt`, but the application still receives AccessDenied. What would you check?

**Answer:**

I would check the KMS key policy.

KMS authorization depends on the key policy and the applicable IAM permissions.

I would verify that the role is allowed to use the key and check for explicit Denies.

---

## Scenario 4 — Least Privilege

**Interviewer:**

> A developer asks you to give the application `kms:*`. Would you do it?

**Answer:**

No.

I would first identify the exact KMS operation required.

If the application only needs encryption, I would grant the minimum required permission instead of full KMS access.

---

## Scenario 5 — Production Application

**Interviewer:**

> How would you securely use KMS in production?

**Answer:**

I would use a dedicated KMS key where appropriate and follow least privilege.

I would allow only the required IAM roles or AWS services to use the key and control access through the KMS key policy and IAM policies.

I would also monitor KMS activity through CloudTrail.

---

## Scenario 6 — Wrong KMS Key

**Interviewer:**

> The application has KMS permissions but is using the wrong key. What would you do?

**Answer:**

I would verify the configured KMS key ID or ARN.

Then I would confirm that the application role has permission to use the correct key.

---

## Scenario 7 — Key Scheduled for Deletion

**Interviewer:**

> You discover that a production KMS key is scheduled for deletion. What would you do?

**Answer:**

I would immediately verify whether the key is still required.

If it is required, I would cancel the deletion and investigate why the deletion was scheduled.

I would avoid permanently deleting a key that is still needed to decrypt important data.

---

## Scenario 8 — AWS Managed Key

**Interviewer:**

> You see `aws/rds` under AWS managed keys. What does it mean?

**Answer:**

It is an AWS-managed KMS key associated with Amazon RDS for supported encryption operations.

AWS manages the key on behalf of the AWS service.

---

## Scenario 9 — Customer-Managed Key

**Interviewer:**

> Why would you use a customer-managed KMS key instead of an AWS-managed key?

**Answer:**

A customer-managed key provides greater control over key management, policies, permissions, and lifecycle settings.

I would choose it when the application's security or compliance requirements need that additional control.

---

## Scenario 10 — Key Administrator vs Key User

**Interviewer:**

> What is the difference between a KMS key administrator and key user?

**Answer:**

A key administrator manages the KMS key.

A key user performs cryptographic operations using the key.

For example:

```text
Administrator
→ Manage policy
→ Manage key
→ Schedule deletion

Key User
→ Encrypt
→ Decrypt
```

---

# 🧠 HARDER SCENARIOS

### Scenario 11

> The KMS key is enabled and the IAM role has `kms:Decrypt`, but decryption still fails.

**Answer:**

I would check:

```text
Correct IAM role?
Correct KMS key?
kms:Decrypt?
KMS key policy?
Explicit Deny?
Key state?
```

I would troubleshoot each layer rather than immediately adding more permissions.

---

### Scenario 12

> The application worked yesterday but suddenly cannot decrypt data.

**Answer:**

I would investigate what changed.

I would check:

* IAM policy changes
* KMS key policy changes
* Key state
* Application configuration
* KMS key ID
* Explicit Deny
* CloudTrail activity

---

### Scenario 13

> Security asks you to restrict who can decrypt sensitive production data.

**Answer:**

I would identify the specific production IAM roles that require decryption and allow only those identities to use the KMS key for `kms:Decrypt`.

I would follow least privilege and avoid granting broad KMS permissions.

---

### Scenario 14

> An engineer accidentally schedules a production KMS key for deletion.

**Answer:**

I would verify the key and immediately cancel the deletion if the key is still required.

Then I would investigate the CloudTrail activity to understand who scheduled the deletion and why.

---

### Scenario 15

> How would you distinguish a KMS permission problem from an application problem?

**Answer:**

I would first reproduce the error and identify the exact AWS API operation.

If AWS returns `AccessDenied`, I would investigate IAM and KMS authorization.

If the application has a different error, such as an application-level exception, I would investigate the application configuration and code as well.

---

# ⭐ MOST IMPORTANT DAY 7 QUESTIONS

1. What is AWS KMS?
2. Why do we use KMS?
3. What is encryption?
4. What is decryption?
5. What is a symmetric KMS key?
6. What is a KMS key policy?
7. What is a key administrator?
8. What is a key user?
9. What is the difference between AWS-managed and customer-managed keys?
10. What does `kms:Encrypt` do?
11. What does `kms:Decrypt` do?
12. Why use least privilege with KMS?
13. How do you troubleshoot KMS AccessDenied?
14. Why can IAM permissions alone sometimes not explain a KMS access problem?
15. What happens when a KMS key is scheduled for deletion?
16. How does CloudTrail help with KMS?
17. How would you secure KMS in production?
18. How do you identify the correct KMS key?
19. How would you restrict decryption access?
20. What is the difference between key administration and key usage?

---

# 🧩 DAY 7 TROUBLESHOOTING FORMULA

```text
              KMS PROBLEM
                   ↓
                  WHO?
                   ↓
            IAM User / Role
                   ↓
                 WHAT?
                   ↓
            KMS Action
                   ↓
               WHICH KEY?
                   ↓
              KMS Key ID
                   ↓
              IAM POLICY
                   ↓
               KEY POLICY
                   ↓
            EXPLICIT DENY?
                   ↓
              KEY STATE?
                   ↓
              TEST AGAIN
```

### Interview Mindset

Don't just say:

> "I will check the KMS policy."

Say:

> "First I will identify the IAM identity making the request and the exact KMS operation that failed. Then I will verify the KMS key being used, check the IAM permissions and KMS key policy, look for explicit Denies and verify the key state. I would then test again and use CloudTrail if I need to investigate the API activity."

---

# ✅ DAY 7 STATUS

* KMS fundamentals covered
* Symmetric encryption concept covered
* Encryption and decryption concepts covered
* Key administrators covered
* Key users covered
* KMS key policy covered
* AWS-managed keys covered
* Customer-managed keys covered
* KMS troubleshooting covered
* Production scenarios covered
* Cost-safe lab approach followed

