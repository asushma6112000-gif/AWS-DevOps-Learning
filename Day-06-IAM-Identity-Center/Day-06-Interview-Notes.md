# Day 6 — IAM Identity Center

## Interview Notes

---

# 1. What is AWS IAM Identity Center?

**Answer:**

AWS IAM Identity Center is an AWS service used to provide centralized workforce access to AWS accounts and applications.

It allows organizations to manage users and groups centrally and assign **permission sets** that define the access users receive in AWS accounts.

---

# 2. What was IAM Identity Center previously called?

**Answer:**

IAM Identity Center was previously called **AWS Single Sign-On (AWS SSO)**.

---

# 3. What problem does IAM Identity Center solve?

**Answer:**

It simplifies workforce access management.

Instead of creating and managing separate IAM users in every AWS account, organizations can centrally manage users and groups and assign them access through permission sets.

---

# 4. What is a Permission Set?

**Answer:**

A permission set is a collection of permissions that defines what access a user or group receives when accessing an AWS account through IAM Identity Center.

Example:

```text
Developer Permission Set
        ↓
Development Account
        ↓
Developer access
```

Another example:

```text
ReadOnly Permission Set
        ↓
Production Account
        ↓
Read-only access
```

---

# 5. What is the difference between an IAM policy and a Permission Set?

**Answer:**

An IAM policy defines permissions.

A permission set is an IAM Identity Center access configuration that can include AWS managed policies, customer managed policies, and other supported policy configurations.

Simple memory:

```text
IAM Policy
→ Defines permissions

Permission Set
→ Defines the access a workforce user/group receives in an AWS account
```

---

# 6. What is an IAM Identity Center user?

**Answer:**

An IAM Identity Center user represents a workforce identity that can be given access to AWS accounts and applications.

The user can sign in through the IAM Identity Center access portal.

---

# 7. What is a Group?

**Answer:**

A group is a collection of users.

Groups make access management easier because permissions can be assigned to a group instead of configuring every user individually.

Example:

```text
DevOps Group
│
├── User 1
├── User 2
└── User 3
```

---

# 8. Why use groups?

**Answer:**

Groups simplify access management.

For example, if 20 developers need the same permissions, I can put them in a Developers group and assign the required permission set to that group.

---

# 9. What is SSO?

**Answer:**

SSO means **Single Sign-On**.

It allows a user to authenticate once and access multiple authorized applications or AWS accounts without maintaining separate login credentials for each account.

---

# 10. How does IAM Identity Center work?

**Answer:**

The basic flow is:

```text
User / Group
      ↓
Permission Set
      ↓
AWS Account
      ↓
Access Portal
      ↓
AWS Console
      ↓
AWS Resources
```

The permission set determines the level of access.

---

# 11. What is the AWS Access Portal?

**Answer:**

The AWS access portal is the web-based portal where users sign in and see the AWS accounts and applications they have been authorized to access.

---

# 12. Does IAM Identity Center use permanent access keys?

**Answer:**

IAM Identity Center is designed around centralized workforce access and temporary credentials.

Users can obtain temporary AWS credentials for authorized access rather than relying on long-term access keys.

---

# 13. Why are temporary credentials better?

**Answer:**

Temporary credentials reduce the security risk associated with long-term credentials.

They have a limited lifetime and can be issued based on the user's authorized access.

---

# 14. IAM User vs IAM Identity Center User

| IAM User                          | IAM Identity Center User                    |
| --------------------------------- | ------------------------------------------- |
| Created inside an AWS account     | Used for centralized workforce access       |
| Mainly account-specific           | Designed for centralized access             |
| Can use long-term access keys     | Uses temporary access for workforce access  |
| IAM policies directly attached    | Permission sets determine account access    |
| Not primarily an SSO solution     | Designed for SSO                            |
| More manual for many AWS accounts | Better suited to multi-account environments |

---

# 15. IAM Role vs Permission Set

**IAM Role:**

An IAM role is an AWS identity with permissions that can be assumed by trusted principals such as AWS services, users, or other identities.

**Permission Set:**

A permission set is an IAM Identity Center configuration used to define the permissions a user or group receives in an AWS account.

Simple memory:

```text
IAM Role
→ AWS identity that can be assumed

Permission Set
→ IAM Identity Center access configuration
```

---

# 16. What is AWS Organizations?

**Answer:**

AWS Organizations is a service used to centrally manage multiple AWS accounts.

Example:

```text
AWS Organization
│
├── Development Account
├── Testing Account
└── Production Account
```

IAM Identity Center can be used with AWS Organizations to provide centralized workforce access across multiple AWS accounts.

---

# 17. Why combine IAM Identity Center with AWS Organizations?

**Answer:**

Because organizations often have many AWS accounts.

IAM Identity Center can centrally manage workforce access while AWS Organizations provides centralized management of the AWS accounts.

Example:

```text
                 AWS Organization
                        │
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
   Development       Testing        Production
        ↑               ↑               ↑
        └──────── IAM Identity Center ───┘
                         ↑
                       Users
                         ↑
                  Permission Sets
```

---

# 18. Can one user access multiple AWS accounts?

**Answer:**

Yes.

A user can be assigned access to multiple AWS accounts.

The user may receive different permission sets for different accounts.

Example:

```text
Developer
│
├── Development → Developer access
├── Testing → Developer access
└── Production → ReadOnly access
```

---

# 19. Can the same user have different permissions in different accounts?

**Answer:**

Yes.

For example:

```text
Development
→ Administrator/Developer-level access

Production
→ ReadOnly access
```

This is useful for enforcing least privilege.

---

# 20. What is least privilege in IAM Identity Center?

**Answer:**

Least privilege means giving a user only the permissions required to perform their job.

For example, a developer may need write access in Development but only read access in Production.

---

# 21. Production Example

**Interviewer:**

> Your company has Development, Testing, and Production AWS accounts. How would you give developers access?

**Answer:**

I would use IAM Identity Center with AWS Organizations.

I would create a Developers group and assign appropriate permission sets to the required AWS accounts.

For example:

```text
Developers Group
│
├── Development → Developer
├── Testing → Developer
└── Production → ReadOnly
```

This provides centralized access while following least privilege.

---

# 22. How would you onboard a new employee?

**Answer:**

I would:

1. Create or provision the user's identity.
2. Add the user to the appropriate group.
3. Assign the required permission set to the group.
4. Give access to the required AWS accounts.
5. Ask the user to sign in through the access portal.

---

# 23. How would you offboard an employee?

**Answer:**

I would remove or disable the user's access according to the organization's identity lifecycle process.

I would also remove the user from groups or revoke the relevant AWS account/application access.

The goal is to make sure the employee no longer has unauthorized access.

---

# 24. What happens if a user has no account assignment?

**Answer:**

The user will not have access to that AWS account through IAM Identity Center.

The user may be able to sign in to the access portal but will not see an AWS account for which they have no assignment.

---

# 25. What happens if a permission set is too restrictive?

**Answer:**

The user may successfully sign in but receive **AccessDenied** when attempting an action that the permission set does not allow.

I would check:

```text
Correct user?
        ↓
Correct AWS account?
        ↓
Correct permission set?
        ↓
Required action allowed?
        ↓
Resource allowed?
        ↓
Explicit deny?
```

---

# 26. What happens if the wrong permission set is assigned?

**Answer:**

The user may receive either too much or too little access.

If the permission set is too restrictive, the user may receive AccessDenied.

If it is too powerful, it can create a security risk.

I would correct the assignment and follow least privilege.

---

# 27. How would you troubleshoot an IAM Identity Center access problem?

**Answer:**

I would troubleshoot systematically:

```text
WHO?
 ↓
Which user/group?
 ↓
WHICH ACCOUNT?
 ↓
Which AWS account are they accessing?
 ↓
WHICH PERMISSION SET?
 ↓
Is the correct permission set assigned?
 ↓
WHAT ACTION?
 ↓
Which AWS action is failing?
 ↓
WHICH RESOURCE?
 ↓
Is the resource allowed?
 ↓
DENY?
 ↓
Check explicit/resource restrictions
 ↓
TEST AGAIN
```

---

# 28. User can log in but cannot access an AWS account.

**Answer:**

I would check the account assignment.

I would verify:

* Correct user/group
* Correct AWS account
* Correct permission set
* Assignment status

The user must have an appropriate assignment to access the account.

---

# 29. User can access the account but gets AccessDenied.

**Answer:**

I would check the permission set and determine whether it contains the required permission.

I would also check:

* Required AWS action
* Resource
* Explicit deny
* Other applicable AWS policy controls

---

# 30. User sees the wrong AWS account.

**Answer:**

I would verify the account assignments associated with the user or group.

I would also confirm that the user selected the correct AWS account from the access portal.

---

# 31. Multiple users suddenly lose access.

**Answer:**

I would first look for a common configuration issue.

For example:

```text
Group
 ↓
Permission Set
 ↓
Account Assignment
```

If many users are affected, I would check whether the group membership, permission set, or account assignment changed.

---

# 32. One user loses access while other users are working.

**Answer:**

I would focus on the individual user's:

* Identity
* Group membership
* Account assignment
* Permission set
* Authentication status

Because other users are working, the issue may be specific to that user's access configuration.

---

# 33. How do you troubleshoot AccessDenied?

**Answer:**

I would identify exactly what action was denied.

Then I would verify:

```text
User
 ↓
Group
 ↓
Permission Set
 ↓
AWS Account
 ↓
Action
 ↓
Resource
 ↓
Allow / Deny
```

I would not immediately grant Administrator access.

I would determine the minimum permission required.

---

# 34. What is the difference between authentication and authorization?

**Authentication:**

> **Who are you?**

Authorization:

> **What are you allowed to do?**

Example:

```text
Login
 ↓
Authentication
 ↓
Who is the user?
 ↓
Authorization
 ↓
What AWS account and permissions can they access?
```

---

# 35. What is the biggest advantage of IAM Identity Center?

**Answer:**

The biggest advantage is centralized workforce access management.

It makes it easier to manage access to multiple AWS accounts and applications while providing SSO and centralized permission management.

---

# 36. Why not create IAM users for every employee?

**Answer:**

For a large multi-account environment, creating and managing individual IAM users across every AWS account becomes difficult and increases administrative overhead.

IAM Identity Center provides a more centralized approach for workforce access.

---

# 37. Does IAM Identity Center replace all IAM roles?

**Answer:**

No.

IAM roles are still an important AWS identity mechanism.

IAM Identity Center provides workforce access, while IAM roles are commonly used by AWS services, applications, workloads, and other trusted principals.

---

# 38. Does IAM Identity Center replace IAM?

**Answer:**

No.

IAM and IAM Identity Center solve related but different problems.

```text
IAM
→ AWS resource access and identities

IAM Identity Center
→ Centralized workforce access and SSO
```

Organizations commonly use both.

---

# 39. 30-Second Interview Answer

**Question: What is IAM Identity Center?**

**Answer:**

> AWS IAM Identity Center is a service for centralized workforce access to AWS accounts and applications. Users or groups are assigned permission sets, and those permission sets determine the level of access they receive in AWS accounts. It provides an SSO-based access portal and supports temporary credentials, making it useful for centralized and least-privilege access in multi-account environments.

---

# 40. 1-Minute Interview Answer

**Question: Explain how IAM Identity Center works.**

**Answer:**

> IAM Identity Center is designed for centralized workforce access. We can manage users and groups and assign permission sets to them. The permission set defines the permissions they receive when accessing an AWS account. In a multi-account environment, such as Development, Testing, and Production, the same user can have different permission sets in each account. Users sign in through the AWS access portal and access only the accounts and applications they are authorized to use. This reduces the need for separate IAM users across accounts and helps organizations implement SSO and least privilege.

---

# 41. Interview Follow-Up Questions

### Q: What is a permission set?

**Answer:**

A configuration that defines the permissions a user or group receives in an AWS account through IAM Identity Center.

### Q: Can one user access multiple AWS accounts?

**Answer:**

Yes, if the appropriate account assignments are configured.

### Q: Can permissions differ between accounts?

**Answer:**

Yes. Different permission sets can be assigned in different AWS accounts.

### Q: What is SSO?

**Answer:**

Single Sign-On allows users to authenticate centrally and access multiple authorized resources or applications.

### Q: Why use groups?

**Answer:**

Groups simplify access management for users with similar access requirements.

### Q: Why use temporary credentials?

**Answer:**

They reduce reliance on long-term credentials and improve security.

### Q: What if the user gets AccessDenied?

**Answer:**

I would verify the account assignment, permission set, required action, resource, and applicable deny controls.

---

# 42. Real-World Access Model

```text
                   COMPANY
                      │
                      ▼
             IAM Identity Center
                      │
              ┌───────┴───────┐
              ↓               ↓
           Groups           Users
              │
              ▼
       Permission Sets
              │
      ┌───────┼────────┐
      ↓       ↓        ↓
     DEV     TEST      PROD
      │       │        │
      ↓       ↓        ↓
 Developer Developer  ReadOnly
```

---

# 43. Interview Mindset

Do not simply say:

> "IAM Identity Center is SSO."

A stronger answer is:

> "IAM Identity Center provides centralized workforce access. Users or groups are assigned permission sets for specific AWS accounts, and users access those accounts through the access portal using temporary credentials."

---

# ⭐ Most Important Day 6 Questions

1. What is IAM Identity Center?
2. What was it previously called?
3. What problem does it solve?
4. What is a permission set?
5. What is an Identity Center user?
6. What is a group?
7. What is SSO?
8. What is the AWS access portal?
9. How does IAM Identity Center work?
10. IAM User vs Identity Center User?
11. IAM Role vs Permission Set?
12. What is AWS Organizations?
13. Why use IAM Identity Center with Organizations?
14. Can one user access multiple AWS accounts?
15. Can the same user have different permissions in different accounts?
16. What is least privilege?
17. How would you onboard a user?
18. How would you offboard a user?
19. How would you troubleshoot AccessDenied?
20. How would you troubleshoot a user who cannot access an AWS account?
21. Why use groups?
22. Why use temporary credentials?
23. Does IAM Identity Center replace IAM?
24. Does it replace IAM roles?
25. What is the main production benefit?

---

# 🧠 Day 6 Memory Formula

```text
IAM IDENTITY CENTER
        ↓
     USER/GROUP
        ↓
 PERMISSION SET
        ↓
   AWS ACCOUNT
        ↓
 ACCESS PORTAL
        ↓
 TEMPORARY ACCESS
        ↓
 AWS RESOURCES
```

### Remember:

```text
USER
→ Who?

PERMISSION SET
→ What access?

AWS ACCOUNT
→ Where?

ACCESS PORTAL
→ How do I access?

TEMPORARY CREDENTIALS
→ How is AWS access provided?
```

---

# ✅ Interview Goal

After Day 6, I should be able to explain IAM Identity Center clearly as a **fresher/intermediate-level Cloud/DevOps candidate**, especially in multi-account AWS environments.

I should be able to explain:

```text
User
→ Group
→ Permission Set
→ AWS Account
→ Access Portal
→ Temporary Access
```

and troubleshoot common access and permission problems systematically.

