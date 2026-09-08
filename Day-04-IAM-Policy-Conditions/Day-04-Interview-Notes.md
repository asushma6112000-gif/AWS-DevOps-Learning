# Day 4 — IAM Policy Conditions — Interview Notes

## 1. What is an IAM Policy Condition?

An IAM Policy Condition is used to make an IAM permission more specific.

It allows or denies an action only when certain conditions are satisfied.

For example, a policy can allow access only when:

* The request comes from a specific AWS Region
* The request comes from a specific IP address
* MFA is enabled
* A specific VPC endpoint is used
* A specific tag is present
* The request uses a particular transport/security condition

Example:

```json
"Condition": {
  "StringEquals": {
    "aws:RequestedRegion": "ap-south-1"
  }
}
```

This condition checks the AWS Region requested by the API call.

---

# 2. Why do we use IAM Policy Conditions?

We use conditions to implement more granular access control.

Without a condition:

```text
User
 ↓
Action
 ↓
Resource
```

With a condition:

```text
User
 ↓
Action
 ↓
Resource
 ↓
Condition
 ↓
Allow/Deny
```

Conditions help implement:

* Least privilege
* Security restrictions
* Location-based access
* MFA-based access
* Network-based restrictions
* Tag-based access
* Time-based access

---

# 3. What is `aws:RequestedRegion`?

`aws:RequestedRegion` is a global condition key.

It allows us to check the AWS Region requested by an API operation.

Example:

```json
"Condition": {
  "StringEquals": {
    "aws:RequestedRegion": "ap-south-1"
  }
}
```

In this example, the policy condition checks for:

```text
ap-south-1
```

which is the Mumbai AWS Region.

---

# 4. What did you do in the Day 4 lab?

I created an IAM user and attached a custom IAM policy containing a condition.

The policy allowed:

```text
s3:GetObject
```

on objects inside a specific S3 bucket.

The policy also contained:

```text
aws:RequestedRegion = ap-south-1
```

I then configured a separate AWS CLI profile and tested the permissions.

---

# 5. What AWS resources did you create?

### IAM User

```text
day4-policy-test-user
```

### IAM Policy

```text
Day4-S3-Region-Condition-Policy
```

### S3 Bucket

```text
sushma-day4-policy-conditions-2026
```

### AWS Region

```text
ap-south-1
```

### AWS CLI Profile

```text
day4
```

---

# 6. What permissions did your Day 4 policy provide?

The policy provided:

```text
s3:GetObject
```

It did not provide:

```text
s3:ListBucket
```

It also did not provide:

```text
s3:PutObject
s3:DeleteObject
```

Therefore, the policy followed the principle of least privilege.

---

# 7. Explain your Day 4 IAM policy.

The policy was:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnlyFromMumbaiRegion",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::sushma-day4-policy-conditions-2026/*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "ap-south-1"
        }
      }
    }
  ]
}
```

The policy has five important parts:

```text
Effect
Action
Resource
Condition
Version
```

### Effect

```text
Allow
```

The statement allows the specified action.

### Action

```text
s3:GetObject
```

Allows reading/downloading S3 objects.

### Resource

```text
arn:aws:s3:::sushma-day4-policy-conditions-2026/*
```

The `/*` means objects inside the bucket.

### Condition

```text
aws:RequestedRegion = ap-south-1
```

Adds an additional condition to the permission.

---

# 8. Why did `aws s3 ls` return AccessDenied?

Command:

```bash
aws s3 ls s3://sushma-day4-policy-conditions-2026 --profile day4
```

returned:

```text
AccessDenied
```

because the policy did not grant:

```text
s3:ListBucket
```

The policy only granted:

```text
s3:GetObject
```

This was expected.

---

# 9. What is the difference between `s3:ListBucket` and `s3:GetObject`?

### `s3:ListBucket`

Allows listing objects in a bucket.

Resource:

```text
arn:aws:s3:::bucket-name
```

### `s3:GetObject`

Allows reading/downloading an object.

Resource:

```text
arn:aws:s3:::bucket-name/*
```

Example:

```text
s3:ListBucket
        ↓
Bucket

s3:GetObject
        ↓
Object
```

---

# 10. Why did `GetObject` work when `ListBucket` failed?

Because the policy explicitly allowed:

```text
s3:GetObject
```

but did not allow:

```text
s3:ListBucket
```

AWS evaluates each API action separately.

Therefore:

```text
ListBucket → Denied
GetObject   → Allowed
```

This demonstrates fine-grained IAM permissions.

---

# 11. How did you test the IAM identity?

I used:

```bash
aws sts get-caller-identity --profile day4
```

This confirmed which IAM identity the AWS CLI was using.

The result showed:

```text
day4-policy-test-user
```

This is useful for troubleshooting credential and permission problems.

---

# 12. Why did you use an AWS CLI profile?

I created a separate profile:

```text
day4
```

This allowed me to explicitly use the Day 4 IAM user's credentials.

Example:

```bash
aws sts get-caller-identity --profile day4
```

and:

```bash
aws s3api get-object \
  --bucket sushma-day4-policy-conditions-2026 \
  --key day4-test.txt \
  day4-downloaded.txt \
  --region ap-south-1 \
  --profile day4
```

---

# 13. What happened when you used the default AWS CLI credentials?

Initially:

```bash
aws sts get-caller-identity
```

returned:

```text
InvalidClientTokenId
```

The default AWS CLI credentials were invalid.

I solved the problem by creating and configuring the Day 4 profile:

```bash
aws configure --profile day4
```

Then I verified it:

```bash
aws sts get-caller-identity --profile day4
```

---

# 14. What is `aws sts get-caller-identity` used for?

It tells us which AWS identity is being used for the current request.

It returns information such as:

```text
UserId
Account
Arn
```

It is very useful when troubleshooting:

```text
AccessDenied
InvalidClientTokenId
InvalidAccessKeyId
```

---

# 15. What is the principle of least privilege?

The principle of least privilege means giving an identity only the permissions it actually needs.

For example, if a user only needs to download an object:

```text
s3:GetObject
```

may be sufficient.

There is no need to give:

```text
s3:DeleteObject
s3:PutObject
s3:ListAllMyBuckets
```

unless those actions are actually required.

---

# 16. What is a Condition in an IAM policy?

A Condition is an additional rule that must be satisfied for a policy statement to apply.

Example:

```json
"Condition": {
  "StringEquals": {
    "aws:RequestedRegion": "ap-south-1"
  }
}
```

Conceptually:

```text
Action allowed
        AND
Condition satisfied
        ↓
Permission applies
```

---

# 17. What are IAM condition operators?

IAM provides different condition operators.

Common examples include:

```text
StringEquals
StringLike
StringNotEquals
StringNotLike
ArnEquals
ArnLike
IpAddress
NotIpAddress
Bool
DateGreaterThan
DateLessThan
NumericEquals
NumericGreaterThan
```

The correct operator depends on the type of value being evaluated.

---

# 18. What is `StringEquals`?

`StringEquals` checks whether a string value exactly matches the expected value.

Example:

```json
"Condition": {
  "StringEquals": {
    "aws:RequestedRegion": "ap-south-1"
  }
}
```

The expected value is:

```text
ap-south-1
```

---

# 19. What is the difference between Action and Condition?

### Action

Defines **what AWS operation** the principal can perform.

Example:

```text
s3:GetObject
```

### Condition

Defines **under what circumstances** the permission applies.

Example:

```text
aws:RequestedRegion = ap-south-1
```

So:

```text
Action    → What can be done?
Condition → Under what conditions?
```

---

# 20. What is the difference between Resource and Condition?

### Resource

Defines **which AWS resource** the permission applies to.

Example:

```text
arn:aws:s3:::my-bucket/*
```

### Condition

Defines **additional circumstances** under which the permission applies.

Example:

```text
aws:RequestedRegion = ap-south-1
```

Therefore:

```text
Resource  → Where?
Condition → Under what condition?
```

---

# 21. What is an explicit Deny?

An explicit Deny is a policy statement that explicitly denies an action.

Example:

```json
{
  "Effect": "Deny",
  "Action": "s3:DeleteObject",
  "Resource": "*"
}
```

An explicit Deny overrides an Allow.

General IAM evaluation principle:

```text
Explicit Deny
     ↓
Overrides Allow
```

---

# 22. Does an IAM Condition automatically deny every other action?

No.

The policy statement only controls the actions and resources specified by that statement.

For example, our policy explicitly allows:

```text
s3:GetObject
```

It does not grant:

```text
s3:ListBucket
```

Therefore, `ListBucket` is not allowed by this policy.

---

# 23. What is the difference between authentication and authorization?

### Authentication

Answers:

```text
Who are you?
```

Example:

```bash
aws sts get-caller-identity --profile day4
```

### Authorization

Answers:

```text
What are you allowed to do?
```

Example:

```text
s3:GetObject → Allowed
s3:ListBucket → Denied
```

---

# 24. What is an IAM User?

An IAM User is an AWS identity that can have credentials and permissions.

In this lab:

```text
day4-policy-test-user
```

was the IAM user.

The user had an access key used for AWS CLI authentication.

---

# 25. What is an IAM Policy?

An IAM Policy is a JSON document that defines permissions.

A policy specifies things such as:

```text
Effect
Action
Resource
Condition
```

Example:

```text
Allow
   ↓
s3:GetObject
   ↓
Specific S3 objects
   ↓
Condition
```

---

# 26. What is the IAM policy evaluation process?

At a high level:

```text
AWS Request
     ↓
Authentication
     ↓
Identify Principal
     ↓
Evaluate Policies
     ↓
Check Conditions
     ↓
Check Explicit Deny
     ↓
Allow or Deny
```

An explicit Deny takes precedence over an Allow.

---

# 27. Why is policy testing important?

A policy can look correct but still produce unexpected results.

Testing helps verify:

* The correct identity is being used
* The intended action is allowed
* Unnecessary actions are denied
* Conditions behave as expected
* Resource ARNs are correct

---

# 28. What commands did you use in Day 4?

### Check identity

```bash
aws sts get-caller-identity --profile day4
```

### Test bucket listing

```bash
aws s3 ls s3://sushma-day4-policy-conditions-2026 --profile day4
```

### Download object

```bash
aws s3api get-object \
  --bucket sushma-day4-policy-conditions-2026 \
  --key day4-test.txt \
  day4-downloaded.txt \
  --region ap-south-1 \
  --profile day4
```

### Verify downloaded file

```bash
cat day4-downloaded.txt
```

---

# 29. What did your final test prove?

The final test proved that:

```text
IAM identity → Correct
s3:GetObject → Allowed
s3:ListBucket → Denied
Object download → Successful
Downloaded file → Correct
```

This demonstrated fine-grained IAM permissions using a policy condition.

---

# 30. Explain your Day 4 project in an interview.

A good interview answer:

"I created an IAM user and attached a custom S3 policy containing an IAM Condition. The policy allowed `s3:GetObject` on objects inside a specific S3 bucket and included the `aws:RequestedRegion` condition for `ap-south-1`. I configured a dedicated AWS CLI profile for the user and verified the identity using STS. I tested S3 permissions and confirmed that object retrieval worked while bucket listing was denied because `s3:ListBucket` was not granted. This helped me understand IAM Conditions, least privilege, and permission troubleshooting."

---

# 31. Important Interview Questions — Quick Revision

### Q: What is an IAM Condition?

A: It adds additional rules that determine when a policy statement applies.

### Q: Why use Conditions?

A: To make permissions more granular and secure.

### Q: What is `aws:RequestedRegion`?

A: A global condition key used to evaluate the AWS Region requested by an API operation.

### Q: What does `s3:GetObject` do?

A: It allows reading/downloading an S3 object.

### Q: What does `s3:ListBucket` do?

A: It allows listing objects in an S3 bucket.

### Q: Why did ListBucket fail in your lab?

A: Because the policy did not grant `s3:ListBucket`.

### Q: Why did GetObject work?

A: Because the policy explicitly allowed `s3:GetObject` for the bucket's objects.

### Q: What does least privilege mean?

A: Giving only the permissions required to perform the required task.

### Q: What command verifies the AWS identity?

A:

```bash
aws sts get-caller-identity
```

### Q: What overrides an Allow?

A: An explicit Deny.

### Q: What is the difference between Action and Condition?

A: Action specifies what can be done; Condition specifies the circumstances under which the permission applies.

---

# 32. Day 4 Interview Key Points

Remember these:

```text
IAM Condition
      ↓
More granular permissions
      ↓
Action + Resource + Condition
      ↓
Least Privilege
      ↓
Test using AWS CLI
      ↓
Troubleshoot AccessDenied
```

Most important concepts:

```text
aws:RequestedRegion
StringEquals
s3:GetObject
s3:ListBucket
IAM Policy
IAM User
STS
AWS CLI Profile
Least Privilege
Explicit Deny
Policy Evaluation
```
i
