# Day 6 — IAM Identity Center

## Troubleshooting and Scenario-Based Interview Q&A

---

# 🖥️ PART 1 — TROUBLESHOOTING INTERVIEW Q&A

## 1. User cannot log in to IAM Identity Center.

**Answer:**

I would first check whether the user is using the correct AWS access portal URL.

Then I would check:

* Username
* Authentication method
* User status
* Identity source
* Whether the user is allowed to sign in

I would also check whether there is an authentication or identity-provider issue.

---

## 2. User can log in but cannot see an AWS account.

**Answer:**

I would check the user's account assignment.

I would verify:

1. Correct user or group.
2. Correct AWS account.
3. Correct permission set.
4. Assignment is configured correctly.

The user must have an appropriate assignment to access the AWS account.

---

## 3. User can see the AWS account but cannot perform an action.

**Answer:**

I would check the permission set.

I would identify the exact AWS action that failed and verify whether the permission set allows that action on the required resource.

For example:

```text
User
 ↓
Permission Set
 ↓
AWS Account
 ↓
AWS Action
 ↓
Resource
```

---

## 4. User receives AccessDenied.

**Answer:**

I would not immediately give Administrator permissions.

I would identify:

```text
WHO?
→ Which user/group?

WHAT?
→ Which AWS action?

WHERE?
→ Which AWS resource?

WHY?
→ Which permission or deny caused the failure?
```

Then I would check the permission set and other applicable AWS policy controls.

---

## 5. User has the correct permission set but still receives AccessDenied.

**Answer:**

I would check whether another policy or control is causing a deny.

I would investigate:

* Permission set
* Resource policy
* Explicit Deny
* Permissions boundary where applicable
* Service Control Policy (SCP) where applicable
* Resource-specific restrictions

An Allow does not automatically overcome an applicable Explicit Deny.

---

## 6. User sees the wrong AWS account.

**Answer:**

I would verify the account assignments for the user or group.

I would check whether the user has access to multiple AWS accounts and confirm that the correct account was selected from the access portal.

---

## 7. Multiple users suddenly lose access.

**Answer:**

Because multiple users are affected, I would look for a common configuration problem.

I would check:

```text
Group
 ↓
Group Membership
 ↓
Permission Set
 ↓
Account Assignment
 ↓
AWS Account
```

I would also investigate recent configuration changes.

---

## 8. One user loses access but other users are working.

**Answer:**

I would focus on the affected user's individual configuration.

I would check:

* User status
* Group membership
* Account assignment
* Permission set
* Authentication status

Since other users are working, the problem may be specific to the affected user's access configuration.

---

## 9. User has Developer access in Development but cannot access Production.

**Answer:**

I would check whether the user has an assignment for the Production account.

Having access to one AWS account does not automatically mean the user has access to another account.

For example:

```text
Development
→ Developer access

Production
→ No assignment
```

The user would not automatically receive Production access.

---

## 10. User has ReadOnly access but needs to create an EC2 instance.

**Answer:**

I would identify the required permissions for the job.

A ReadOnly permission set normally does not provide the permissions required to create infrastructure.

I would assign an appropriate permission set according to the organization's least-privilege requirements rather than giving Administrator access unnecessarily.

---

## 11. User can access the AWS Console but cannot access S3.

**Answer:**

I would verify whether the assigned permission set contains the required S3 permissions.

I would identify the exact failed S3 action, such as:

```text
s3:ListBucket
s3:GetObject
s3:PutObject
```

Then I would check the relevant resource permissions and any applicable deny policies.

---

## 12. User can read an S3 object but cannot upload.

**Answer:**

I would check whether the permission set allows:

```text
s3:PutObject
```

Read and write are separate permissions.

Having:

```text
s3:GetObject
```

does not automatically provide:

```text
s3:PutObject
```

---

## 13. User can upload but cannot delete an S3 object.

**Answer:**

I would check whether:

```text
s3:DeleteObject
```

is allowed.

Upload and delete are separate permissions.

This may be intentional in a least-privilege environment.

---

## 14. A user was added to a group but still cannot access the account.

**Answer:**

I would verify:

1. User is actually a member of the group.
2. The group has the required permission set assignment.
3. The assignment targets the correct AWS account.
4. The permission set provides the required permissions.

I would then test the user's access again.

---

## 15. Group assignment gives users too much access.

**Answer:**

I would review the permission set assigned to the group.

If the permission set provides unnecessary permissions, I would reduce the permissions according to least privilege.

I would avoid giving broad Administrator access when it is not required.

---

## 16. A new employee needs access to Development but not Production.

**Answer:**

I would give the user or their group an assignment to the Development account only.

I would not provide Production access unless the user's job requires it.

This follows least privilege.

---

## 17. A developer needs different access in Development and Production.

**Answer:**

I would use different permission sets or assignments.

Example:

```text
Developer
│
├── Development
│     ↓
│   Developer Access
│
└── Production
      ↓
    ReadOnly Access
```

This provides different access levels based on the environment.

---

## 18. User can log in but cannot see any AWS accounts.

**Answer:**

I would check whether the user or their group has any AWS account assignments.

I would verify:

```text
User
 ↓
Group
 ↓
Permission Set
 ↓
AWS Account Assignment
```

If there is no appropriate assignment, the user may have no AWS account available in the access portal.

---

## 19. Permission set appears correct but access still fails.

**Answer:**

I would verify that the correct permission set is actually assigned to the correct user/group and AWS account.

Then I would check the exact action, resource, and any applicable deny conditions.

I would not assume that the permission set name alone proves the effective permissions.

---

## 20. User was removed from a group but still appears to have access.

**Answer:**

I would verify the user's current assignments.

The user may have received access through another group or through a direct assignment.

I would review all relevant account assignments and remove access that is no longer required.

---

# 🎯 PART 2 — SCENARIO-BASED INTERVIEW Q&A

## Scenario 1 — New Developer

**Interviewer:**

> A new developer joins the company. They need access to Development but should not access Production. What would you do?

**Answer:**

I would add the developer to the appropriate group and assign a permission set to the Development account.

I would not assign Production access because it is not required.

This follows least privilege.

---

## Scenario 2 — Production ReadOnly

**Interviewer:**

> Developers need to troubleshoot Production but should not modify resources. How would you design access?

**Answer:**

I would provide a ReadOnly permission set for Production.

For Development, they could have a more appropriate developer-level permission set.

Example:

```text
Developers
│
├── Development → Developer
└── Production → ReadOnly
```

---

## Scenario 3 — AccessDenied

**Interviewer:**

> A developer can log in successfully but receives AccessDenied when creating an EC2 instance. What do you check?

**Answer:**

I would first identify the exact EC2 action that was denied.

Then I would check:

1. User/group.
2. Permission set.
3. AWS account.
4. Required EC2 permissions.
5. Resource restrictions.
6. Explicit Deny.
7. SCP or other applicable controls.

I would grant only the minimum permissions required.

---

## Scenario 4 — Wrong Account

**Interviewer:**

> A developer says they cannot find the Development account in the access portal. What do you check?

**Answer:**

I would check the user's or group's AWS account assignment.

I would verify:

```text
Correct user?
 ↓
Correct group?
 ↓
Correct permission set?
 ↓
Correct AWS account?
```

---

## Scenario 5 — Multiple Accounts

**Interviewer:**

> Your company has 50 AWS accounts. How would you manage developer access?

**Answer:**

I would use IAM Identity Center together with AWS Organizations.

I would manage users and groups centrally and use permission sets and account assignments to provide the required access.

This is more scalable than manually creating IAM users in every account.

---

## Scenario 6 — Same User, Different Permissions

**Interviewer:**

> Can the same developer have different permissions in Development and Production?

**Answer:**

Yes.

For example:

```text
Development
→ Developer access

Production
→ ReadOnly access
```

This is a common least-privilege design.

---

## Scenario 7 — Group-Based Access

**Interviewer:**

> Why would you use a group instead of assigning every developer individually?

**Answer:**

Groups simplify access management.

If 20 developers need the same access, I can manage their access through the group rather than configuring each user separately.

When a new developer joins, I can add them to the appropriate group.

---

## Scenario 8 — Employee Leaves

**Interviewer:**

> An employee leaves the company. What would you do?

**Answer:**

I would follow the organization's identity lifecycle process to disable or remove the user's access.

I would also review group membership and account assignments to make sure the user no longer has unauthorized access.

---

## Scenario 9 — Too Much Access

**Interviewer:**

> A developer accidentally receives Administrator-level access to Production. What would you do?

**Answer:**

I would treat it as an excessive-permission issue.

I would immediately review the assignment and replace the overly broad permission with the minimum access required for the developer's job.

I would also investigate how the excessive access was granted.

---

## Scenario 10 — Many Users Lose Access

**Interviewer:**

> Twenty developers suddenly lose access to Production. What is your troubleshooting approach?

**Answer:**

Because many users are affected, I would first check common access configuration.

I would investigate:

```text
Group membership
       ↓
Permission set
       ↓
Account assignment
       ↓
Production account
```

I would also check recent changes to IAM Identity Center and AWS Organizations configuration.

---

## Scenario 11 — User Can Log In but No AWS Account

**Interviewer:**

> The user successfully signs in but sees no AWS accounts. What is likely wrong?

**Answer:**

I would check the user's account assignments.

Authentication succeeded, but the user may not have been authorized for any AWS account.

This demonstrates the difference between:

```text
Authentication
→ Who are you?

Authorization
→ What are you allowed to access?
```

---

## Scenario 12 — Permission Set Changed

**Interviewer:**

> A permission set was changed and several developers suddenly cannot perform an action. What would you do?

**Answer:**

I would identify the permission that was removed or changed.

Then I would verify whether the change was intentional.

If the action is required, I would add the minimum required permission according to least privilege.

---

## Scenario 13 — Developer Requests Administrator

**Interviewer:**

> A developer asks for Administrator access because they received AccessDenied. What would you do?

**Answer:**

I would not immediately grant Administrator access.

I would first identify:

```text
Exact action
 ↓
Exact resource
 ↓
Required permission
 ↓
Reason for AccessDenied
```

Then I would grant only the required permission if appropriate.

---

## Scenario 14 — Security Review

**Interviewer:**

> Security asks how you would reduce excessive AWS access.

**Answer:**

I would review:

* Users
* Groups
* Permission sets
* Account assignments
* Access levels
* Unused access
* Excessive permissions

I would apply least privilege and remove unnecessary access.

---

## Scenario 15 — Production Incident

**Interviewer:**

> During a Production incident, a developer says they suddenly cannot perform a required action. What would you do?

**Answer:**

I would first identify the exact action and resource.

Then I would verify the user's current account assignment and permission set.

I would check whether a recent policy, permission set, group, SCP, or resource policy change caused the denial.

I would avoid giving unrestricted Administrator access just to bypass the problem.

---

# 🧠 PART 3 — TROUBLESHOOTING FORMULA

When troubleshooting IAM Identity Center access, I would follow:

```text
              ACCESS PROBLEM
                    ↓
                  WHO?
                    ↓
             User / Group
                    ↓
                WHICH?
                    ↓
              AWS Account
                    ↓
                WHICH?
                    ↓
             Permission Set
                    ↓
                 WHAT?
                    ↓
             Failed Action
                    ↓
                WHERE?
                    ↓
               Resource
                    ↓
                DENY?
                    ↓
        Explicit / SCP / Resource
             restrictions
                    ↓
               TEST AGAIN
```

---

# 🔍 Authentication vs Authorization

This is an important interview concept.

### Authentication

```text
WHO ARE YOU?
```

Example:

> User successfully signs in to the access portal.

### Authorization

```text
WHAT ARE YOU ALLOWED TO DO?
```

Example:

> User can access Development but cannot access Production.

Simple memory:

```text
Authentication
→ Login

Authorization
→ Permissions
```

---

# 🛠️ General Troubleshooting Checklist

When a user reports an access problem, check:

### Step 1 — User

```text
Is the correct user active?
```

### Step 2 — Group

```text
Is the user in the correct group?
```

### Step 3 — AWS Account

```text
Is the correct account assigned?
```

### Step 4 — Permission Set

```text
Is the correct permission set assigned?
```

### Step 5 — Action

```text
What exact AWS action failed?
```

### Step 6 — Resource

```text
What resource is being accessed?
```

### Step 7 — Policies / Denies

```text
Is there an explicit deny?
Is an SCP restricting the action?
Is a resource policy involved?
```

### Step 8 — Test Again

```text
Make the minimum required change.
Test again.
```

---

# ⭐ Interview Mindset

Do not say:

> "I will give the user Administrator access."

A better answer is:

> "First I will identify the exact action that is failing, verify the user's account assignment and permission set, check the required resource permissions and applicable deny controls, and then grant only the minimum permission required."

This demonstrates **least-privilege thinking**.

---

# 🎯 MOST IMPORTANT SCENARIOS TO REMEMBER

### Scenario A

**User cannot see AWS account**

→ Check account assignment.

### Scenario B

**User can see account but gets AccessDenied**

→ Check permission set and required action.

### Scenario C

**Many users lose access**

→ Check group, permission set, and account assignment.

### Scenario D

**One user loses access**

→ Check user's status, group membership, and assignments.

### Scenario E

**User has too much access**

→ Review permission set and reduce permissions.

### Scenario F

**Developer asks for Administrator**

→ Identify exact missing permission instead of granting Administrator.

---

# 💼 Production Troubleshooting Example

**Problem:**

A developer can access Development but cannot create an EC2 instance.

### My approach:

```text
User
 ↓
Which group?
 ↓
Which permission set?
 ↓
Which AWS account?
 ↓
Which EC2 action failed?
 ↓
Which resource?
 ↓
Is the permission allowed?
 ↓
Is there an explicit deny?
 ↓
Is an SCP restricting it?
 ↓
Fix minimum required permission
 ↓
Test again
```

---

# 🔥 Strong Interview Answer

> "When troubleshooting IAM Identity Center access, I first determine whether the issue is authentication or authorization. If the user cannot log in, I investigate the identity and authentication configuration. If the user can log in but cannot access an account or perform an action, I check the account assignment, group membership, permission set, exact AWS action, resource, and applicable deny controls such as explicit denies or SCPs. I avoid giving Administrator access as a quick fix and follow least privilege."

---

# 🧩 Day 6 Troubleshooting Memory

```text
LOGIN PROBLEM
     ↓
Authentication

ACCOUNT MISSING
     ↓
Account Assignment

ACCESS DENIED
     ↓
Permission Set / Policy

TOO MUCH ACCESS
     ↓
Least Privilege

MANY USERS AFFECTED
     ↓
Group / Permission Set / Assignment

ONE USER AFFECTED
     ↓
User / Group / Assignment
```

---

# ✅ Day 6 Troubleshooting Goal

After completing Day 6, I should be able to troubleshoot common IAM Identity Center access problems and explain my approach clearly in a Cloud/DevOps interview.

My core troubleshooting model is:

```text
WHO?
 ↓
WHICH ACCOUNT?
 ↓
WHICH PERMISSION SET?
 ↓
WHAT ACTION?
 ↓
WHICH RESOURCE?
 ↓
IS IT ALLOWED?
 ↓
IS THERE A DENY?
 ↓
TEST AGAIN
```

**Key principle:**

> **Don't add permissions blindly. Find the exact access problem and fix it with the minimum required permission.**

