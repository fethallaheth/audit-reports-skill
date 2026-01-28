# Sherlock Audit Finding Template

Copy this template when submitting findings to Sherlock contests. Submit as a GitHub Issue in the contest repository.

---

# [CONCISE TITLE - e.g., `Malicious Relayer Can Swap Adapter Against User's Will`]

## Summary
[2-3 sentence overview of the vulnerability]

**Example:** A malicious relayer will cause a loss of funds or unfavorable swap execution for the user as the relayer will execute the swap with a different adapter than the user requested (e.g., using Uniswap instead of Aave).

## Root Cause
[Explain WHY the vulnerability exists - the technical root cause]

**Key points to cover:**
- What validation/check is missing or broken?
- What assumption is violated?
- Show the vulnerable code snippet

**Example format:**
```
Missing request.adapter validation in the signed hash will cause a loss of
user control as a malicious relayer will replace the adapter with another
whitelisted adapter that may have unfavorable rates or fees.

In swap function the SWAP_TYPE_HASH does not include request.adapter as
part of the signed payload:

  bytes32 internal immutable constant SWAP_TYPE_HASH = keccak256(
      abi.encode(
          // ... other fields ...
          // adapter is MISSING from this hash!
      )
  );
```

## Internal Pre-conditions
[What conditions must be met within the protocol for this to be exploitable?]

**Example:**
- [ ] Validators need to sign a swap request
- [ ] A malicious relayer needs to be in the relayers mapping
- [ ] SpotSwap needs to have multiple adapters configured for the token pair

## External Pre-conditions
[What external conditions (outside the protocol) must be met?]

**Example:**
- None
- OR: Price oracle must be manipulatable via flash loan
- OR: Specific token must have low liquidity

## Attack Path
[Step-by-step description of how the attack works]

**Format:** Numbered list showing the attack flow

**Example:**
1. User requests a swap using Aave adapter (for better rates)
2. Validators sign the SwapRequest hash which excludes adapter
3. Malicious relayer intercepts the signed request
4. Malicious relayer replaces adapter address with UniswapAdapter
5. Malicious relayer calls bridge.swap() with the tampered request
6. The swap executes using Uniswap instead of Aave, at unfavorable rates

## Impact
[What are the consequences of this vulnerability?]

**Be specific about:**
- Fund loss amounts (if quantifiable)
- User control/permission issues
- Protocol insolvency risks
- Data/integrity impacts

**Example:**
```
The user suffers from execution against their will:
- Unfavorable swap rates - executed on a different DEX than requested
- Higher fees - if the alternate adapter/pool has higher fees
- Slippage - if the alternate path has less liquidity
```

## PoC
[Proof of Concept - executable code demonstrating the vulnerability]

```solidity
function testName() public {
    // Setup code

    // Exploit steps

    // Verification
}
```

**Tips for PoC:**
- Use Foundry (forge-std/Test.sol)
- Include setup, exploit, and verification phases
- Add console.log for clarity
- Ensure test passes with the bug, fails after fix

## Mitigation
[Specific code-level fix for the vulnerability]

**Example:**
```
Include request.adapter in the signed payload.

Before:
  bytes32 internal immutable constant SWAP_TYPE_HASH = keccak256(
      abi.encode(
          request.fromToken,
          request.toToken,
          // adapter MISSING
      )
  );

After:
  bytes32 internal immutable constant SWAP_TYPE_HASH = keccak256(
      abi.encode(
          request.fromToken,
          request.toToken,
          request.adapter,  // FIX: Add adapter to signed hash
      )
  );
```

---

## Sherlock Submission Checklist

- [ ] Title is descriptive and under 100 characters
- [ ] Summary clearly explains the vulnerability
- [ ] Root cause identifies the specific missing/broken check
- [ ] Internal pre-conditions are realistic (not overly complex)
- [ ] External pre-conditions are minimal (or "None")
- [ ] Attack path is step-by-step and reproducible
- [ ] Impact clearly states consequences to users/protocol
- [ ] PoC is executable and demonstrates the issue
- [ ] Mitigation provides specific code fix
- [ ] Issue is labeled as `HIGH` or `MEDIUM` severity

---

## Severity Quick Reference

| Severity | Criteria | Loss Threshold |
|----------|----------|----------------|
| **HIGH** | Direct fund loss without extensive external conditions | >1% AND >$10 |
| **MEDIUM** | Fund loss with constraints OR breaks core functionality | >0.01% AND >$10 |
| **INVALID** | No fund loss, gas only, user error only | N/A |

**DOS/Griefing:**
- Funds locked >1 week → MEDIUM
- Time-sensitive function → MEDIUM
- Both → HIGH

---

## Before Submitting - Invalid Issues Checklist

**DO NOT submit if your issue is:**

- [ ] Gas optimization (extra gas cost only)
- [ ] Incorrect event value (doesn't cause fund loss)
- [ ] Zero address check (standard validation)
- [ ] User input validation (prevents user mistakes only)
- [ ] Admin mistake (common sense - e.g., fee > 100%)
- [ ] UX issue (minor inconvenience only)
- [ ] Airdop/extra rewards loss (not part of protocol design)
- [ ] View function returns wrong value (unless used by other functions causing loss)
- [ ] Non-standard token behavior (unless mentioned in README)
- [ ] From previous audit marked "acknowledged" or "wont fix"
- [ ] Chain re-org/network liveness issue
- [ ] Sequencer downtime/misbehavior
- [ ] ERC721 unsafe mint (user's choice)
- [ ] Storage gap (simple contract)
- [ ] Design choice (no fund loss)

**For full criteria, see `sherlock-criteria.md`**
