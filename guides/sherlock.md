# Sherlock Judging Criteria

Official Sherlock judging guidelines for Watsons. Source: [Sherlock Judging Guidelines](https://docs.sherlock.xyz/audits/judging/guidelines)

## Table of Contents

1. [Severity Criteria](#severity-criteria)
2. [Valid Issue Categories](#valid-issue-categories)
3. [Invalid Issue Categories](#invalid-issue-categories)
4. [Special Cases](#special-cases)
5. [Duplication Guidelines](#duplication-guidelines)
6. [Best Practices](#best-practices)

---

## Severity Criteria

### HIGH Severity

**Definition:** Direct loss of funds without extensive limitations of external conditions. The loss must be **significant**.

**Significant Loss Thresholds:**
- Users lose **>1% AND >$10** of their principal, OR
- Users lose **>1% AND >$10** of their yield, OR
- Protocol loses **>1% AND >$10** of fees

**Key points:**
- Likelihood is NOT considered when identifying severity
- If a single attack can cause 0.01% loss but can be replayed indefinitely, it's considered 100% loss → High

### MEDIUM Severity

**Definition:** Causes loss of funds but requires certain external conditions or specific states, OR loss is highly constrained but **relevant**.

**Relevant Loss Thresholds:**
- Users lose **>0.01% AND >$10** of their principal, OR
- Users lose **>0.01% AND >$10** of their yield, OR
- Protocol loses **>0.01% AND >$10** of fees

**Also includes:**
- Breaking core contract functionality, rendering it useless or leading to relevant loss of funds

### DOS / Griefing / Locking

| Condition | Severity |
|-----------|----------|
| Funds locked >1 week | MEDIUM |
| Impacts time-sensitive functions | MEDIUM |
| BOTH conditions apply | HIGH |

**Note:** If a single DOS attack is <1 week but can be repeated, judge based on single occurrence. If single occurrence is >2 days and takes only 2-3 iterations to cause 7-day DOS, may be valid Medium.

---

## Valid Issue Categories

### Always Valid (with proper impact)

1. **Slippage issues** - Direct loss of funds with detailed explanation
2. **Reentrancy** - All forms (same function, cross-function, cross-contract, read-only, cross-chain)
3. **Access control** - Missing authorization on critical functions
4. **Front-running/sandwich** - With proper attack path and impact
5. **EIP Compliance** - Only if external integrations require it AND EIP is in final state

### Oracle Issues

**Chainlink min/max checks** - Medium ONLY if:
- Watson explicitly mentions specific price feeds for in-scope tokens/chains
- Proper attack path included
- At least medium severity impact

**Stale price checks** - May be valid if:
- Protocol uses pull-based oracle (e.g., Pyth)
- Not requesting/checking for staleness can cause using very old prices

### Out of Gas

Valid Medium/High if:
- Malicious user can fill arrays to cause OOG
- Practical call flow results in OOG

**Invalid** if:
- Array length controlled by trusted admin/owner
- Impractical parameter usage to reach OOG

---

## Invalid Issue Categories

### Always Invalid (Low/Informational)

| Category | Reason |
|----------|--------|
| **Gas optimizations** | User/protocol pays extra gas only |
| **Incorrect event values** | Wrong values in events don't cause fund loss |
| **Zero address checks** | Standard validation, not a security issue |
| **User input validation** | Preventing user mistakes not valid (unless causes major malfunction) |
| **Admin input/call validation** | Admin assumed to use correctly |
| **Contract/admin blacklisting** | Not considered protocol's fault |
| **Front-running initializers** | No irreversible damage, can redeploy |
| **User experience issues** | Minor inconvenience without fund loss |
| **User blacklist** | User harming only themselves |
| **Future opcode gas repricing** | Assumption about future EVM changes |
| **call vs transfer** | Design choice without good reason for >2300 gas |
| **Accidental direct token transfers** | User mistake if only hurts themselves |
| **Loss of airdrops** | Not part of original protocol design |
| **Storage gaps** (simple contracts) | Low priority unless complex inheritance |
| **Incorrect view function values** | Unless used in function causing fund loss |
| **Stale price recommendations** | General recommendations invalid |
| **Previous contest "wont fix" issues** | Already rejected |
| **Previous audit acknowledged issues** | Already known |
| **Chain re-org / network liveness** | Network-level issues |
| **ERC721 unsafe mint** | User chooses to use unsafe mint |
| **Future integration issues** | Not in current scope |
| **Non-standard tokens** | Unless explicitly mentioned in README |
| **Sequencer downtime** | Sequencers assumed reliable |

### Admin Trust Assumptions

- Admin functions generally assumed to be used correctly
- If admin would **unknowingly** cause issues → Can be valid
- Internal protocol roles are trusted by default
- Roles can be untrusted ONLY if:
  - README specifically claims they're untrusted, OR
  - User can get role without admin/owner permission

**Examples:**

**Invalid:** "Admin can break deposit by setting fee to 100%+" → Common sense

**Valid:** "Admin sets fee to 20%, causing liquidations to fail when utilization <10%" → Admin unaware of consequences

### Design Decisions

- Design decisions are NOT valid issues
- Even if suboptimal, must not imply fund loss
- Otherwise considered informational

---

## Special Cases

### Hierarchy of Truth

1. **Contest README** (primary source of truth)
2. **CODE COMMENTS** (can be used as context)
3. **Default guidelines** (if README/comments are silent)

**Rules:**
- README trumps CODE COMMENTS if contradiction
- Judge can decide CODE COMMENTS are outdated
- Public statements up to 24h before contest ends can override README
- Historical decisions are NOT sources of truth

**README can ONLY define invariants via this specific question:**
> "What properties/invariants do you want to hold even if breaking them has a low/unknown impact?"

**Examples:**

| README Statement | Issue | Valid? |
|-----------------|-------|--------|
| "Admin can only call XYZ once" | Code allows calling XYZ twice | ✓ Medium |
| "Variable X must match USDC amount" | User donates USDC to break invariant | ✗ Invalid (no harm) |

### Contract Scope

- If contract is in scope → all parent contracts included
- Vulnerability in library used by in-scope contract → Valid
- Vulnerability in out-of-scope contract → Invalid

### Front-running Severity

If deployment chain has **private mempool** (e.g., Optimism):
- High → Medium
- Medium → Invalid

**Note:** Must explain unintentional front-running. Saying "attacker monitors mempool" is invalid.

---

## Duplication Guidelines

A "potential duplicate" IS a duplicate ONLY if ALL three are met:

1. **Identifies the same root cause**
2. **Identifies at least Medium impact**
3. **Identifies a valid attack/vulnerability path**

Otherwise, judged separately (solo, duplicate of different issue, invalid, etc.)

### Root Cause Groupings

**Same root cause if:**
- Same logic mistake (e.g., uint256→uint128 unsafe cast)
- Same conceptual mistake (e.g., different untrusted external admins stealing funds)

**Category groupings:**
- Slippage protection issues
- Reentrancy (all types: same function, cross-function, cross-contract, read-only, cross-chain)
- Access control
- Front-run/sandwich

**If underlying code, impact, or fixes are different** → Treated separately

---

## Best Practices

### DO:

- Read contest README and documents thoroughly
- Submit issues valid according to guidelines
- Be specific about impact
  - ✅ "Loss of funds for users as there is no access control for the 'withdraw' function"
  - ❌ "Loss of funds for the user"
- Keep code snippets limited to impact scope
- Be descriptive in Vulnerability Details
- Submit different issues separately (even if on same line)
- Combine obvious repetitions (e.g., safeTransfer in multiple places)

### DON'T:

- Submit multiple different issues in one submission
- Add unnecessarily long code snippets
- Copy-paste from other contests/reports/audits
- Submit issues unlikely to be valid

### PoC Recommendations

PoC is **recommended** for:
- Non-obvious issues with complex vulnerabilities
- Issues with non-trivial input constraints
- Precision loss issues
- Reentrancy attacks
- Gas consumption/reverting attacks

**Note:** If original report has no PoC and issue cannot be clearly understood without one → INVALID

---

## Quick Severity Decision Tree

```
Is there fund loss?
├─ No
│  └─ INVALID (unless breaks README-defined invariant)
└─ Yes
   ├─ Direct loss, minimal external conditions
   │  └─ HIGH (if significant: >1% AND >$10)
   ├─ Loss with specific conditions/constrained
   │  └─ MEDIUM (if relevant: >0.01% AND >$10)
   └─ DOS/Griefing
      ├─ Funds locked >1 week → MEDIUM
      ├─ Time-sensitive function → MEDIUM
      └─ Both → HIGH
```

---

## Summary Checklist

Before submitting, verify your issue is NOT:

- [ ] Gas optimization
- [ ] Incorrect event value
- [ ] Zero address check
- [ ] User input validation only
- [ ] Admin mistake (common sense)
- [ ] UX issue only
- [ ] Airdrop loss
- [ ] View function error (unused)
- [ ] Non-standard token (not in README)
- [ ] From previous audit (acknowledged)

Verify your issue HAS:

- [ ] Clear fund loss OR breaks README-defined invariant
- [ ] Specific impact (>1% for High, >0.01% for Medium)
- [ ] Valid attack path
- [ ] Root cause clearly identified
