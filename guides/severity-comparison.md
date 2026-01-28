# Severity Criteria Reference

Detailed severity guidelines for Sherlock, Code4rena, and Cantina audit contests.

## Table of Contents

1. [Sherlock Severity Criteria](#sherlock-severity-criteria)
2. [Code4rena Severity Criteria](#code4rena-severity-criteria)
3. [Cantina Severity Criteria](#cantina-severity-criteria)

---

## Sherlock Severity Criteria

Source: [Sherlock Judging Guidelines](https://docs.sherlock.xyz/audits/judging/guidelines)

### HIGH Severity

Issues that result in a direct loss of user or protocol funds.

**Examples:**
- Direct theft of user funds (e.g., missing access control allows anyone to withdraw)
- Direct theft of protocol funds (e.g., attacker can drain treasury)
- Breaking of core protocol invariants (e.g., permanent lock of funds, insolvency)
- Griefing that causes significant loss (e.g., repeated DoS attacks costing users gas)

**Key factors for HIGH:**
- Direct fund loss (not theoretical)
- Exploitable in current conditions (no unlikely assumptions)
- Significant impact (not negligible amounts)

### MEDIUM Severity

Issues that are:
1. Harder to exploit but still result in fund loss, OR
2. Result in significant but indirect value loss

**Examples:**
- Fund loss requiring complex attack paths (3+ steps, external conditions)
- Permanent locking of funds for some users (not all)
- Breaking of important (but not core) invariants
- Griefing with limited impact

**Key factors for MEDIUM:**
- Fund loss requires unlikely conditions or complex setup
- Indirect value loss (e.g., opportunity cost, loss of yield)
- Affects subset of users or specific scenarios

### LOW Severity

Issues with minimal impact:

**Examples:**
- Breaking invariants that don't result in value loss
- Temporary locking of funds with recovery path
- Code quality issues
- Minor inaccuracies in calculations (negligible impact)

---

## Code4rena Severity Criteria

Source: [Code4rena Documentation](https://docs.code4rena.com)

### HIGH Risk Rating

Issues where:
- Code is directly exploitable
- Results in loss of user/protocol funds
- Breaking of protocol guarantees

**Examples:**
- Smart contract vulnerability allowing ETH/token theft
- Price manipulation allowing arbitrage/MEV profit
- Incorrect access control exposing privileged functions

### MEDIUM Risk Rating

Issues where:
- Code is exploitable but with limitations
- Impact is significant but not critical
- Requires specific conditions to exploit

**Examples:**
- Fund loss requiring external protocol failure
- DoS attacks with mitigation paths
- Incorrect logic causing loss of yield (not principal)

### LOW Risk Rating

Minor issues with limited impact:

**Examples:**
- Edge cases with minimal user impact
- Missing events or incorrect event emissions
- Minor documentation issues
- Non-critical code quality improvements

### GAS Optimizations

Pure gas optimization findings without security impact:

**Examples:**
- Unnecessary SLOADs
- Inefficient loops
- Unused storage variables
- Redundant checks

**Note:** GAS issues are scored separately from severity.

---

## Cantina Severity Criteria

Source: [Cantina Finding Severity](https://docs.cantina.xyz/cantina-docs/cantina-competitions/judging-process/finding-severity-criteria)

### CRITICAL / HIGH

Direct loss of user funds or protocol insolvency.

**Examples:**
- Direct theft of user deposits
- Attacker can mint unlimited tokens
- Protocol invariant break causing fund loss
- Price oracle manipulation allowing profit extraction

### MEDIUM

Significant issues that don't cause direct fund loss:

**Examples:**
- Temporary denial of service
- Loss of yield (not principal)
- Incorrect fee calculations
- Griefing attacks with limited cost
- Breaking important but not critical invariants

### LOW

Minor issues:

**Examples:**
- Code quality improvements
- Missing events
- Minor documentation errors
- Non-standard patterns (no security impact)

---

## Cross-Platform Severity Comparison

| Scenario | Sherlock | Code4rena | Cantina |
|----------|----------|-----------|---------|
| Direct fund theft | HIGH | HIGH | HIGH |
| Complex 3-step attack | HIGH/MED | MED | MED |
| Permanent lock (all users) | HIGH | HIGH | HIGH |
| Permanent lock (some users) | MED | MED | MED |
| Temporary DoS | MED | MED | MED |
| Loss of yield (not principal) | MED | MED | MED |
| Gas optimization | Not accepted | GAS | LOW |
| Code quality | LOW | LOW | LOW |

## Severity Classification Decision Tree

```
Is fund loss possible?
├─ Yes, direct and immediate
│  └─ HIGH
├─ Yes, but requires complex/unlikely conditions
│  └─ MEDIUM
├─ Yes, but only yield/opportunity cost (not principal)
│  └─ MEDIUM
└─ No
   └─ Is it a gas optimization?
      ├─ Yes (Code4rena only)
      │  └─ GAS
      └─ No
         └─ LOW
```

## Tips for Choosing Severity

1. **Default to lower severity** when unsure - judges can upgrade
2. **Focus on impact** over exploit complexity for HIGH severity
3. **Be realistic** about attack conditions - don't over-assume
4. **Document assumptions** - explain why certain conditions are met
5. **Compare to past issues** - look at previous contest reports for similar issues

## Common Severity Mistakes

**Over-severity:**
- Calling theoretical issues "HIGH" without realistic attack path
- Marking gas issues as "HIGH" or "MEDIUM" (should be GAS or LOW)
- Calling design choices "vulnerabilities" (unless they break documented specs)

**Under-severity:**
- Calling direct fund theft "MEDIUM" due to "unlikely" conditions
- Marking permanent fund locks as "LOW"
- Calling oracle manipulation "GAS" or "LOW"
