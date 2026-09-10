# Day 6 — IAM Identity Center

## AWS IAM Identity Center — Learning Lab

### 🎯 Objective

The objective of Day 6 is to understand **AWS IAM Identity Center**, previously known as **AWS Single Sign-On (AWS SSO)**.

I learned how IAM Identity Center can provide centralized access to AWS accounts and applications using users, groups, and permission sets.

---

# 🔐 What is IAM Identity Center?

**AWS IAM Identity Center** is an AWS service used to provide centralized workforce access to AWS accounts and applications.

Instead of creating separate IAM users inside every AWS account, organizations can manage workforce identities centrally and provide access through:

```text
User
  ↓
Group
  ↓
Permission Set
  ↓
AWS Account
  ↓
AWS Console
```

IAM Identity Center is commonly used in organizations with multiple AWS accounts.

---

# 🏗️ IAM Identity Center Architecture

```text
                 IAM Identity Center
                         │
                         ▼
                      Users
                         │
                         ▼
                      Groups
                         │
                         ▼
                 Permission Set
                         │
                         ▼
                    AWS Account
                         │
                         ▼
                  Access Portal
                         │
                         ▼
                 AWS Management Console
```

The main idea is:

> **Identity → Permission Set → Account → Access**

---

# 👤 User

A user represents a person who needs access to AWS resources or applications.

Example:

```text
User:
day6-cloud-user
```

In a production organization, users could represent:

```text
Developer
DevOps Engineer
Cloud Engineer
Security Engineer
Administrator
```

---

# 👥 Group

Groups allow organizations to manage permissions for multiple users together.

Example:

```text
Developers
│
├── Developer 1
├── Developer 2
└── Developer 3
```

Instead of assigning permissions individually to every user, a permission set can be assigned to the group.

This makes access management easier when the organization grows.

---

# 🔑 Permission Set

A **Permission Set** defines the permissions a user or group receives when accessing an AWS account through IAM Identity Center.

Example:

```text
Day6-ReadOnly-PermissionSet
        │
        └── Read-only AWS permissions
```

Permission sets can use AWS managed policies or customer managed policies depending on the organization's access design.

Examples of access levels:

```text
ReadOnly
Developer
PowerUser
Administrator
```

---

# 🏢 AWS Account

The permission set is assigned to an AWS account.

Example:

```text
User
  ↓
Permission Set
  ↓
Production AWS Account
```

The same user can receive different permission sets in different AWS accounts.

For example:

```text
Development Account
→ Developer access

Production Account
→ ReadOnly access
```

This helps implement least-privilege access.

---

# 🌐 Access Portal

IAM Identity Center provides an **AWS access portal** where users can sign in.

The user can see the AWS accounts and applications they are allowed to access.

Conceptually:

```text
User
  ↓
Access Portal
  ↓
Select AWS Account
  ↓
Select Permission Set
  ↓
AWS Console
```

---

# ⏳ Temporary Credentials

IAM Identity Center provides users with temporary credentials when they access AWS resources.

This is preferable to using permanent access keys for workforce access.

Conceptually:

```text
User Login
    ↓
Authentication
    ↓
Permission Set
    ↓
Temporary Credentials
    ↓
AWS Resources
```

Temporary credentials reduce the need to distribute long-term AWS access keys.

---

# 🔄 IAM User vs IAM Identity Center User

## IAM User

Traditional IAM user:

```text
IAM User
   ↓
IAM Policy
   ↓
AWS Account
```

IAM users are created inside a specific AWS account.

They can have credentials such as:

* Console password
* Access keys

---

## IAM Identity Center

IAM Identity Center:

```text
Identity Center User
       ↓
Permission Set
       ↓
AWS Account
       ↓
AWS Console
```

It is designed for centralized workforce access across AWS accounts and applications.

---

# 🆚 IAM User vs IAM Identity Center

| Feature                     | IAM User               | IAM Identity Center |
| --------------------------- | ---------------------- | ------------------- |
| Main purpose                | AWS account identity   | Workforce access    |
| Centralized access          | Limited                | Yes                 |
| Multiple AWS accounts       | More manual            | Designed for it     |
| Permission model            | IAM policies           | Permission sets     |
| Temporary access            | Possible through roles | Common access model |
| SSO experience              | Not the main purpose   | Yes                 |
| Enterprise workforce access | Less suitable          | Common choice       |

---

# 🏢 IAM Identity Center + AWS Organizations

In a multi-account organization, IAM Identity Center can work together with **AWS Organizations**.

Example:

```text
AWS Organization
│
├── Development Account
│
├── Testing Account
│
└── Production Account
       │
       ▼
IAM Identity Center
       │
       ▼
Users / Groups
       │
       ▼
Permission Sets
```

This allows centralized workforce access management across multiple AWS accounts.

---

# 🔐 Example Production Scenario

Imagine a company has three AWS accounts:

```text
Development
Testing
Production
```

A DevOps engineer may need:

```text
Development
→ Developer permissions

Testing
→ Developer permissions

Production
→ ReadOnly permissions
```

IAM Identity Center can provide different account access based on the required permission set.

This follows the principle of:

> **Least Privilege**

---

# 🛡️ Why IAM Identity Center is Useful

Organizations can use IAM Identity Center to:

* Centralize workforce access
* Provide SSO
* Manage access to multiple AWS accounts
* Use permission sets
* Reduce the need for individual IAM users
* Provide temporary credentials
* Simplify employee onboarding and offboarding
* Apply least-privilege access

---

# 🧪 Day 6 Safe Lab Limitation

For this Day 6 learning exercise, I did **not** enable or configure IAM Identity Center in my AWS account.

During the setup process, AWS displayed a warning that enabling the required Organization-based IAM Identity Center setup could change the account to a paid **Pay-As-You-Go** plan.

Because this learning environment is intended to remain cost-safe, I did not proceed with the billing-impacting setup.

Therefore, this Day 6 documentation focuses on:

* IAM Identity Center architecture
* Users
* Groups
* Permission Sets
* AWS account assignments
* Access Portal
* Temporary credentials
* Production use cases
* Troubleshooting concepts
* Interview preparation

I am **not claiming that an actual IAM Identity Center user, permission set, or account assignment was created during this lab.**

---

# 🧠 What I Learned

### 1. IAM Identity Center

Centralized workforce access management for AWS accounts and applications.

### 2. User

Represents an individual who needs access.

### 3. Group

Allows multiple users to be managed together.

### 4. Permission Set

Defines the permissions assigned to a user or group for an AWS account.

### 5. Access Portal

Provides a centralized login location for users.

### 6. Temporary Credentials

Users can receive temporary credentials instead of relying on permanent access keys.

### 7. AWS Organizations

Helps manage multiple AWS accounts and works with IAM Identity Center for centralized account access.

---

# 🎯 Interview Explanation

If an interviewer asks:

> **What is AWS IAM Identity Center?**

I would answer:

> AWS IAM Identity Center is an AWS service used for centralized workforce access to AWS accounts and applications. Instead of creating separate IAM users in every AWS account, organizations can manage users and groups centrally and use permission sets to provide the required access to AWS accounts. Users can sign in through an access portal and receive temporary credentials based on their assigned permissions.

---

# 🔥 Short Interview Answer

> IAM Identity Center provides centralized SSO-based workforce access to AWS accounts and applications. Users or groups are assigned permission sets, and those permission sets determine what access they receive in an AWS account.

---

# 🧩 Day 6 Architecture Summary

```text
                IAM Identity Center
                         │
                         ▼
                  User / Group
                         │
                         ▼
                  Permission Set
                         │
                         ▼
                    AWS Account
                         │
                         ▼
                   Access Portal
                         │
                         ▼
                  Temporary Access
                         │
                         ▼
                  AWS Resources
```

---

# 💼 Production Use Case

A company has:

```text
100 Developers
20 DevOps Engineers
10 Security Engineers
5 Administrators
```

and multiple AWS accounts.

Instead of manually creating IAM users in every account, the company can use IAM Identity Center to centrally manage workforce access.

Example:

```text
DevOps Group
     │
     ├── Development → Developer
     │
     ├── Testing → Developer
     │
     └── Production → ReadOnly
```

This provides centralized access management and helps maintain least privilege.

---

# ✅ Day 6 Status

* IAM Identity Center concept understood
* IAM Identity Center architecture understood
* Users understood
* Groups understood
* Permission Sets understood
* AWS account assignments understood
* Access Portal understood
* Temporary credentials understood
* IAM User vs Identity Center comparison understood
* AWS Organizations relationship understood
* Production use case understood
* Safe-lab limitation documented
* No billing-impacting Identity Center setup performed

---

## ⭐ Key Memory

```text
IAM Identity Center

WHO?
→ User / Group

WHAT ACCESS?
→ Permission Set

WHERE?
→ AWS Account

HOW?
→ Access Portal / SSO

CREDENTIALS?
→ Temporary credentials
```

