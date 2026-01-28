# Cantina Judging Criteria

Official Cantina judging guidelines for security researchers. Source: [Cantina Documentation](https://docs.cantina.xyz)

## Table of Contents

1. [Severity Matrix](#severity-matrix)
2. [Impact Criteria](#impact-criteria)
3. [Likelihood Criteria](#likelihood-criteria)
4. [Invalid Issue Categories](#invalid-issue-categories)
5. [PoC Requirements](#poc-requirements)
6. [Duplication Rules](#duplication-rules)

---

## Severity Matrix

| | Impact: High | Impact: Medium | Impact: Low |
|---|---|---|---|
| **Likelihood: High** | **High** | **High** | **Medium** |
| **Likelihood: Medium** | **High** | **Medium** | **Low** |
| **Likelihood: Low** | **Medium** | **Low** | **Informational** |

**Note:** This matrix is a guideline, not absolute. Context matters—severity assessments require more than checkboxes.

---

## Impact Criteria

### High Impact

**Loss of User Funds:**
- Vulnerability could lead to significant funds being stolen or lost

**Breaks Core Functionality:**
- Causes failure in fundamental protocol operations

### Medium Impact

**Temporary Disruption or DoS:**
- Leads to temporary downtime or denial of service
- Users experience disruptions but security not compromised

**Minor Fund Loss or Exposure:**
- Small amounts could be stolen (edge cases, price manipulation)
- Not widespread risk

**Breaks Non-Core Functionality**

### Low Impact

**No Assets at Risk:**
- State handling issues
- Incorrect function implementation
- Logic errors that don't threaten assets

---

## Likelihood Criteria

### High Likelihood

- Can be triggered by any user, without significant constraints
- Will generate outsized returns to the exploiter

### Medium Likelihood

- Significant constraints exist (capital requirement, planning, other users' actions)

### Low Likelihood

- Unusual scenarios (paused/exception state)
- Requires admin actions
- Many constraints that cannot be induced by user
- Causes significant loss to the user
- External upgradability issues (only if explicitly in scope)

---

## Severity Caps

### At Most Low Severity

| Issue Type | Note |
|------------|------|
| **Minimal Loss** | Rounding errors, minor fee discrepancies (even if repeatable infinitely) |
| **Weird ERC20 Tokens** | Non-compliant/weird ERC20 issues |
| **View Functions** | Errors not used within protocol |

### At Most Informational Severity

| Issue Type | Note |
|------------|------|
| **Admin Errors** | Wrong parameters, admin actions on integrated protocols |
| **Malicious Admin** | Unless explicitly in contest scope |
| **User Errors** | Without significant impact on other users |
| **Design Philosophy** | Trade-offs in permissionless protocols |
| **Missing Basic Validation** | Standard input checks |
| **Second-order Effects** | Issues arising from fix of another issue |

---

## Invalid Issues (Do Not Submit)

| Category | Reason |
|----------|--------|
| **Speculation on Future Code** | Not in current scope unless directly relates to current code |
| **Known Issues** | Already reported in LightChaser |
| **Public Fixes** | Duplicates after public fix are out of scope |

---

## Key Considerations

### Protocol Behavior
- Competition README is the main reference
- Not other sources (docs, Discord, etc.)

### Mandatory PoC Rule

**Default rule:** All High/Medium submissions require coded PoC before competition ends.

**Applies to:**
- Researchers with reputation score < 80

**Exemptions:**
- Reputation score ≥ 80
- Cantina Dedicated researchers (provide upon request)

**PoC Requirements:**
- Must compile and demonstrate impact
- Additional precautions help:
  - Mention which test file
  - Valid for audit branch
  - Additional requirements listed
  - PoC output provided
  - Output vs expected output explained

**Exception: PoC not required for missing function(s)**

---

## Duplication Rules

A "potential duplicate" IS a duplicate ONLY if ALL three are met:

1. **Clear Identification of Root Cause**
   - Must explicitly identify underlying root cause

2. **Medium or Higher Severity Impact**
   - Must demonstrate at least Medium impact

3. **Valid Attack Path**
   - Coded PoC demonstrating exploit, OR
   - Detailed walkthrough with all steps/conditions (reputation ≥ 80)

### Root Cause Resolution

If addressing the root cause would resolve the vulnerability → duplicate

### Failure to Meet Criteria

| Failure | Consequence |
|---------|-------------|
| Root cause not identified | Downgraded or invalidated |
| Insufficient impact analysis | Downgraded or invalidated |
| Incomplete/invalid attack path | Downgraded or invalidated |

**Invalid Attack Path includes:**
- Incorrect PoC (can't execute, doesn't demonstrate impact, unrealistic assumptions)
- Insufficient description (incomplete, unclear, missing steps/conditions)

### Withdrawn Findings

Withdrawn findings will NOT be considered for judgment, regardless of situation.

---

## Escalation Process

- Disagreement over severity or finding? Escalation process exists
- **Invalid escalations = penalties**
- Ensure your case is strong before escalating

---

## Submission Quality

Issues without necessary context/clarity will be invalidated.

**Avoid this by:**
- Properly filling out submission template
- Making findings clear, actionable, well-supported
- Remembering judges work under time pressure

---

## Quick Severity Decision Tree

```
Start: Does it cause fund loss or break core functionality?
├─ Yes + High Likelihood (any user, no constraints)
│  └─ HIGH
├─ Yes + Medium Likelihood (significant constraints)
│  └─ HIGH → Medium (based on context)
├─ Yes + Low Likelihood (admin action, unusual state)
│  └─ Medium
├─ No (temp disruption, minor loss)
│  └─ Medium → Low (based on impact)
└─ No assets at risk
   └─ Low → Informational
```

---

## Cantina vs Others: Key Differences

| Aspect | Cantina | Code4rena | Sherlock |
|--------|---------|-----------|----------|
| **Likelihood Considered** | Yes | Yes | No |
| **Severity Matrix** | Yes (3x3) | High/Med/QA | High/Med only |
| **PoC Required** | Rep < 80 | Signal < 0.4 | Recommended |
| **Admin Mistakes** | Informational | QA/Invalid | Invalid |
| **View Functions** | Low cap | Low cap | Low/Invalid |
| **Weird Tokens** | Low cap | Invalid (USDT except) | Invalid (not in README) |
| **Approve Front-run** | Not specified | Invalid | Can be valid |
| **Duplicate Criteria** | Root + Impact + Path | Not specified | Root + Impact + Path |
