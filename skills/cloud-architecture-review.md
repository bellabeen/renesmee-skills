---
name: cloud-architecture-review
version: 1.0.0
skill_type: analysis
owner: renesmee
category: cloud-architecture
risk_level: medium
description: Operational playbook for evaluating cloud architectures (AWS, Azure, GCP, or Multi-cloud) against the Well-Architected Framework pillars, evaluating system trade-offs, scalability, and disaster recovery.

tags:
  - cloud-architecture
  - systems-design
  - well-architected
  - reliability
  - infrastructure-design

required_inputs:
  - architectural_diagram_or_spec
  - business_requirements_slas
  - technology_stack_constraints
  - projected_user_traffic

expected_outputs:
  - well_architected_gap_analysis
  - reliability_dr_assessment
  - cost_optimization_report
  - target_state_architecture_recommendation

authoritative_sources:
  - AWS Well-Architected Framework Whitepapers
  - Google Cloud Architecture Framework
  - Microsoft Azure Well-Architected Framework
  - Site Reliability Engineering (SRE) Handbook (Google)

last_reviewed: 2026-06-10
---

# Purpose

The purpose of this skill is to provide a structured methodology for auditing cloud-native and hybrid system architectures. It guides the agent in identifying architectural anti-patterns, single points of failure (SPOFs), scalability bottlenecks, and cost inefficiencies. It ensures that system recommendations explain clear technical trade-offs between performance, complexity, and expense.

# When To Use

Use this skill when:
1. Conducting architecture design reviews before migrating on-premise workloads to the cloud.
2. Reviewing existing cloud deployments to optimize performance, cost, or reliability.
3. Performing audit checks for major system upgrades or scaling events (e.g., preparing for peak shopping traffic).
4. Evaluating systems design diagrams or specifications during engineering design review phases.

# When NOT To Use

Do NOT use this skill when:
1. Conducting low-level code audits, runtime debugging, or dynamic security penetration tests.
2. Managing real-time incident responses (which requires incident triage and troubleshooting protocols).
3. Reviewing simple, single-instance static websites or hobby projects where standard multi-tier architectural overhead is unnecessary.

# Inputs

The agent must validate that the following inputs are provided:
1. `architectural_diagram_or_spec` (object/string): Structural diagrams, text descriptions, or IaC files representing the current or proposed cloud architecture.
2. `business_requirements_slas` (object): Defined availability targets (e.g., 99.9% uptime), Recovery Time Objective (RTO), Recovery Point Objective (RPO), and regulatory compliance needs.
3. `technology_stack_constraints` (array): Locked-in cloud providers, legacy systems to integrate, developer skills, or regulatory geographic requirements.
4. `projected_user_traffic` (object): Anticipated traffic patterns (e.g., peak requests per second, concurrent users, read/write ratio).

# Methodology

Every cloud architecture review must analyze the system across:
1. **Security Impact (Security & Compliance)**: Data encryption mechanisms (at rest and in transit), network isolation (VPCs/subnets), IAM permissions governance, and threat detection.
2. **Cost Impact (Cost Optimization)**: Resource utilization efficiency, use of managed/serverless options vs. self-hosted, storage lifecycle management, and compute sizing.
3. **Operational Complexity**: Deployment pipeline automation, logging/monitoring dashboards, centralized alerting, operational maintenance burdens, and infrastructure drift management.
4. **Scalability**: Stateless vs. stateful design, load balancing configurations, auto-scaling trigger thresholds, database read replica deployment, and content delivery network (CDN) utilization.
5. **Long-Term Maintainability**: Technical debt mitigation, loose coupling (microservices/event-driven vs. monoliths), documentation quality, and simplicity of upgrades/migrations.

## Decision Criteria: Architectural Pattern Selection
- **Monolithic vs. Microservices**: Choose microservices only if team boundaries require independent deployability or scaling profiles differ dramatically. Reject microservices for small teams (under 10 engineers) due to operational complexity.
- **Serverless vs. Containers**: Choose serverless (e.g., AWS Lambda, Cloud Run) for variable/bursty traffic with low run times (< 15 mins). Choose containers (e.g., ECS/EKS, GKE) for predictable, high-volume throughput to minimize cold starts and runtime cost overhead.
- **SQL vs. NoSQL Databases**: Choose SQL (e.g., PostgreSQL, RDS) for complex transactional relationships and strict ACID compliance. Choose NoSQL (e.g., DynamoDB, MongoDB) for high-frequency write operations, predictable low-latency queries, and flexible schemas.

# Process

## Step 1: System Mapping & Baseline Identification
1. Analyze the provided `architectural_diagram_or_spec`.
2. Reconstruct the data flow and system dependency graph.
3. Map each component to its role (e.g., Client, Ingress, Compute, Storage, Caching, Queueing).
4. Identify any missing parts of the system (e.g., "Configuration shows database backend but no backup storage/mechanism"). Document these missing gaps as uncertainty flags.

## Step 2: Well-Architected Framework Audit
1. Evaluate the system against the Well-Architected Framework pillars:
   - **Reliability**: How does the system handle component failures? (e.g., Multi-AZ, database replication).
   - **Performance Efficiency**: Is compute sizing matched to current load? Are CDNs and caches used effectively?
   - **Cost Optimization**: Are there idle resources, over-provisioned instances, or lack of auto-scaling?
   - **Operational Excellence**: Are runbooks, logs, metrics, and tracing (e.g., OpenTelemetry) configured?
   - **Security**: Is network traffic restricted between tiers? (e.g., database only accepting traffic from the application layer).

## Step 3: Reliability & Disaster Recovery (DR) Modeling
1. Identify all Single Points of Failure (SPOFs) in the architecture.
2. Calculate the theoretical system availability based on component dependencies:
   - Serial dependencies: $A_{system} = A_1 \times A_2$
   - Parallel redundant dependencies: $A_{system} = 1 - (1 - A_1)^2$
3. Contrast calculated limits with the target `business_requirements_slas` (RTO/RPO).
4. Outline DR strategies (e.g., Backup & Restore, Pilot Light, Warm Standby, Active-Active Multi-Region) depending on target metrics.

## Step 4: Scaling & Bottleneck Analysis
1. Analyze the system capacity under `projected_user_traffic`.
2. Locate potential bottlenecks (e.g., database lock contention, write limits, API rate limiting, queue limits).
3. Evaluate cache efficiency: recommend caching at the browser, CDN, API Gateway, application, or database layer (e.g., Redis).
4. Recommend stateless compute adjustments to ensure vertical/horizontal auto-scaling operates smoothly.

## Step 5: Target State Formulation & Trade-off Matrix
1. Design the target state architecture resolving all critical gaps.
2. Construct a trade-off matrix comparing the current state to the proposed state.
3. Itemize the cost, complexity, and operational risk of migrating to the target state.

# Risk Assessment

| Risk Category | Likelihood | Impact | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **Single Point of Failure (SPOF)** | Medium | Critical | Deploy multi-AZ databases; deploy autoscaling groups across multiple subnets. |
| **Database Connection Exhaustion** | High | High | Implement connection pooling (e.g., RDS Proxy) and database read replicas. |
| **Cost Overrun (Inefficient Sizing)** | High | Medium | Set up billing alarms; implement auto-stop schedules for non-production environments. |
| **Operational Blindspot** | Medium | High | Enforce centralized logging (e.g., CloudWatch, ELK) and setup real-time error alerting (e.g., Sentry). |
| **Stateful Scaling Bottlenecks** | Low | High | Decouple architectures using message queues (e.g., SQS, RabbitMQ) to handle load spikes asynchronously. |

# Validation Checklist

- [ ] Has every component in the system design been mapped to a specific role?
- [ ] Are all database backends configured with multi-AZ failover or equivalent replication?
- [ ] Does the DR strategy meet the business-mandated RTO and RPO limits?
- [ ] Have all single points of failure (SPOFs) been identified and documented?
- [ ] Has the cost impact of adding high-availability components (e.g., NAT Gateways, multi-region database replication) been calculated?
- [ ] Are caching strategies recommended before proposing database scale-ups?
- [ ] Has the team's operational capability been factored into the choice of service complexity (e.g., avoiding self-hosted Kubernetes for small teams)?

# Output Format

The agent must output the architectural review report in the following markdown structure, followed by a valid JSON payload containing the telemetry.

### Markdown Report Structure

```markdown
# Cloud Architecture Review: [System Name]

## Executive Summary
* **Target Availability SLA**: [Uptime Percentage]%
* **Evaluated Cloud Provider(s)**: [e.g., AWS, GCP]
* **DR Target Metrics**: RTO: [Time] | RPO: [Time]
* **Overall Architectural Health**: [Good / Moderate / High Risk]

## Architectural Gap Analysis
| Component | Risk Level | Description of Issue | Mitigation Recommendation |
| :--- | :--- | :--- | :--- |
| Database | High | Single instance database deployment (SPOF) | Configure Multi-AZ replication |
| Assets | Medium | Static assets served from compute instances | Migrate to S3/CloudFront CDN |

## Proposed Target State Architecture
### Design Changes
1. [Detail 1]
2. [Detail 2]

### Trade-Off Assessment
* **Cost Difference**: [Estimated monthly change in USD]
* **Complexity Level**: [Low / Medium / High]
* **Performance Gain**: [Uptime and Latency Impact]
```

### JSON Telemetry Output

```json
{
  "systemName": "string",
  "overallHealth": "GOOD" | "MODERATE" | "HIGH_RISK",
  "slaTargetMet": true | false,
  "drMetrics": {
    "rtoMinutes": 0,
    "rpoMinutes": 0
  },
  "issuesCount": {
    "critical": 0,
    "high": 0,
    "medium": 0,
    "low": 0
  },
  "componentsReviewed": [
    {
      "name": "string",
      "risk": "LOW" | "MEDIUM" | "HIGH" | "CRITICAL",
      "spof": true | false,
      "remediationComplexity": "LOW" | "MEDIUM" | "HIGH"
    }
  ]
}
```

# Guardrails

1. **Anti-Hallucination Guard**: Never invent SLA details or mock traffic loads if not explicitly provided. If traffic parameters are omitted, model recommendations assuming a baseline of 10 requests per second and state: "TRAFFIC_ASSUMPTION: Scalability modeled under baseline parameters."
2. **Managed Services Preference**: Always prioritize managed cloud services (e.g., RDS, ECS, Cloud Run) over self-hosted variants (e.g., Postgres on EC2) unless explicit cost constraints or licensing issues prevent doing so.
3. **No Multi-Region Overkill**: Do not recommend multi-region architectures unless the SLA requires >99.99% uptime or the target RTO is under 15 minutes, due to the high costs and operational overhead.

# References

* AWS Well-Architected Framework: [AWS Framework](https://aws.amazon.com/architecture/well-architected/)
* Google Cloud Architecture Framework: [Google Cloud Framework](https://cloud.google.com/architecture/framework)
* Site Reliability Engineering Book: [Google SRE](https://sre.google/sre-book/table-of-contents/)

# Improvement Opportunities

1. **IaC Code Scanners**: Integrate automatic validation against Terraform config scanning tools (e.g., tfsec, Checkov).
2. **Infrastructure Cost Calculators**: Embed programmatic access to Cloud Pricing APIs (e.g., AWS Price List API) to generate exact pricing delta estimates.
3. **Network Diagram Generation**: Generate Mermaid.js sequence and system flow diagrams directly representing the target architecture design.
