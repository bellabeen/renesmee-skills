---
name: aws-security-review
version: 1.0.0
skill_type: analysis
owner: renesmee
category: cloud-security
risk_level: high
description: A structured operational playbook for conducting security audits on AWS environments, analyzing IAM policies, network configurations, data encryption settings, and logging coverage against the CIS AWS Foundations Benchmark.

tags:
  - aws
  - security
  - iam
  - compliance
  - audit

required_inputs:
  - aws_configuration_data
  - compliance_framework
  - environment_criticality

expected_outputs:
  - security_findings_report
  - iam_remediation_plan
  - network_hardening_recommendations
  - audit_telemetry_json

authoritative_sources:
  - CIS AWS Foundations Benchmark
  - AWS Security Pillar - Well-Architected Framework
  - AWS Security Best Practices Guide

last_reviewed: 2026-06-10
---

# Purpose

The purpose of this skill is to establish a rigorous, repeatable methodology for conducting security reviews of AWS infrastructure. It guides the agent through identifying IAM misconfigurations, exposed network endpoints, unencrypted data, and insufficient auditing/logging, while evaluating tradeoffs between security posture, cost, and operational overhead.

# When To Use

Use this skill when:
1. Conducting pre-deployment security reviews of AWS CloudFormation, Terraform, or CDK infrastructure-as-code (IaC) templates.
2. Auditing live AWS environments using configuration dumps (e.g., from AWS CLI, AWS Config, or Security Hub).
3. Triaging compliance status prior to external SOC2, ISO 27001, or PCI-DSS audits.
4. Performing security assessments during mergers/acquisitions or onboarding legacy AWS accounts.

# When NOT To Use

Do NOT use this skill when:
1. Conducting active penetration testing or vulnerability scanning (e.g., dynamic network port scanning, exploit execution).
2. Auditing non-AWS cloud platforms (Azure, GCP, OCI) which require different service-specific knowledge and security benchmarks.
3. Reviewing application-level software vulnerabilities (e.g., SQL injection, XSS) which fall under application security or code review scopes.

# Inputs

The agent must validate that the following inputs are provided:
1. `aws_configuration_data` (object/string): JSON/YAML configuration dumps, IaC templates, or IAM policy definitions representing the target AWS resources.
2. `compliance_framework` (string): The benchmark to audit against (e.g., `CIS-AWS-Benchmark-v2.0.0`, `PCI-DSS`, `AWS-Foundational-Security-Best-Practices`).
3. `environment_criticality` (string): The environment classification (`production`, `staging`, `development`, `sandbox`).
4. `target_regions` (array, optional): List of AWS regions scoped for the review. Defaults to all active regions.

# Methodology

Every AWS security review must evaluate findings across the following dimensions:
1. **Security Impact**: Severity of the vulnerability (Critical, High, Medium, Low) and potential blast radius if exploited.
2. **Cost Impact**: Direct cost of remediation (e.g., enabling AWS Config, VPC Flow Logs, GuardDuty, or KMS key rotation).
3. **Operational Complexity**: Overhead of implementing controls (e.g., impact of IAM restrictions on developer velocity, downtime for network changes).
4. **Scalability**: Ability of security controls to adapt as resources scale (e.g., using AWS Organizations Service Control Policies vs. individual IAM policies).
5. **Long-Term Maintainability**: Drift detection capabilities and integration with automated compliance checks (e.g., AWS Security Hub, AWS Config Rules).

## Decision Framework: Severity Classification
- **Critical**: Remote code execution possibilities, public credentials/secrets exposure, or unrestricted public access to highly sensitive database/storage resources (e.g., S3, RDS) in production.
- **High**: Wildcard (`*`) administrative IAM policies, unencrypted data at rest in production, or public management ports (SSH/RDP) open to the internet.
- **Medium**: Missing multi-factor authentication (MFA) for console users, access keys older than 90 days, or incomplete log configuration (e.g., CloudTrail not enabled in all regions).
- **Low**: Lack of resource tagging, minor configuration drifts, or missing description fields in Security Groups.

# Process

## Step 1: Configuration Ingestion & Validation
1. Parse the `aws_configuration_data`. Ensure it contains legible JSON/YAML structure.
2. Validate that the input contains sufficient information to execute the review (e.g., IAM policies, Security Group rules, KMS settings). If data is missing or truncated, state the uncertainty and list the missing files/configs.
3. Classify target resources by service type (e.g., IAM, EC2/VPC, S3, RDS, CloudTrail).

## Step 2: Identity & Access Management (IAM) Audit
1. Audit all IAM Users, Roles, and Groups:
   - Identify policies containing `"Effect": "Allow"` and `"Action": "*"` or `"Resource": "*"`.
   - Inspect cross-account trust relationships for unauthorized external entity access.
   - Detect long-lived credential configurations (e.g., active access keys older than 90 days, console users without MFA).
2. Evaluate tradeoffs of restricting policies:
   - *Security Gain*: Restricts unauthorized access and reduces blast radius.
   - *Operational Risk*: High risk of breaking production workflows. Suggest dry-run testing or AWS Access Analyzer logs to verify actual usage before restricting.

## Step 3: Network Security & Ingress/Egress Audit
1. Analyze VPC Security Groups, Network ACLs, and Route Tables:
   - Scan for inbound rules allowing access from `0.0.0.0/0` or `::/0` on sensitive ports (e.g., 22 (SSH), 3389 (RDP), 3306 (MySQL), 5432 (Postgres)).
   - Inspect Internet Gateway attachments and public subnets to verify that database or internal application servers are not exposed directly.
2. Evaluate tradeoffs of security group restriction:
   - *Security Gain*: Prevents brute-force attacks and network scanning.
   - *Operational Complexity*: Requires engineers to configure VPN, AWS Client VPN, or Systems Manager Session Manager for access, increasing overhead.

## Step 4: Data Protection & Encryption Audit
1. Verify encryption-at-rest and in-transit configurations:
   - Check S3 bucket policies for `"aws:SecureTransport": "true"` enforcement.
   - Audit S3 buckets, RDS databases, DynamoDB tables, and EBS volumes for encryption status.
   - Verify if default AWS-managed KMS keys are used instead of Customer Managed Keys (CMKs) where separation of duties is required.
2. Evaluate tradeoffs:
   - *Security Gain*: Protects data from physical theft or unauthorized internal snooping.
   - *Cost Impact*: CMKs carry a flat cost ($1 per key/month) plus API request fees. Encryption/decryption overhead is negligible for modern systems but should be budgeted.

## Step 5: Logging, Monitoring, & Detection Audit
1. Assess audit trail configuration:
   - Check if AWS CloudTrail is enabled globally across all regions, with log file validation turned on and logs stored in an encrypted S3 bucket.
   - Audit GuardDuty, Security Hub, and AWS Config deployment status.
   - Check if VPC Flow Logs are enabled for public or sensitive subnets.
2. Evaluate tradeoffs:
   - *Cost Impact*: High-volume VPC Flow Logs or heavy AWS Config usage can dramatically increase the monthly AWS bill. Recommend selective logging for critical subnets only.

# Risk Assessment

| Risk Category | Likelihood | Impact | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **IAM Credential Leak** | High | Critical | Enforce short-lived STS tokens, IAM Roles for EC2, and disable root account access keys. |
| **Public Data Exposure** | Medium | High | Implement S3 Block Public Access at the account level; execute automated drift monitoring. |
| **Compliance Penalty** | Medium | Medium | Map architectural controls directly to CIS benchmarks; run continuous AWS Config rules. |
| **Operational Downtime** | Low | High | Never modify production IAM policies or security groups without auditing access logs (Access Analyzer / VPC Flow Logs) first. |

# Validation Checklist

- [ ] Are all IAM console users configured with Multi-Factor Authentication (MFA)?
- [ ] Is there zero usage of root access keys?
- [ ] Are there no Security Groups allowing inbound `0.0.0.0/0` traffic to database ports?
- [ ] Are all S3 buckets encrypted at rest, and is public access blocked at the account/bucket level unless explicitly required?
- [ ] Is AWS CloudTrail enabled in all active regions and configured with log file validation?
- [ ] Have the cost implications of logging and monitoring remediations been calculated?
- [ ] Has verification of IAM policy usage been done via Access Analyzer before recommending deletion?

# Output Format

The agent must output the security review in the following markdown structure, followed by a valid JSON payload containing the telemetry.

### Markdown Report Structure

```markdown
# AWS Security Review Report: [Account ID / Project Name]

## Executive Summary
* **Environment Criticality**: [Production/Staging/Development]
* **Target Compliance Benchmark**: [Benchmark Name]
* **Overall Security Score**: [A/B/C/D/F] (Based on critical findings count)
* **Summary of Findings**: [Brief narrative]

## Security Findings Matrix
| ID | Service | Severity | Finding Description | Remediation Cost | Complexity |
| :--- | :--- | :--- | :--- | :--- | :--- |
| SEC-001 | IAM | Critical | Wildcard Admin privileges on Role | Negligible | High |
| SEC-002 | S3 | High | Unencrypted S3 Bucket containing PII | Low | Low |

## Detailed Findings & Remediations
### [SEC-XXX]: [Finding Title]
* **Severity**: [Critical/High/Medium/Low]
* **Resource Identifier**: `[Resource ARN / ID]`
* **Vulnerability Description**: [Explain what the security risk is and how it violates the benchmark]
* **Remediation Steps**:
  1. [Step 1]
  2. [Step 2]
* **Tradeoff Analysis**:
  * *Security Benefit*: [Detail]
  * *Operational Impact*: [Detail]
  * *Cost Estimate*: [Detail]
```

### JSON Telemetry Output

```json
{
  "accountId": "string",
  "reviewDate": "YYYY-MM-DD",
  "overallScore": "string",
  "findingsSummary": {
    "total": 0,
    "critical": 0,
    "high": 0,
    "medium": 0,
    "low": 0
  },
  "findings": [
    {
      "id": "SEC-001",
      "service": "string",
      "severity": "CRITICAL" | "HIGH" | "MEDIUM" | "LOW",
      "resourceId": "string",
      "ruleId": "string",
      "description": "string",
      "remediation": {
        "steps": ["string"],
        "costUsd": 0.0,
        "complexity": "LOW" | "MEDIUM" | "HIGH"
      }
    }
  ]
}
```

# Guardrails

1. **Anti-Hallucination Guard**: Never assume resources exist or are misconfigured without direct config proof. If the configuration dump is incomplete, state: "INSUFFICIENT_EVIDENCE: Cannot verify configuration status for [Service]."
2. **Production Safety Guard**: Do not recommend remediation commands that immediately delete active IAM roles, users, or credentials without explicitly instructing the user to verify usage history first.
3. **KMS Lockout Protection**: When recommending KMS key policy changes, always verify that the policy maintains root account access or administrator access to prevent permanent key lockout.

# References

* CIS AWS Foundations Benchmark: [CIS Benchmarks](https://www.cisecurity.org/benchmark/amazon_web_services)
* AWS IAM Security Best Practices: [AWS IAM Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
* AWS Security Hub User Guide: [AWS Security Hub](https://docs.aws.amazon.com/securityhub/)

# Improvement Opportunities

1. **Terraform Integration**: Enhance the playbook to automatically output remediation blocks in Terraform configuration code.
2. **Automated IAM Least-Privilege Generation**: Incorporate Access Analyzer access logs to automatically output a minimal IAM policy JSON.
3. **Cost Optimization Rules**: Pair security findings with cost-saving steps (e.g., deleting idle Elastic IPs identified during Security Group sweeps).
