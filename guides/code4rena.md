# Code4rena Judging Criteria

Official Code4rena judging guidelines for Wardens. Source: [Code4rena Documentation](https://docs.code4rena.com)

## Table of Contents

1. [Submission Requirements](#submission-requirements)
2. [Severity Criteria](#severity-criteria)
3. [Invalid Issue Categories](#invalid-issue-categories)
4. [Special Cases](#special-cases)
5. [PoC Requirements](#poc-requirements)
6. [QA Report Guidelines](#qa-report-guidelines)

---

## Submission Requirements

### Mandatory Elements

- [ ] Submitted before the deadline (NO late submissions accepted)
- [ ] Uses correct submission form
- [ ] Follows correct format
- [ ] Describes location and potential impact
- [ ] Offers detailed reproduction steps (coded PoC encouraged)
- [ ] Not a known issue (check audit README)
- [ ] Written in English

### Submission Types

| Type | How to Submit |
|------|---------------|
| **High/Medium** | Individually, one per submission |
| **Low/Governance** | Single QA report per warden |

### Good Citizenship Required

- Contribute more value than you take
- Gaming the system = rule violation
- Spam submissions (including LLM-generated nonsense) = account suspension
- Judges may deem wardens ineligible based on behavior/net quality

---

## Severity Criteria

### Risk Rating: 3 - High

**Assets can be stolen/lost/compromised:**
- Directly, OR
- Indirectly with valid attack path (no hand-wavy hypotheticals)

### Risk Rating: 2 - Medium

**Assets not at direct risk, but:**
- Protocol function or availability could be impacted, OR
- Value leak with hypothetical attack path + stated assumptions + external requirements

### QA (Low Risk) - Includes

**Low Risk:**
- Assets not at risk
- State handling issues
- Function incorrect as to spec
- Issues with comments

**Governance/Centralization:**
- Centralization risks
- Systemic risks
- Admin privileged functions

**Note:** Informational findings (code style, clarity, syntax, versioning) are discouraged.

---

## Severity Decision Guide

### Loss of Fees

Evaluated similar to loss of capital:
- Dust amounts (rounding errors, marginal variations) → QA/Low
- Real amounts → depends on conditions/likelihood

### Loss of Yield

**Matured yield** = loss of capital:
- Dust amounts → QA/Low
- Real amounts → depends on conditions

**Unmatured yield or yield in motion** = capped at Medium

---

## Invalid Issue Categories

### Always Invalid (or QA at best)

| Category | Severity Cap | Notes |
|----------|--------------|-------|
| **Approve/safeApprove front-run** | Invalid (informational) | NOT a valid vulnerability |
| **Protocol doesn't support CryptoPunks** | Informational | Refactoring only |
| **Conditional on user mistake** | QA at best | Users expected to preview transactions |
| **Unused view functions** | Low (QA) | Within protocol only |
| **Fault in OOS library** | Out of scope | Unless incorrect implementation |
| **Speculation on future code** | Invalid | Unless root cause in contest scope |
| **Weird/non-standard ERC20** | Invalid | Unless explicitly in scope |
| **Fee-on-transfer tokens** | Invalid | Unless explicitly in scope |
| **USDT non-standard behavior** | In scope | Exception to weird token rule |

### Admin/Privilege Issues

| Scenario | Severity |
|----------|----------|
| Reckless admin mistakes | Invalid |
| Direct misuse of privileges | QA Report |
| Mistakes only unblocked through admin error | QA Report |
| Privilege escalation | Up to Medium (likelihood + impact) |
| Privileged function + reasonable assumptions = vulnerability | Up to Medium |

**Note:** All roles expected to be trustworthy and responsible. Assume calls are previewed.

---

## Special Cases

### Constant Variables

For in-scope contracts, assume constant variables are mainnet settings (unless README says otherwise).
- Testnet values in constants → QA/Low at most

### Out-of-Scope (OOS) Library vs Incorrect Implementation

| Root Cause | Treatment |
|------------|-----------|
| Bug in OOS contract itself | Out of scope |
| Incorrect inheritance of OOS | In scope |
| Improper calls to OOS library | In scope |
| In-scope contract misuses OOS | Valid finding |

### Event-Related Impacts

Assessed based on broader-level impact:
- Events used for on-chain processes (bridging, proofs) → assessed on impacted functionality
- EIP non-compliance → assessed on impact
- No broader impact demonstrated → capped at Low
- Readability/accessibility of data → Low
- Front-end display issues → Low

---

## PoC Requirements

### Mandatory for Solidity/EVM Audits

**For all High/Medium submissions:**
- Coded, runnable PoC required
- Must use test suite provided in audit repo
- Must compile and demonstrate impact
- Exception: Wardens with signal ≥ 0.4 (PoC recommended but optional)

### Good PoC Practices

- Modify existing test files
- Provide diff/instructions to apply
- Any PoC that reverts should demonstrate precise error (not just revert)

### Burden of Proof

- Wardens have burden of proof
- Insufficient proof = judge needs to do extra research/coding
- Higher value submissions = higher burden
- Insufficient proof → not eligible above satisfactory score

---

## QA Report Guidelines

### Formatting

- Use standard labels: L-01, L-02, etc. for Low
- Use C-01, C-02, etc. for Centralization/governance
- Non-standard labels (R-, I-, S-) = informational (discouraged)

### QA Report Assessment

- Assessed on quality and thoroughness vs other reports
- Overstating severity = score reduction
- Review published audit reports for examples

---

## Centralization Risk Categories

**Submit as QA (Low Risk):**
- Direct misuse of admin privileges
- Issues requiring admin mistakes
- Privileged function issues with no valid exploit path

**Can be Medium (if likelihood + impact support):**
- Privilege escalation
- Reasonable assumptions + privileged functions = vulnerability

---

## Quick Severity Checklist

### High (3)
- [ ] Assets stolen/lost/compromised directly OR
- [ ] Valid indirect attack path (no hand-wavy hypotheticals)

### Medium (2)
- [ ] Assets not directly at risk BUT
- [ ] Protocol function/availability impacted OR
- [ ] Value leak with stated assumptions + external requirements

### QA/Low (1)
- [ ] No assets at risk
- [ ] State handling, function incorrect per spec, comment issues
- [ ] Centralization risks, admin privileges

### Invalid
- [ ] Approve front-run
- [ ] User mistake (no impact on others)
- [ ] Speculation on future code
- [ ] Weird tokens (not in scope)
- [ ] Admin mistake only

---

## Key Differences from Sherlock

| Aspect | Code4rena | Sherlock |
|--------|-----------|----------|
| PoC Required | Yes (unless signal ≥ 0.4) | Recommended |
| QA Report | Single report for all Low | No QA category |
| Approve front-run | Invalid | Can be valid |
| Weird tokens | Invalid (USDT exception) | Can be valid if in README |
| Likelihood | Considered | NOT considered |
