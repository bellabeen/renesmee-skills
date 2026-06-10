---
name: dlmm-analysis
version: 1.0.0
skill_type: analysis
owner: renesmee
category: defi-market-making
risk_level: medium
description: Comprehensive operational playbook for analyzing Dynamic Liquidity Market Maker (DLMM) pools, simulating liquidity shapes, estimating dynamic fees, and formulating yield-optimization strategies while managing impermanent loss risk.

tags:
  - defi
  - solana
  - market-making
  - liquidity-provision
  - meteora

required_inputs:
  - pool_address
  - token_pair_data
  - target_deposit_usd
  - LP_volatility_profile

expected_outputs:
  - bin_configuration_recommendation
  - yield_projection_sheet
  - risk_mitigation_strategy
  - rebalancing_playbook

authoritative_sources:
  - Meteora DLMM Official Documentation (docs.meteora.ag)
  - Solana FM / Birdeye API Data Schema
  - Kamino/Jupiter Liquidity Allocation Reports

last_reviewed: 2026-06-10
---

# Purpose

The purpose of this skill is to provide a structured, deterministic playbook for analyzing Dynamic Liquidity Market Maker (DLMM) pools. It instructs the agent on how to systematically evaluate pool health, analyze bin steps, simulate different liquidity shapes (Spot, Curve, Bid-Ask), model dynamic fee behavior, estimate impermanent loss under various price paths, and design operational rebalancing strategies.

# When To Use

Use this skill when:
1. Evaluating a new or existing DLMM pool for capital deployment.
2. Determining the optimal price range and bin shape for a specific token pair based on historical volatility.
3. Conducting post-mortem performance reviews of an active liquidity position.
4. Designing automated rebalancing logic or threshold-based alerts for market-making agents.
5. Recommending whether to deploy into a DLMM vs. a standard Concentrated Liquidity Market Maker (CLMM) or constant product (AMM V2) pool.

# When NOT To Use

Do NOT use this skill when:
1. Dealing with pegged assets (e.g., USDC-USDT) that do not benefit from dynamic fee adjustments, where a standard CLMM with a narrow, fixed band is more fee-efficient.
2. The target token pair lacks adequate historical data or reliable price oracle feeds.
3. The underlying token project exhibits high rug-pull risk, severe contract vulnerabilities, or centralized freeze authority, making any LP strategy secondary to security risks.
4. Long-term passive holding is desired without any capability or resources for active position management.

# Inputs

The agent must validate that the following inputs are provided:
1. `pool_address` (string): The on-chain address of the DLMM pool.
2. `token_pair_data` (object): Includes the symbols, decimals, and current market prices of Token X and Token Y.
3. `target_deposit_usd` (number): The total fiat value intended for deployment.
4. `LP_volatility_profile` (string): The risk tolerance and volatility outlook (`low-volatility-range-bound`, `high-volatility-directional`, `moderate-volatility-accumulation`).
5. `historical_lookback_days` (number, optional): The period for calculating average daily volume, volatility, and fee generation. Defaults to `30`.

# Methodology

Every DLMM analysis must evaluate:
1. **Security Impact**: Assess the underlying token contract capabilities (e.g., freeze authority, mint authority, transaction tax) and oracle health (e.g., Pyth or Switchboard update latency).
2. **Cost Impact**: Analyze swap fee overhead, gas/compute costs for SOL rebalancing, pool deposit/withdrawal slippage, and potential capital lockups.
3. **Operational Complexity**: Quantify the required rebalancing frequency (e.g., hourly, daily, weekly) based on bin step and price volatility, and outline monitoring systems.
4. **Scalability**: Estimate pool TVL capacity relative to the `target_deposit_usd` to prevent fee dilution (aim for LP share < 10% of total pool TVL).
5. **Long-Term Maintainability**: Plan for changes in market regimes, protocol migration paths (e.g., Meteora V1 to V2), and SDK/API schema updates.

## Decision Criteria

### Bin Step Selection
- **Bin Step < 0.1% (< 10 bps)**: High concentration, suitable for stablecoins or highly correlated pairs (e.g., SOL-mSOL).
- **Bin Step 0.1% - 0.5% (10 - 50 bps)**: Standard volatility pairs (e.g., SOL-USDC).
- **Bin Step > 0.5% (> 50 bps)**: High volatility, exotic, or meme tokens.

### Liquidity Shape Selection
- **Spot**: Even distribution. Use when market direction is uncertain and price is expected to wander widely across the range.
- **Curve**: Concentrated in the center. Use when volatility is low to moderate and the price is expected to mean-revert or stay range-bound.
- **Bid-Ask**: Concentrated at the outer edges. Use when executing a DCA strategy to buy low and sell high, or when expecting sudden price breakouts.

# Process

## Step 1: Data Gathering & Pool Health Validation
1. Query the blockchain ledger or indexer API (e.g., Birdeye, Jupiter API, Geyser) using the `pool_address`.
2. Extract the following metrics:
   - Current TVL (USD)
   - 24h Trading Volume (USD)
   - Base Fee Rate (basis points)
   - Current Bin Step
   - Active Bin ID and Price
   - Token reserve ratio (Token X vs. Token Y)
3. Check the tokens for safety flags (e.g., mint authority enabled, blacklist capacity). Reject the pool if security flags are triggered.
4. Verify the volume-to-liquidity (V/L) ratio:
   $$\text{V/L Ratio} = \frac{24\text{h Volume}}{\text{TVL}}$$
   If V/L < 0.1, flag the pool as low-efficiency.

## Step 2: Volatility Profile and Bin Step Analysis
1. Retrieve historical daily close prices for the token pair over the last `historical_lookback_days`.
2. Calculate the annualized volatility ($\sigma$) and average true range (ATR) to estimate price bounds.
3. Define the deployment price range:
   - Lower Bound: $P_{lower} = P_{current} \times (1 - z \times \frac{\sigma}{\sqrt{365}})$
   - Upper Bound: $P_{upper} = P_{current} \times (1 + z \times \frac{\sigma}{\sqrt{365}})$
   - (Where $z$ is the confidence multiplier, e.g., $1.96$ for 95% confidence).
4. Map the price bounds to bin indexes using the bin step $s$:
   - $\text{Bin Index Difference} = \frac{\ln(P_{upper} / P_{lower})}{\ln(1 + s)}$
5. If the required number of bins exceeds the protocol limit (e.g., 69 bins on Meteora), adjust the bounds or select a pool with a larger bin step.

## Step 3: Liquidity Shape Simulation
1. Simulate capital allocation across the calculated bins for the three shapes:
   - **Spot**: Equal weight $W_i = \frac{1}{N}$ for all $N$ bins.
   - **Curve**: Normal distribution centered on the active bin: $W_i \propto e^{-\frac{(i - i_{active})^2}{2\sigma_{bin}^2}}$.
   - **Bid-Ask**: Inverse normal distribution with peaks at $P_{lower}$ and $P_{upper}$.
2. Calculate the capital efficiency multiplier ($E$) for each shape compared to constant product:
   - $E = \frac{\text{Concentrated Liquidity in Active Bin}}{\text{Total Deposit}}$

## Step 4: Dynamic Fee and Yield Projection
1. Estimate dynamic fees using historical volume and volatility data:
   - Calculate average variable fee based on pool swap frequency and volatility factor.
   - Expected Daily Fees: $Fee_{est} = 24\text{h Volume} \times (BaseFee + VariableFee) \times \frac{\text{target\_deposit\_usd}}{\text{TVL} + \text{target\_deposit\_usd}}$
2. Project Impermanent Loss (IL) for each shape at $P_{lower}$ and $P_{upper}$:
   - For a single bin, IL is total if price moves completely out. Project aggregate IL across the distributed bin set.
3. Compute the Net Yield Indicator (NYI):
   - $\text{NYI} = \text{Projected Fees} - \text{Projected Impermanent Loss} - \text{Rebalance Costs}$

## Step 5: Risk Modeling & Rebalancing Strategy Definition
1. Formulate a dynamic rebalancing strategy using explicit triggers:
   - **Time-based**: Check position every $T$ hours (e.g., 4 hours).
   - **Threshold-based**: Rebalance if price moves out of active bins by more than $K$ bins.
   - **Inventory-based**: Rebalance if token composition shifts past a 90:10 ratio.
2. Outline the execution steps for rebalancing:
   - Withdraw remaining liquidity from the pool.
   - Swap excess token back to target ratio via aggregator (e.g., Jupiter) to minimize price impact.
   - Re-deposit into a new centered range using the simulated shape.

# Risk Assessment

| Risk Category | Likelihood | Impact | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **Impermanent Loss (IL)** | High | High | Concentrated ranges around high-yield pools; set strict stop-losses or out-of-range rebalance triggers. |
| **Toxic Flow** | Medium | Medium | Avoid pool pairs where volume is driven solely by arbitrageurs during one-way price dumps; prioritize organic volume pairs. |
| **Smart Contract Failure** | Low | High | Use audited protocols (e.g., Meteora); limit maximum capital exposure per single pool to 10% of total portfolio. |
| **Oracle Lag / Manipulation** | Low | Medium | Ensure pools use multi-source decentralized oracles (e.g., Pyth) with low heartbeat intervals. |
| **Rebalance Fee Bleed** | High | Low | Set a minimum rebalance threshold where projected yield improvement exceeds gas and swap fees by at least 3x. |

# Validation Checklist

- [ ] Is the token pair free of transfer-tax and blacklist vulnerabilities?
- [ ] Is the pool volume-to-liquidity (V/L) ratio greater than 0.1?
- [ ] Does the proposed strategy's TVL representation remain under 10% of total pool TVL?
- [ ] Are the lower and upper price bounds mathematically derived from historical volatility?
- [ ] Has the Impermanent Loss been simulated at the range boundaries?
- [ ] Is the Net Yield Indicator (NYI) positive after accounting for expected slippage and rebalancing costs?
- [ ] Are the threshold triggers for position rebalancing clearly defined?

# Output Format

The agent must output the analysis report in the following markdown structure, followed by a valid JSON payload containing the telemetry.

### Markdown Report Structure

```markdown
# DLMM Pool Analysis Report: [TOKEN_X]-[TOKEN_Y]

## Executive Summary
* **Pool Address**: `[Pool Address]`
* **Recommendation**: [Deploy / Monitor / Avoid]
* **Liquidity Shape**: [Spot / Curve / Bid-Ask]
* **Target Deposit**: $[Amount]
* **Expected Daily APR**: [Percentage]%

## Pool Health Indicators
| Metric | Value | Threshold Check |
| :--- | :--- | :--- |
| TVL | $[Amount] | [Pass/Fail] |
| 24h Volume | $[Amount] | [Pass/Fail] |
| V/L Ratio | [Ratio] | [Pass/Fail] |
| Bin Step | [Basis Points] | [Pass/Fail] |

## Configuration & Price Range
* **Current Price**: [Price]
* **Lower Bound**: [Price] ([Percent] below current)
* **Upper Bound**: [Price] ([Percent] above current)
* **Bin Count**: [Number] bins

## Simulation & Projections (30-Day Outlook)
* **Projected Fee Earnings**: $[Amount]
* **Worst-Case IL (Boundary)**: $[Amount] ([Percent] of deposit)
* **Estimated Net APR**: [Percentage]%

## Rebalancing Playbook
* **Rebalance Trigger**: [Criteria]
* **Slippage Tolerance**: [Percentage]%
```

### JSON Telemetry Output

```json
{
  "poolAddress": "string",
  "recommendedStrategy": {
    "shape": "spot" | "curve" | "bid_ask",
    "binStep": 0,
    "lowerBoundPrice": 0,
    "upperBoundPrice": 0,
    "numberOfBins": 0
  },
  "financials": {
    "projectedDailyApr": 0.0,
    "worstCaseIlPercentage": 0.0,
    "netYieldIndicatorUsd": 0.0
  },
  "operationalTriggers": {
    "rebalancePriceDeviationPercentage": 0.0,
    "checkIntervalHours": 0
  }
}
```

# Guardrails

1. **TVL Safeguard**: Never recommend deployment if the deposit size would exceed 15% of the pool's total TVL, as this dilutes yield and increases slippage risk during exit.
2. **Volatility Guard**: If the annualized volatility ($\sigma$) exceeds 300% (common in newly launched meme tokens), enforce the use of a `Spot` shape with a minimum width of 40 bins to prevent immediate range breakout.
3. **Fee Dilution Guard**: Do not deploy if the projected fee APR is less than 1.5x the standard lending rate or single-sided staking rate of the base asset.
4. **Oracle Safety**: If Pyth oracle status is invalid or latency exceeds 60 seconds, suspend all deployment recommendations.

# References

* Meteora DLMM Documentation: [Meteora Docs](https://docs.meteora.ag)
* Solana Web3.js SDK / Token Program Standards: [Solana Docs](https://docs.solana.com)
* Uniswap V3 concentrated liquidity mathematical model (comparison): [Uniswap V3 Whitepaper](https://uniswap.org/whitepaper.pdf)

# Improvement Opportunities

1. **Dynamic Shape Shifting**: Automatically shift between `Curve` and `Spot` based on real-time volatility indices (e.g., DVOL equivalents).
2. **Cross-Pool Hedging**: Integrate delta-hedging strategies where LPs borrow volatile assets from lending protocols (like Kamino or Marginfi) to hedge against impermanent loss.
3. **Jito-MEV Protection**: Implement Jito tip bundles for transaction submission to prevent front-running and sandwich attacks during rebalancing transactions.
