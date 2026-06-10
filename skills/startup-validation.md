---
name: startup-validation
version: 1.0.0
skill_type: decision
owner: renesmee
category: startup-operations
risk_level: medium
description: Operational playbook for validating new startup concepts, assessing product-market fit (PMF), estimating market sizing (TAM/SAM/SOM), calculating unit economics, and designing low-cost validation experiments.

tags:
  - product-management
  - venture-building
  - market-research
  - lean-startup
  - business-strategy

required_inputs:
  - business_concept_brief
  - target_customer_segment
  - primary_revenue_model
  - initial_funding_budget

expected_outputs:
  - market_opportunity_assessment
  - unit_economics_calculator
  - competitor_differentiation_matrix
  - lean_validation_experiment_playbook

authoritative_sources:
  - The Lean Startup (Eric Ries)
  - Y Combinator Startup Playbook & Library
  - Strategyzer Value Proposition Design & Business Model Generation

last_reviewed: 2026-06-10
---

# Purpose

The purpose of this skill is to provide a structured framework to evaluate the commercial viability, operational feasibility, and market potential of early-stage business concepts. It enables the agent to audit ideas objectively, prevent premature scaling, map out critical market size metrics, and recommend low-cost testing mechanisms before capital deployment.

# When To Use

Use this skill when:
1. Conducting initial market validation for a new product, service, or feature concept.
2. Reviewing startup pitches or investment briefs for venture capital, angel syndicates, or internal accelerators.
3. Evaluating pivot options for an existing business that has reached a growth plateau.
4. Planning go-to-market (GTM) strategies and pricing structures for newly incubated ideas.

# When NOT To Use

Do NOT use this skill when:
1. Analyzing mature companies with established product-market fit, which requires deep corporate finance auditing, M&A due diligence, or public stock valuation models.
2. Formulating macroeconomic policies or government regulatory frameworks.
3. Navigating complex non-profit, political, or public sector project structures, which operate under different non-commercial optimization parameters.

# Inputs

The agent must validate that the following inputs are provided:
1. `business_concept_brief` (string): The description of the product/service, core value proposition, and problem solved.
2. `target_customer_segment` (string): Demographics, psychographics, or professional attributes of the target customer.
3. `primary_revenue_model` (string): How the startup plans to make money (e.g., `SaaS-subscription`, `transactional-marketplace`, `freemium-ads`).
4. `initial_funding_budget` (number): The capital available to fund validation and initial development.
5. `geographic_scope` (string, optional): The target launch market (e.g., `United States`, `European Union`, `Southeast Asia`). Defaults to global-digital.

# Methodology

Every startup validation must analyze the opportunity across the following parameters:
1. **Security Impact (Compliance & Intellectual Property)**: Evaluate data privacy regulations (GDPR/CCPA), industry-specific compliance requirements (HIPAA for healthtech, SOC2/PCI for fintech), and IP protection strategies.
2. **Cost Impact (Unit Economics & Runway)**: Analyze estimated Customer Acquisition Cost (CAC), Lifetime Value (LTV), gross margins, operational burn rate, and projected runway.
3. **Operational Complexity**: Assess execution barriers, regulatory approvals, supply chain dependencies, hiring requirements, and initial tooling costs.
4. **Scalability**: Evaluate market sizing (TAM/SAM/SOM), distribution channels, network effects, and marginal costs of serving additional customers.
5. **Long-Term Maintainability**: Consider barriers to entry, competitive moat sustainability, technology stack longevity, and customer retention dynamics.

## Decision Criteria: Idea Viability Rating
- **High Viability**: Large growing market, clear unmet customer need, positive unit economic potential (LTV:CAC > 3:1), low regulatory hurdles, and simple validation pathways.
- **Medium Viability**: Moderate market size, competitive space requiring strong differentiation, complex unit economics requiring high scale, or moderate operational hurdles.
- **Low Viability**: Small stagnant market, severe customer acquisition headwinds, negative unit economic outlook, extreme regulatory blockers, or copycat model with no visible competitive advantage.

# Process

## Step 1: Problem-Solution Fit Analysis
1. Analyze the core customer pain point described in `business_concept_brief`.
2. Evaluate if the proposed solution directly addresses this pain point or if it is a "solution looking for a problem."
3. *Anti-Hallucination Guard*: Do not invent customer demand. If there is no documented evidence, industry reports, or user testimony confirming the problem, flag it as: "UNVERIFIED_DEMAND: No proof that target customers value solving this problem."

## Step 2: Market Sizing (TAM, SAM, SOM)
1. Calculate the market size using bottom-up estimation based on the `target_customer_segment` and `geographic_scope`:
   - **Total Addressable Market (TAM)**: Total market demand if 100% market share is achieved (e.g., total companies globally who could use this tool).
   - **Serviceable Addressable Market (SAM)**: The portion of the TAM targeted by the product's features and geographic reach.
   - **Serviceable Obtainable Market (SOM)**: The portion of the SAM that can be realistically captured in the next 1-3 years with current resources.
2. List all data sources and assumptions. State the level of uncertainty for each assumption.

## Step 3: Unit Economics & Financial Modeling
1. Model the primary revenue streams based on the `primary_revenue_model`.
2. Estimate the core metrics:
   - Average Order Value (AOV) or Average Revenue Per User (ARPU).
   - Cost of Goods Sold (COGS) including hosting, payment fees, or physical manufacturing costs.
   - Target Customer Acquisition Cost (CAC) and Lifetime Value (LTV): $\text{LTV} = \frac{\text{ARPU} \times \text{Gross Margin}}{\text{Churn Rate}}$.
3. Evaluate the cash runway: $\text{Runway (months)} = \frac{\text{initial\_funding\_budget}}{\text{Monthly Burn Rate}}$.

## Step 4: Competitive Landscape Assessment
1. Identify direct competitors (offering the same solution) and indirect competitors (offering alternative ways to solve the same problem).
2. Construct a differentiation matrix detailing features, pricing, target customers, and market positioning.
3. Identify the "Unfair Advantage" or "Moat" (e.g., proprietary tech, network effects, exclusive partnerships). If no moat exists, explain the tradeoffs of entering a highly competitive market.

## Step 5: Lean Validation Experiment Design
1. Identify the riskiest assumption (the one assumption that, if false, invalidates the entire business model).
2. Design a low-cost, fast validation experiment (e.g., Landing Page with sign-up form, smoke test, Wizard of Oz prototype, concierge test).
3. Define the precise success metric (e.g., >5% conversion rate on landing page, >50 pre-orders, >20 customer interviews).

# Risk Assessment

| Risk Category | Likelihood | Impact | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **No Market Need** | High | Critical | Execute landing page smoke tests and user interviews before writing any code. |
| **High CAC / Unprofitable Unit Economics** | High | High | Focus on organic referral loops and content marketing; optimize conversion funnels early. |
| **Regulatory Blockers** | Medium | Critical | Conduct regulatory audits (e.g., HIPAA, GDPR, FINRA) in Step 1. Consult specialized counsel. |
| **Tech/Implementation Risk** | Low | Medium | Build a Minimum Viable Product (MVP) using no-code tools or existing frameworks. |
| **Competitor Retaliation** | Medium | Medium | Target a hyper-specific, underserved niche before expanding to broader markets. |

# Validation Checklist

- [ ] Has the core customer pain point been verified via external evidence?
- [ ] Is the TAM/SAM/SOM calculation bottom-up rather than top-down estimations?
- [ ] Has the target LTV:CAC ratio been modeled at 3:1 or better?
- [ ] Are the COGS estimates inclusive of all hidden platform fees, hosting, and support costs?
- [ ] Has the competitor analysis identified at least 3 direct or indirect competitors?
- [ ] Is the validation experiment budget within the initial funding limit?
- [ ] Are the success metrics for the validation experiment quantifiable and time-bound?

# Output Format

The agent must output the startup validation report in the following markdown structure, followed by a valid JSON payload containing the telemetry.

### Markdown Report Structure

```markdown
# Startup Validation Report: [Project Name / Concept]

## Executive Summary
* **Concept Classification**: [e.g., SaaS, B2C Marketplace, D2C]
* **Target Launch Market**: [Geographic Region]
* **Viability Rating**: [High / Medium / Low]
* **Riskiest Assumption**: [Detail the single point of failure assumption]

## Market Sizing & Opportunity
* **Total Addressable Market (TAM)**: $[Amount] / [Count] users
* **Serviceable Addressable Market (SAM)**: $[Amount] / [Count] users
* **Serviceable Obtainable Market (SOM)**: $[Amount] / [Count] users
* **Market Sizing Methodology & Notes**: [Explain bottom-up calculations]

## Financial & Unit Economics Outlook
| Metric | Target Estimate | Notes / Assumptions |
| :--- | :--- | :--- |
| ARPU | $[Amount]/mo | [Pricing Assumption] |
| CAC | $[Amount] | [Marketing Channel Cost] |
| Gross Margin | [Percentage]% | [Hosting & Support Deductions] |
| LTV:CAC | [Ratio] | [Target Churn assumption] |
| Monthly Burn | $[Amount]/mo | [Team & Operations estimate] |
| Runway | [Number] months | [Based on Initial Budget] |

## Lean Validation Experiment Playbook
### Experiment: [Experiment Type - e.g., Smoke Test Landing Page]
* **Hypothesis**: "If we display [Value Proposition], then [Target Segment] will convert at [Target Rate]."
* **Execution Plan**:
  1. [Step 1]
  2. [Step 2]
* **Required Budget**: $[Amount]
* **Success Criteria**: [e.g., 8% email sign-up rate from 1,000 unique visits within 14 days]
```

### JSON Telemetry Output

```json
{
  "conceptName": "string",
  "viabilityRating": "HIGH" | "MEDIUM" | "LOW",
  "marketSizing": {
    "currency": "USD",
    "tam": 0.0,
    "sam": 0.0,
    "som": 0.0
  },
  "financials": {
    "projectedArpu": 0.0,
    "targetCac": 0.0,
    "grossMarginPercentage": 0.0,
    "ltvToCacRatio": 0.0,
    "projectedRunwayMonths": 0.0
  },
  "validationExperiment": {
    "type": "string",
    "riskiestAssumption": "string",
    "successMetric": "string",
    "estimatedCostUsd": 0.0
  }
}
```

# Guardrails

1. **Anti-Hallucination Guard**: Never invent TAM numbers or business valuations. If industry research is unavailable, state the lack of data and formulate a proxy comparison (e.g., comparing to a adjacent market size).
2. **Founder Bias Mitigation**: Never give a positive rating to a project simply because the founder has high enthusiasm. Ground all ratings strictly in unit economics, market size, and replication outcomes.
3. **No-Code First Rule**: If the budget is under $10,000, do not recommend custom software development. Enforce validation using existing third-party landing page builders, automation platforms (e.g., Zapier), or off-the-shelf templates.

# References

* Y Combinator Startup Library: [YC Library](https://www.ycombinator.com/library)
* Strategyzer Business Model Canvas: [Strategyzer Tool](https://www.strategyzer.com/canvas/business-model-canvas)
* The Lean Startup Principles: [Lean Startup](http://theleanstartup.com/principles)

# Improvement Opportunities

1. **GTM Ad Campaign Mockups**: Automatically generate ad copy and target parameters for validation experiments (e.g., Meta or LinkedIn Ads target groups).
2. **Dynamic Runway Sensitivity Analysis**: Implement a sensitivity matrix showing runway variations based on CAC increases or lower customer retention rates.
3. **AI Persona Simulation**: Simulate initial feedback by running the value proposition against synthetic target buyer profiles built from industry demographic data.
