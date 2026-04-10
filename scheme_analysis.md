# Scheme Analysis Workflow

This file defines the exact step-by-step process to follow when a user provides a scheme document and/or transaction data for analysis. Follow every step in order. Do not skip steps.

---

## When to Use This Workflow

Trigger this workflow whenever the user provides:
- A scheme document (pasted text or a file path in the project), OR
- A scheme document + transaction / sales data

---

## Pre-Flight Check

Before starting analysis:

1. **Determine input mode** for each input:
   - Pasted text in chat → read directly from the conversation
   - File reference (e.g., "the file in resources/scheme.pdf") → use the Read tool to load the file before proceeding
2. **Identify data structure**: If transaction data is present, list all column names immediately.
3. **Check Output/ directory**: It must exist before saving. If it doesn't, create it.

---

## STEP 1 — Intake

- Read the full scheme document without skimming.
- If a data file is referenced, read it in full.
- Confirm to yourself what you have received. Do not begin analysis yet.

Internal checklist:
- [ ] Scheme document received: yes / no
- [ ] Transaction / sales data received: yes / no
- [ ] Data columns identified (if data present): [list them]
- [ ] All files loaded: yes / no

---

## STEP 2 — First-Pass Comprehension

Read the scheme document and form a working hypothesis. Write 5–8 internal bullet points covering:

- Scheme type (sales incentive / insurance commission / loan commission / hybrid)
- Who the eligible participants are (role, region, product line)
- What the primary metric(s) being measured are
- What the measurement period is (monthly / quarterly / annual)
- What outputs / payouts the scheme is designed to produce
- Any slab or tier structure you can see
- Any obvious ambiguities or gaps you spotted

This is your internal understanding only — do not present it to the user yet.

---

## STEP 3 — Clarification Gate

**THIS STEP IS MANDATORY. DO NOT SKIP. DO NOT PROCEED TO STEP 4 UNTIL COMPLETE.**

Review your first-pass comprehension. Identify every point that is ambiguous, missing, or has more than one valid interpretation.

### Always check for these triggers:

| Topic | Ask if unclear |
|-------|---------------|
| Slab calculation method | Is it RETRO (entire achievement re-rated) or INCREMENTAL (only the marginal portion above the threshold gets the new rate)? |
| Cap scope | Does the cap apply per-month, per-quarter, or for the entire plan period? |
| Accelerator type | Is the accelerator a multiplier applied ON TOP of the base rate, or does it REPLACE the base rate? |
| Stacking rules | Can the same transaction earn incentives under more than one rule simultaneously? |
| Hierarchy overrides | Does the manager's override REDUCE the subordinate's payout, or is it purely ADDITIVE? |
| Persistency basis | Is persistency calculated on active policy COUNT or active policy PREMIUM VALUE? |
| Clawback scope | Under what conditions is commission reversed (lapse, default, prepayment)? Over what look-back period? |
| Data completeness | If data is provided: does it represent ALL transactions for the scheme period, or is it a sample? |

### Question format (use this exactly):

```
Before I complete the analysis, I need to clarify a few points about the scheme:

**Q1 — [Topic]:** [Specific question]
  - Option A: [First interpretation]
  - Option B: [Second interpretation]
  *(Please choose one, or describe if neither fits)*

**Q2 — [Topic]:** [Specific question]
  - Option A: ...
  - Option B: ...
```

Post the questions, then **stop and wait** for the user's response.

### After the user responds:
- Incorporate their answers into your understanding.
- If new ambiguities arise from the answers, ask follow-up questions using the same format.
- Only proceed to Step 4 when ALL critical ambiguities are resolved.

### When there are no ambiguities:
State: *"The scheme appears complete and unambiguous. No clarifications needed. Proceeding with analysis."*
Then move directly to Step 4.

---

## STEP 4 — Variable Classification

### 4A — Input Variables

Scan the scheme document AND the data (if provided). List every field or column that arrives as a raw input — not computed, just given.

Produce a table:

| Variable Name | Source | Data Type | Description |
|---------------|--------|-----------|-------------|
| [name] | Scheme doc / Data column: [column name] | Text / Number / Date / Boolean | [what it represents] |

### 4B — Intermediate Variables

Identify every derived metric that must be computed before the final incentive can be calculated. Think through the full calculation chain backward from the output.

**Row-level (per-transaction) variables:**
Write formulas as:
```
Variable_Name = Field1 OPERATOR Field2
```
Example: `Net_Disbursement = Gross_Disbursement - Cancellations`

Produce a table:

| Variable Name | Type | Formula | Depends On |
|---------------|------|---------|------------|
| [name] | Row-level | [formula] | [input fields used] |

**Consolidated (per-agent or per-period) variables:**
Write formulas as:
```
Variable_Name = AGGREGATION(source_variable)
                WHERE [filter condition]
                GROUP BY [dimension]
```
Example:
```
Total_Revenue_Q1 = SUM(Revenue)
                   WHERE transaction_date BETWEEN Q1_start AND Q1_end
                   GROUP BY agent_id
```

Produce a table:

| Variable Name | Aggregation | Source Variable | Filter | Group By | Depends On |
|---------------|-------------|-----------------|--------|----------|------------|
| [name] | SUM / COUNT / MAX / MIN | [row-level var] | [condition] | [dimension] | [intermediate vars] |

### 4C — Output Variables

List every final payout or incentive amount the scheme produces.

| Output Variable | Description | Depends On | Formula | Cap | Floor |
|-----------------|-------------|------------|---------|-----|-------|
| [name] | [what it pays] | [intermediate vars used] | [formula] | [max value or "None"] | [min value or "None"] |

---

## STEP 5 — Slab and Rule Documentation

For every incentive rule in the scheme:

### Rule [N]: [Rule Name]
- **Applies when:** [condition under which this rule is active]
- **Calculation type:** Retro / Incremental / Flat / Multiplier
- **Cap:** [value, or "None"]
- **Floor:** [value, or "None"]
- **Exclusions:** [any transactions or participants excluded]

Slab table (if applicable):

| Slab # | From | To | Rate / Payout Amount | Notes |
|--------|------|----|---------------------|-------|
| 1 | 0 | [threshold] | [rate or amount] | |
| 2 | [threshold] | [next threshold] | [rate or amount] | |

---

## STEP 6 — Worked Example

**If transaction data is provided:**
- Pick 1–2 representative agents or cases from the data.
- Walk through the full calculation chain: input values → intermediate variables → output variables.
- Show every arithmetic step explicitly.
- Label the final output clearly.

**If no data is provided:**
- Construct a hypothetical example using plausible numbers that illustrate the scheme logic.
- State clearly: *"This is a hypothetical example — no actual data was provided."*

Format:
```
### Example: [Agent Name or "Agent A"]

Input values:
- [Variable]: [value]
- [Variable]: [value]

Step-by-step:
1. [Intermediate variable] = [formula with substituted values] = [result]
2. [Next variable] = ... = [result]
3. [Output variable] = ... = [final amount]

Final Outputs:
- [Output variable 1]: [amount]
- [Output variable 2]: [amount]
```

---

## STEP 7 — Assumptions and Gaps Log

List every assumption made during analysis:
`[ASSUMPTION] [description of what was assumed and why]`

List every unresolved gap or question that could not be clarified:
`[GAP] [description of what is missing and what impact it has on the analysis]`

---

## STEP 8 — Save Output File

### Filename Convention
```
Output/[descriptive_scheme_name]_analysis_[YYYY-MM-DD].md
```

Derive the scheme name from the document content. Use today's date. Replace spaces with underscores.

Examples:
- `Output/Q1_Sales_Commission_analysis_2026-04-09.md`
- `Output/DSA_Home_Loan_Commission_analysis_2026-04-09.md`

### Output File Structure

The saved file must contain all of the following sections:

```markdown
# Scheme Analysis: [Scheme Name]

**Date:** [YYYY-MM-DD]
**Scheme Type:** [Sales Incentive / Insurance Commission / Loan Commission / Hybrid]
**Source:** [filename or "pasted text"]

---

## 1. Scheme Summary

[3–5 sentences describing what the scheme is, who it applies to, and what it rewards]

**Eligible Participants:** [roles, geographies, product lines]
**Scheme Period:** [monthly / quarterly / annual]
**Primary Metric(s):** [e.g., Revenue, Disbursement Amount, Policy Count]

---

## 2. Input Variables

| Variable Name | Source | Data Type | Description |
|---------------|--------|-----------|-------------|

---

## 3. Intermediate Variables

### 3A. Row-Level Variables

| Variable | Formula | Depends On |
|----------|---------|------------|

### 3B. Consolidated Variables

| Variable | Aggregation | Source Variable | Filter | Group By |
|----------|-------------|-----------------|--------|----------|

---

## 4. Incentive Rules

### Rule [N]: [Rule Name]
- Applies when: ...
- Calculation type: Retro / Incremental / Flat
- Cap: ...
- Floor: ...

| Slab # | From | To | Rate / Amount | Notes |
|--------|------|----|---------------|-------|

---

## 5. Output Variables

| Output Variable | Description | Formula | Cap | Floor |
|-----------------|-------------|---------|-----|-------|

---

## 6. Worked Example

[Full step-by-step calculation for 1–2 agents or cases]

---

## 7. Assumptions and Gaps

### Assumptions Made
- [ASSUMPTION] ...

### Open Gaps
- [GAP] ...

---
*Generated by Claude Code — Scheme Analysis Agent*
```

### Save Procedure

1. Use the **Write tool** to save the file to the Output/ path.
2. After saving, confirm to the user:
   `"Analysis saved to: Output/[filename].md"`
3. Print a short 3–5 line summary in chat covering:
   - Scheme name and type
   - Number of input / intermediate / output variables identified
   - Whether any gaps remain open
4. Do NOT print the full analysis inline.
