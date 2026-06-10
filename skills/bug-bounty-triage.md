---
name: bug-bounty-triage
version: 1.0.0
skill_type: analysis
owner: renesmee
category: vulnerability-management
risk_level: high
description: Operational playbook for intake, replication, validation, scoring (CVSS v3.1/v4.0), deduplication, and reward calculation of bug bounty vulnerability reports.

tags:
  - appsec
  - bug-bounty
  - triage
  - cvss
  - vulnerability-management

required_inputs:
  - submission_report
  - program_policy
  - internal_known_issues_log
  - asset_inventory_criticality

expected_outputs:
  - triage_status_decision
  - cvss_score_evaluation
  - proof_of_concept_verification_steps
  - bounty_reward_recommendation

authoritative_sources:
  - FIRST CVSS v3.1 & v4.0 Specification Documents
  - OWASP Top 10 API Security Risks & Web Application Risks
  - HackerOne Triage Guidelines / Bugcrowd VRT (Vulnerability Rating Taxonomy)

last_reviewed: 2026-06-10
---

# Purpose

The purpose of this skill is to standardize the intake and evaluation process of incoming bug bounty reports. It instructs the agent on how to verify proof-of-concepts, filter spam or out-of-scope reports, calculate precise severity scores using CVSS metrics, deduplicate findings against existing logs, and recommend fair bounty payouts based on asset criticality and business impact.

# When To Use

Use this skill when:
1. Triaging new vulnerability submissions from external security researchers on platforms like HackerOne, Bugcrowd, Intigriti, or via self-hosted security disclosures.
2. Assessing internal pentest reports that require classification and prioritization under similar framework methodologies.
3. Deciding on severity disputes raised by researchers regarding business impact versus technical score.

# When NOT To Use

Do NOT use this skill when:
1. Conducting generic automated vulnerability scanning (SAST/DAST) alerts, which require high-volume automated filtering rather than human-like report analysis.
2. Handling compliance/security questionnaire reviews from clients or partners.
3. Conducting code-level software engineering patches (though this playbook informs the patch priority).

# Inputs

The agent must validate that the following inputs are provided:
1. `submission_report` (object): Contains the researcher's report: Title, Description, Asset URL/Path, Vulnerability Type, Proof of Concept (PoC) steps, and impact claim.
2. `program_policy` (object/string): The public-facing or internal rules of the bug bounty program, including in-scope assets, out-of-scope vulnerabilities (e.g., rate-limiting without impact, DNSSEC missing), and bounty tables.
3. `internal_known_issues_log` (array): A list of open issues, past patches, and recently accepted risks to check for duplicates.
4. `asset_inventory_criticality` (object): A mapping of system assets to business criticality (e.g., `payment-gateway: Tier-1`, `marketing-blog: Tier-3`).

# Methodology

Every bug bounty triage decision must evaluate:
1. **Security Impact**: Verifiable technical impact on Confidentiality, Integrity, and Availability (CIA triad). Determine if the vulnerability allows lateral movement or privilege escalation.
2. **Cost Impact**: Direct cost of bounty payouts vs. cost of potential breach. Also calculate the developer hours required to remediate and deploy the patch.
3. **Operational Complexity**: Overhead of recreating the researcher's environment, performing clean-room reproduction, and coordinating with engineering teams for patch verification.
4. **Scalability**: Verify if the vulnerability pattern is present across other endpoints, subdomains, or microservices (systemic issue).
5. **Long-Term Maintainability**: Implementing regression tests in CI/CD pipelines to ensure the vulnerability is not reintroduced in future releases.

## Decision Tree: Triage Status
- **Accept**: Valid, in-scope, reproducible vulnerability with clear security impact.
- **Duplicate**: Valid, but matches an existing open bug, internal ticket, or previously submitted report.
- **Out of Scope**: Valid vulnerability but on an asset explicitly excluded in `program_policy` or of a type listed as out of scope (e.g., SPF record configuration).
- **Informative**: Valid technical finding but has no demonstrable security impact or exploitation likelihood (e.g., server banner disclosure).
- **Spam / Invalid**: Theoretical finding with no PoC, or generated via copy-pasting generic scanner outputs without verification.

# Process

## Step 1: In-Scope and Policy Alignment Validation
1. Cross-reference the submitted asset URL/Identifier in `submission_report` with the in-scope definitions in `program_policy`.
2. Check if the vulnerability type matches the excluded vulnerability list in the policy (e.g., CSRF on logout, missing security headers, rate limiting without business impact).
3. If out-of-scope, immediately transition to **Out of Scope** status. Document the specific policy clause to communicate to the researcher.

## Step 2: Proof-of-Concept (PoC) Replication
1. Analyze the replication steps provided by the researcher.
2. Determine if the replication requires specific headers, session tokens, or multi-user flows.
3. Draft a clean-room replication playbook.
4. *Anti-Hallucination Guard*: If the PoC is incomplete, non-reproducible, or relies on assumptions not backed by request/response logs, state: "REPRODUCTION_FAILED: Missing critical steps/payloads" and request additional details. Do not invent the exploitation path.

## Step 3: Deduplication Sweep
1. Search the `internal_known_issues_log` using keywords (e.g., endpoint name, vulnerability type, parameter).
2. Compare the root cause of the submission with active tickets:
   - If the vulnerability exploits the same endpoint and parameters via the same method, mark as **Duplicate**.
   - If the vulnerability exploits the same endpoint but via a completely different vector, evaluate if the root cause patch would fix both. If yes, classify as duplicate but note the novel vector.
3. Document the original ticket ID for duplication linking.

## Step 4: CVSS Scoring & Business Impact Analysis
1. Calculate the base CVSS score (using CVSS v3.1 or v4.0 calculator vectors):
   - Attack Vector (AV), Attack Complexity (AC), Privileges Required (PR), User Interaction (UI), Scope (S), Confidentiality (C), Integrity (I), Availability (A).
2. Adjust the score based on the `asset_inventory_criticality`:
   - If a High vulnerability is on a sandbox/development asset, reduce environmental metrics.
   - If a Medium vulnerability allows access to a production database, escalate to High/Critical.
3. Document the reasoning for any changes between the researcher's self-assessed severity and the final triaged severity.

## Step 5: Bounty Recommendation & Communication Drafting
1. Map the final CVSS severity rating (Low/Medium/High/Critical) to the bounty table in `program_policy`.
2. Apply modifiers:
   - *High-quality report modifier*: Increase reward (+10-20%) if the researcher provided complete remediation code, script tools, or video walk-throughs.
   - *Collateral risk modifier*: Decrease reward if the researcher violated rules (e.g., accessed other users' private data without permission, performed excessive load testing).
3. Draft the researcher communication response using a respectful, objective tone.

# Risk Assessment

| Risk Category | Likelihood | Impact | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **Researcher Dispute** | High | Low | Maintain transparent scoring based strictly on CVSS metrics and published policy guidelines. |
| **Public Disclosure Leak** | Low | High | Triage critical bugs within 24 hours and patch within agreed SLA (e.g., 7 days for Critical). |
| **Overpayment / Underpayment** | Medium | Medium | Implement peer review for payouts above a specific threshold (e.g., >$1,000 USD). |
| **Incomplete Patching** | High | High | Do not close tickets until QA/AppSec has rerun the reproduction PoC against the patched staging environment. |

# Validation Checklist

- [ ] Has the asset been verified as in-scope under the program policy?
- [ ] Has the vulnerability been reproduced successfully in a safe staging/sandbox environment?
- [ ] Did you check the duplicate logs for similar endpoints or parameters?
- [ ] Is the CVSS vector calculated mathematically according to the FIRST standard?
- [ ] Has the asset criticality tier been factored into the final impact rating?
- [ ] Has the payout recommendation been matched against the bounty table?
- [ ] Is the response template clear, polite, and free of emotional or defensive language?

# Output Format

The agent must output the triage report in the following markdown structure, followed by a valid JSON payload containing the telemetry.

### Markdown Report Structure

```markdown
# Bug Bounty Triage Report: [Report ID / Title]

## Triage Summary
* **Submission ID**: `[ID]`
* **Status Decision**: [Accept / Duplicate / Out of Scope / Informative / Spam]
* **Duplicate Link**: [Ticket ID / N/A]
* **Triaged Severity**: [Low / Medium / High / Critical]
* **Recommended Bounty**: $[Amount] USD

## Technical Evaluation
* **Vulnerability Type**: [e.g., IDOR, SQLi, SSRF]
* **Target Asset**: `[Domain / URL / Path]`
* **CVSS v3.1 Vector**: `CVSS:3.1/AV:...`
* **Base Score**: [Score] | **Environmental Score**: [Score]

## Reproduction Log
1. [Step 1 taken to reproduce]
2. [Step 2 taken to reproduce]
* **Result**: [Successfully Reproduced / Failed to Reproduce - Details]

## Tradeoff & Payout Analysis
* **Policy Range**: $[Min] - $[Max] USD
* **Adjustment Factors**: [e.g., +10% for custom remediation script, or -15% for aggressive scan noise]
* **Business Risk Context**: [Explain how this vulnerability impacts business operations or client trust]
```

### JSON Telemetry Output

```json
{
  "reportId": "string",
  "status": "ACCEPT" | "DUPLICATE" | "OUT_OF_SCOPE" | "INFORMATIVE" | "SPAM",
  "vulnerabilityType": "string",
  "asset": "string",
  "assetCriticality": "TIER_1" | "TIER_2" | "TIER_3",
  "scoring": {
    "cvssVector": "string",
    "baseScore": 0.0,
    "adjustedScore": 0.0,
    "severity": "LOW" | "MEDIUM" | "HIGH" | "CRITICAL"
  },
  "financials": {
    "recommendedBountyUsd": 0.0,
    "duplicateOfId": "string"
  }
}
```

# Guardrails

1. **Anti-Hallucination Guard**: Never confirm a vulnerability exists if you cannot reproduce it or explain the exact mechanical flow of the exploit. Never invent hypothetical business impact scenarios without explicit context.
2. **Researcher Privacy Guard**: Do not leak internal environment details, database schemas, server names, or developer names in the researcher-facing communications.
3. **SLA Adherence**: Flag any critical vulnerability reports to lead engineering teams immediately (within 2 hours of ingestion) to meet security response SLAs.

# References

* FIRST CVSS Calculator: [FIRST CVSS v3.1](https://www.first.org/cvss/calculator/3.1)
* Bugcrowd Vulnerability Rating Taxonomy: [Bugcrowd VRT](https://bugcrowd.com/vulnerability-rating-taxonomy)
* OWASP Top 10 Web Application Vulnerabilities: [OWASP Top 10](https://owasp.org/www-project-top-ten/)

# Improvement Opportunities

1. **Automated PoC Scripting**: Generate curl commands or python scripts automatically from the reproduction steps to allow engineering teams to easily verify patches.
2. **Duplicate Similarity Scoring**: Implement NLP-based text similarity comparison against previous reports to flag duplicates before human analysis begins.
3. **Integration with Jira/Linear**: Map accepted vulnerabilities directly to engineering team project boards with priority fields matched to the triage rating.
