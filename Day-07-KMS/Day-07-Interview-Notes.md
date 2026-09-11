# Day 7 — AWS KMS Interview Notes

## 🔐 AWS Key Management Service (KMS)

### 1. What is AWS KMS?

**Answer:**

AWS KMS is a managed AWS service used to create and control cryptographic keys and control how those keys are used for encryption and decryption.

---

### 2. Why do we use KMS?

**Answer:**

We use KMS to protect sensitive data through encryption and to control which users, roles, or AWS services can use encryption keys.

---

### 3. What is encryption?

**Answer:**

Encryption converts readable data into protected data so unauthorized users cannot easily read it.

```text
Plaintext → Encryption → Ciphertext
```

---

### 4. What is decryption?

**Answer:**

Decryption converts encrypted data back into readable data for an authorized user or service.

```text
Ciphertext → Decryption → Plaintext
```

---

### 5. What is a symmetric KMS key?

**Answer:**

A symmetric KMS key uses the same cryptographic key for encryption and decryption operations.

---

### 6. What is a KMS key policy?

**Answer:**

A KMS key policy is a resource-based policy that controls who can manage or use a KMS key and what actions they can perform.

---

### 7. What is a key administrator?

**Answer:**

A key administrator is an IAM user or role that has permissions to manage the KMS key.

For example, administrators can manage the key policy or schedule key deletion.

---

### 8. What is a key user?

**Answer:**

A key user is an IAM user or role that has permissions to perform cryptographic operations using the KMS key.

Examples include:

```text
kms:Encrypt
kms:Decrypt
kms:DescribeKey
```

---

### 9. What is the difference between key administrator and key user?

**Answer:**

The key administrator manages the KMS key.

The key user uses the KMS key for cryptographic operations.

```text
Administrator → Manages key

User → Uses key
```

---

### 10. What is an AWS-managed KMS key?

**Answer:**

An AWS-managed key is a KMS key created and managed by AWS for an AWS service.

Examples can include:

```text
aws/rds
aws/secretsmanager
```

---

### 11. What is a customer-managed KMS key?

**Answer:**

A customer-managed key is a KMS key that the customer creates and controls.

The customer can manage its policy, permissions, aliases, rotation settings, and deletion schedule.

---

### 12. AWS-managed key vs customer-managed key?

**Answer:**

AWS-managed keys are created and managed by AWS for supported AWS services.

Customer-managed keys are created and managed by the customer and provide greater control over key management and policies.

---

### 13. What does `kms:Encrypt` mean?

**Answer:**

It allows an authorized identity to use the KMS key for encryption.

---

### 14. What does `kms:Decrypt` mean?

**Answer:**

It allows an authorized identity to use the KMS key for decryption.

---

### 15. What does `kms:DescribeKey` mean?

**Answer:**

It allows an identity to retrieve information about the KMS key.

---

### 16. Why should KMS permissions follow least privilege?

**Answer:**

Only the required users, roles, and services should be allowed to use the key.

This reduces the risk of unauthorized encryption or decryption operations.

---

### 17. Is KMS regional?

**Answer:**

KMS keys are regional resources. A key created in one AWS region is not automatically the same key in another region.

---

### 18. Can a KMS key be immediately deleted?

**Answer:**

A customer-managed KMS key is not permanently deleted immediately. AWS requires a scheduled deletion waiting period before permanent deletion.

---

### 19. How does KMS help with auditing?

**Answer:**

KMS integrates with AWS CloudTrail, which can record KMS API activity and help identify who performed key-related operations.

---

### 20. Give a real-world example of KMS.

**Answer:**

Suppose an application stores sensitive customer information.

The application can use encryption to protect the data, while KMS controls access to the cryptographic key.

```text
Application
     ↓
AWS Service / Application
     ↓
KMS
     ↓
Encryption Key
     ↓
Encrypted Data
```

---

# ⭐ Best Interview Answer

> I understand KMS as a centralized AWS service for managing encryption keys. I can configure key administrators and key users through KMS key policies and control operations such as `kms:Encrypt` and `kms:Decrypt`. In production, I would follow least privilege and use KMS with services such as S3, RDS, or Secrets Manager when encryption requirements exist.

