# Day 5 — IAM Policy Evaluation + Explicit Deny

## Interview Notes

---

# PART 1 — WHAT I LEARNED

## 1. What is IAM Policy Evaluation?

IAM policy evaluation is the process AWS uses to determine whether an AWS request should be allowed or denied.

When a user, role, or application makes an AWS API request, AWS evaluates the applicable policies.

The basic concept is:

```text
AWS Request
     ↓
Who is making the request?
     ↓
What action are they trying to perform?
     ↓
Which resource are they accessing?
     ↓
Is there an Allow?
     ↓
Is there an Explicit Deny?
     ↓
Final Decision
```

The most important rule is:

```text
Explicit Deny
     ↓
Overrides
     ↓
Allow
```

---

# 2. What is an Implicit Deny?

An implicit deny means AWS does not find an applicable Allow permission.

For example:

```text
User
 ↓
s3:DeleteObject
 ↓
No Allow policy
 ↓
AccessDenied
```

There is no explicit Deny here.

The request is simply denied because the required permission was not allowed.

---

# 3. What is an Explicit Deny?

An explicit Deny is a policy statement with:

```json
"Effect": "Deny"
```

Example:

```json
{
  "Effect": "Deny",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::example-bucket/*"
}
```

If an applicable Allow and Explicit Deny both exist:

```text
Allow
 +
Explicit Deny
 ↓
DENY
```

Explicit Deny always wins.

---

# 4. Why is Explicit Deny Important?

Explicit Deny is commonly used as a security guardrail.

For example, an organization may allow developers to use S3 but explicitly deny access to sensitive buckets.

This prevents an Allow policy somewhere else from granting the restricted access.

---

# PART 2 — MY ACTUAL DAY 5 LAB

# What I Built

I built a practical AWS IAM Policy Evaluation lab using:

```text
IAM User
    ↓
IAM Policies
    ↓
S3 Bucket
    ↓
GetObject
```

First I created an Allow policy and verified that the user could download an S3 object.

Then I created an Explicit Deny policy for the same action and resource.

After attaching the Deny policy, I repeated the exact same test.

The first test succeeded.

The second test returned:

```text
AccessDenied
```

This demonstrated:

```text
Allow + Explicit Deny = DENY
```

---

# STEP 1 — Created the S3 Bucket

I created an S3 bucket for the Day 5 lab.

Bucket:

```text
sushma-day5-explicit-deny-2026-58321
```

Region:

```text
ap-south-1
```

The bucket was used only for testing IAM policy evaluation.

---

# STEP 2 — Created the Test File

On my Mac, I created:

```bash
echo "Day 5 Explicit Deny Test" > day5-test.txt
```

Then I verified the file:

```bash
cat day5-test.txt
```

Output:

```text
Day 5 Explicit Deny Test
```

---

# STEP 3 — Created the IAM User

I created an IAM user:

```text
day5-explicit-deny-user
```

The user was used to test S3 permissions.

I did not give the user broad permissions.

The purpose was to demonstrate controlled access.

---

# STEP 4 — Created the Allow Policy

I created this IAM policy:

```text
Day5-Allow-S3-GetObject
```

Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowGetObject",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::sushma-day5-explicit-deny-2026-58321/*"
    }
  ]
}
```

This policy allows:

```text
s3:GetObject
```

only for objects inside my Day 5 S3 bucket.

It does not allow:

```text
s3:PutObject
s3:DeleteObject
```

or access to other buckets.

This follows the principle of least privilege.

---

# STEP 5 — Attached the Allow Policy

I attached:

```text
Day5-Allow-S3-GetObject
```

directly to:

```text
day5-explicit-deny-user
```

At this point, the user had permission to read objects from the Day 5 bucket.

---

# STEP 6 — Configured AWS CLI

I configured an AWS CLI profile for the Day 5 IAM user:

```text
day5
```

I used the profile instead of the default AWS CLI credentials.

---

# STEP 7 — Verified the IAM Identity

I verified which identity the CLI was using:

```bash
aws sts get-caller-identity --profile day5
```

The command confirmed that the request was being made by:

```text
day5-explicit-deny-user
```

This was important because before troubleshooting permissions, I wanted to confirm:

```text
WHO is making the request?
```

---

# STEP 8 — Uploaded the Test Object

The Day 5 IAM user only had:

```text
s3:GetObject
```

It did not have:

```text
s3:PutObject
```

Therefore, I uploaded the test file through the S3 Console using an appropriate administrative identity.

The object was:

```text
day5-test.txt
```

The object content was:

```text
Day 5 Explicit Deny Test
```

---

# STEP 9 — Tested GetObject Before the Explicit Deny

I used:

```bash
aws s3api get-object \
  --bucket sushma-day5-explicit-deny-2026-58321 \
  --key day5-test.txt \
  day5-downloaded.txt \
  --region ap-south-1 \
  --profile day5
```

The command succeeded.

Then I checked the downloaded file:

```bash
cat day5-downloaded.txt
```

Output:

```text
Day 5 Explicit Deny Test
```

This proved that:

```text
IAM User
   ↓
Allow s3:GetObject
   ↓
S3 Object
   ↓
SUCCESS
```

I captured this as:

```text
01-allow-getobject-success.png
```

---

# STEP 10 — Created the Explicit Deny Policy

Next, I created another IAM policy:

```text
Day5-Deny-S3-GetObject
```

Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ExplicitDenyGetObject",
      "Effect": "Deny",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::sushma-day5-explicit-deny-2026-58321/*"
    }
  ]
}
```

This policy explicitly denies:

```text
s3:GetObject
```

for the Day 5 bucket.

---

# STEP 11 — Attached the Explicit Deny

I attached:

```text
Day5-Deny-S3-GetObject
```

to the same IAM user:

```text
day5-explicit-deny-user
```

Now the user had both:

```text
Day5-Allow-S3-GetObject
             +
Day5-Deny-S3-GetObject
```

So the policy evaluation became:

```text
Allow GetObject
       +
Explicit Deny GetObject
       ↓
      DENY
```

---

# STEP 12 — Tested the Same GetObject Operation Again

I ran the exact same command:

```bash
aws s3api get-object \
  --bucket sushma-day5-explicit-deny-2026-58321 \
  --key day5-test.txt \
  day5-downloaded.txt \
  --region ap-south-1 \
  --profile day5
```

This time the request failed.

AWS returned:

```text
AccessDenied
```

The error specifically indicated that the request was denied because of an:

```text
explicit deny in an identity-based policy
```

This was the main proof of the lab.

I captured this as:

```text
02-explicit-deny-accessdenied.png
```

---

# STEP 13 — Compared Both Tests

## Before Explicit Deny

```text
Allow GetObject
      ↓
GetObject
      ↓
SUCCESS
```

## After Explicit Deny

```text
Allow GetObject
      +
Explicit Deny GetObject
      ↓
ACCESS DENIED
```

Therefore:

```text
EXPLICIT DENY OVERRIDES ALLOW
```

---

# PART 3 — WHAT I LEARNED FROM THE LAB

The most important lesson from this lab is that simply having an Allow policy does not guarantee access.

AWS also evaluates applicable Deny statements.

The key rule is:

```text
Explicit Deny > Allow
```

I also learned how to troubleshoot permissions systematically.

I should first identify:

```text
WHO?
 ↓
WHAT ACTION?
 ↓
WHICH RESOURCE?
 ↓
IS THERE AN ALLOW?
 ↓
IS THERE AN EXPLICIT DENY?
```

---

# PART 4 — TROUBLESHOOTING INTERVIEW Q&A

## 1. What would you check if an AWS request returns AccessDenied?

**Answer:**

I would first identify which IAM identity is making the request.

I can use:

```bash
aws sts get-caller-identity
```

Then I would identify the failed AWS action and resource.

After that I would check the applicable IAM policies, resource policies, permissions boundaries, SCPs, and explicit Denies.

---

## 2. What is the most important IAM policy evaluation rule?

**Answer:**

The most important rule is:

```text
Explicit Deny overrides Allow.
```

Even if an Allow policy grants the action, an applicable Explicit Deny will cause the request to be denied.

---

## 3. What is the difference between implicit deny and explicit deny?

**Answer:**

Implicit deny happens when there is no applicable Allow permission.

Explicit deny happens when a policy specifically contains:

```json
"Effect": "Deny"
```

An Explicit Deny takes precedence over an Allow.

---

## 4. A user has Allow for `s3:GetObject`, but still gets AccessDenied. What would you check?

**Answer:**

I would check:

1. Correct IAM identity.
2. Correct action.
3. Correct resource ARN.
4. Identity policies.
5. S3 bucket policy.
6. Explicit Deny.
7. Permissions boundary.
8. SCP.
9. KMS permissions if encryption is involved.

---

## 5. How do you prove that an Explicit Deny is causing the problem?

**Answer:**

I would inspect the AccessDenied error and check the applicable policies.

If AWS identifies an Explicit Deny in the error, that is strong evidence.

I would also review the IAM policies attached to the identity and any resource-based policies or organizational controls.

---

## 6. Why shouldn't you immediately add AdministratorAccess when you get AccessDenied?

**Answer:**

Because that hides the actual permission problem and violates least privilege.

I would first identify the missing or denied action and grant only the required permission.

---

## 7. What if an IAM user has `s3:GetObject` but cannot upload?

**Answer:**

That is expected because:

```text
s3:GetObject
```

and:

```text
s3:PutObject
```

are different permissions.

I would check whether `s3:PutObject` is allowed.

---

## 8. What if an IAM user can upload but cannot delete?

**Answer:**

I would check:

```text
s3:DeleteObject
```

Upload and delete are separate S3 actions.

The user may have permission to upload but not delete.

---

## 9. What if the correct IAM policy is attached but access is still denied?

**Answer:**

I would check for another policy or control that contains an Explicit Deny.

I would also check:

```text
Bucket Policy
Permissions Boundary
SCP
KMS Policy
Resource ARN
```

---

## 10. How do you troubleshoot IAM systematically?

**Answer:**

I follow:

```text
WHO?
 ↓
WHAT ACTION?
 ↓
WHICH RESOURCE?
 ↓
ALLOW?
 ↓
EXPLICIT DENY?
 ↓
OTHER POLICY CONTROLS?
 ↓
TEST AGAIN
```

I don't immediately add more permissions.

---

# PART 5 — SCENARIO-BASED INTERVIEW Q&A

## Scenario 1 — Allow and Explicit Deny

**Interviewer:**

> An IAM user has an Allow policy for `s3:GetObject`, but another policy explicitly denies `s3:GetObject`. Can the user download the object?

**Answer:**

No.

The Explicit Deny overrides the Allow.

Therefore the final decision is:

```text
DENY
```

---

## Scenario 2 — Production AccessDenied

**Interviewer:**

> Your production application suddenly receives AccessDenied when accessing S3. What would you do?

**Answer:**

I would first verify which IAM identity the application is using.

Then I would identify the exact S3 action and resource that failed.

I would check the IAM policy, bucket policy, explicit Deny, permissions boundary, SCP, and KMS permissions if applicable.

I would not immediately add AdministratorAccess.

---

## Scenario 3 — Correct Role but AccessDenied

**Interviewer:**

> The EC2 instance has the correct IAM role, but S3 still returns AccessDenied. What would you check?

**Answer:**

I would verify the identity using:

```bash
aws sts get-caller-identity
```

Then I would check:

```text
Action
Resource ARN
IAM Allow
Bucket Policy
Explicit Deny
Permissions Boundary
SCP
KMS
```

---

## Scenario 4 — Developer Says "The IAM Role Is Broken"

**Interviewer:**

> The developer says the IAM role is broken because the application cannot delete an S3 object. What would you say?

**Answer:**

I would not assume the role is broken.

I would check whether the role actually has:

```text
s3:DeleteObject
```

The role could intentionally allow upload and download while denying deletion as part of least-privilege design.

---

## Scenario 5 — Read Works, Delete Fails

**Interviewer:**

> The application can read an S3 object but cannot delete it. Is that necessarily a problem?

**Answer:**

No.

`GetObject` and `DeleteObject` are separate permissions.

The application may have been intentionally granted read-only access.

I would verify the required permission before changing the policy.

---

## Scenario 6 — Explicit Deny Added

**Interviewer:**

> You add an Explicit Deny to a user that already has an Allow. What happens?

**Answer:**

The request is denied.

The Explicit Deny takes precedence over the Allow.

---

## Scenario 7 — SCP

**Interviewer:**

> An IAM policy allows an action, but the account is still unable to perform it. What organizational control could cause this?

**Answer:**

An AWS Organizations Service Control Policy, or SCP, could contain a restriction.

An SCP acts as a permissions guardrail at the organization, OU, or account level.

The IAM policy Allow alone does not override an applicable SCP restriction.

---

## Scenario 8 — Permissions Boundary

**Interviewer:**

> An IAM user has an Allow policy but cannot perform the action. What else would you check?

**Answer:**

I would check whether a permissions boundary limits the maximum permissions available to the identity.

---

## Scenario 9 — Bucket Policy

**Interviewer:**

> The IAM policy allows S3 access, but the request is still denied. What resource-based policy would you check?

**Answer:**

I would check the S3 bucket policy.

I would look for an Explicit Deny or another restriction involving the IAM principal, action, resource, conditions, or network context.

---

## Scenario 10 — Wrong Resource ARN

**Interviewer:**

> The IAM policy says Allow `s3:GetObject`, but the user still gets AccessDenied. What if the resource ARN is wrong?

**Answer:**

The Allow statement may not apply to the requested object.

I would compare the policy resource ARN with the actual S3 bucket and object ARN.

---

# PART 6 — HARDER INTERVIEW QUESTIONS

## 1. Does an Allow always mean access is granted?

**Answer:**

No.

An applicable Explicit Deny can override the Allow.

Other controls such as permissions boundaries and SCPs can also limit effective permissions.

---

## 2. Does Explicit Deny need to be in the same policy as the Allow?

**Answer:**

No.

The Allow and Explicit Deny can come from different applicable policies.

AWS evaluates all applicable policies.

If an applicable Explicit Deny exists, it overrides the Allow.

---

## 3. Why is least privilege important when designing IAM policies?

**Answer:**

Least privilege means giving only the permissions required to perform a task.

It reduces the impact of accidental or unauthorized actions.

For example, if an application only needs to download S3 objects, I would grant:

```text
s3:GetObject
```

instead of giving full S3 access.

---

## 4. How would you explain policy evaluation in one sentence?

**Answer:**

AWS evaluates the applicable permissions for a request, and an Explicit Deny overrides any Allow.

---

# PART 7 — MY DAY 5 PROJECT EXPLANATION FOR INTERVIEW

## 30-SECOND VERSION

> "In Day 5, I built an IAM policy evaluation lab using an S3 bucket and IAM user. First, I created an Allow policy for `s3:GetObject` and successfully downloaded an S3 object using AWS CLI. Then I created a second policy with an Explicit Deny for the same `GetObject` action and resource. After attaching the Deny policy, I repeated the same command and received AccessDenied. This demonstrated that an Explicit Deny overrides an Allow."

---

# 1-MINUTE VERSION

> "For my Day 5 AWS lab, I created an S3 bucket and an IAM user specifically for testing policy evaluation. I created a least-privilege Allow policy that permitted only `s3:GetObject` on objects inside my test bucket. I configured an AWS CLI profile for that user and verified the identity using `aws sts get-caller-identity`. I then downloaded an S3 object successfully, proving that the Allow policy was working. After that, I created another IAM policy containing an Explicit Deny for the same `s3:GetObject` action and resource and attached it to the same user. When I repeated the exact same S3 download command, AWS returned AccessDenied and indicated that the Explicit Deny caused the failure. This demonstrated the IAM policy evaluation rule that Explicit Deny overrides Allow."

---

# PART 8 — DAY 5 TROUBLESHOOTING FORMULA

```text
              AWS ACCESS PROBLEM
                       ↓
                     WHO?
                       ↓
              Which identity?
                       ↓
       aws sts get-caller-identity
                       ↓
                  WHAT ACTION?
                       ↓
             Which API failed?
                       ↓
                 WHICH RESOURCE?
                       ↓
                 Correct ARN?
                       ↓
                    ALLOW?
                       ↓
              Applicable policy?
                       ↓
              EXPLICIT DENY?
                       ↓
          Boundary / SCP / Resource
                 Policy / KMS
                       ↓
                 TEST AGAIN
```

---

# PART 9 — MOST IMPORTANT DAY 5 QUESTIONS

1. What is IAM policy evaluation?
2. What is an implicit deny?
3. What is an Explicit Deny?
4. What is the difference between Allow and Deny?
5. Does Explicit Deny override Allow?
6. How do you troubleshoot AccessDenied?
7. What does `aws sts get-caller-identity` do?
8. Why can a user have Allow and still receive AccessDenied?
9. What is least privilege?
10. What is a permissions boundary?
11. What is an SCP?
12. How can an S3 bucket policy affect access?
13. Why can `GetObject` work while `PutObject` fails?
14. Why can upload work while delete fails?
15. How do you verify the resource ARN?
16. How would you troubleshoot a production AccessDenied issue?
17. Why should you not immediately add AdministratorAccess?
18. Can Allow and Deny exist in different policies?
19. What happens when an applicable Explicit Deny exists?
20. Explain your Day 5 lab step-by-step.

---

# PART 10 — KEY COMMANDS FROM MY LAB

## Verify IAM Identity

```bash
aws sts get-caller-identity --profile day5
```

## Download S3 Object

```bash
aws s3api get-object \
  --bucket sushma-day5-explicit-deny-2026-58321 \
  --key day5-test.txt \
  day5-downloaded.txt \
  --region ap-south-1 \
  --profile day5
```

## Check Downloaded File

```bash
cat day5-downloaded.txt
```

## Create Test File

```bash
echo "Day 5 Explicit Deny Test" > day5-test.txt
```

---

# PART 11 — DAY 5 FINAL TAKEAWAY

The main concept I demonstrated was:

```text
ALLOW
  +
EXPLICIT DENY
  ↓
DENY
```

The most important interview statement is:

> "AWS evaluates the applicable policies for a request, and an Explicit Deny overrides an Allow."

My Day 5 lab gave me practical experience with:

* IAM policy evaluation
* Allow policies
* Explicit Deny
* Implicit deny
* S3 permissions
* IAM users
* AWS CLI profiles
* STS identity verification
* AccessDenied troubleshooting
* Resource ARN checking
* Least privilege
* Permissions boundaries
* SCPs
* S3 bucket policies
* KMS considerations

# ✅ DAY 5 COMPLETED

* IAM Policy Evaluation understood
* Implicit Deny understood
* Explicit Deny understood
* Allow vs Deny tested
* S3 GetObject Allow tested successfully
* Explicit Deny tested successfully
* AccessDenied observed
* AWS CLI identity verified
* Troubleshooting practiced
* Scenario-based interview questions prepared
* Actual project explanation prepared
* Interview answers prepared

