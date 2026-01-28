# Code4rena Audit Finding Template

Copy this template when submitting findings to Code4rena contests.

---

## [CONCISE TITLE - e.g., `Unauthorized ETH withdrawal due to missing access control`]

**Impact:** Attacker can drain all ETH from the Vault contract, resulting in complete loss of user funds. The vulnerability is exploitable without any special conditions or prior setup.

**Risk Rating:** HIGH

### Description
The `withdraw()` function in `src/Vault.sol` lacks access control, allowing any address to withdraw contract funds. This occurs because the function is missing the `onlyOwner` modifier that should restrict withdrawals to authorized addresses only.

### Affected Code
**File:** `src/Vault.sol`
**Lines:** 142-150

```solidity
function withdraw(uint256 amount) external {
    // @audit-issue: Missing access control
    balances[msg.sender] -= amount;
    payable(msg.sender).transfer(amount);
    emit Withdrawal(msg.sender, amount);
}
```

### Root Cause
The function was designed to allow only the contract owner to withdraw funds, but the `onlyOwner` modifier was omitted during implementation. This allows any external caller to invoke the function and receive ETH.

### Proof of Concept
The following Foundry test demonstrates the vulnerability:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/Vault.sol";

contract UnauthorizedWithdrawExploit is Test {
    Vault public vault;
    address public owner = address(0x1);
    address public attacker = address(0xattacker);

    function setUp() public {
        vm.prank(owner);
        vault = new Vault();
        vault.deposit{value: 100 ether}();
    }

    function testAttackerCanDrainVault() public {
        // Initial state: 100 ETH in vault
        assertEq(address(vault).balance, 100 ether);

        // Exploit: Attacker calls withdraw without authorization
        vm.prank(attacker);
        vault.withdraw(100 ether);

        // Verify: All funds transferred to attacker
        assertEq(address(vault).balance, 0);
        assertEq(attacker.balance, 100 ether);
    }

    function testPartialWithdrawal() public {
        // Attacker can withdraw any amount
        vm.prank(attacker);
        vault.withdraw(50 ether);

        assertEq(address(vault).balance, 50 ether);
        assertEq(attacker.balance, 50 ether);
    }
}
```

**To run the test:**
```bash
forge test --match-test testAttackerCanDrainVault -vvv
```

### Recommended Mitigation
Add the `onlyOwner` modifier to restrict access:

```solidity
function withdraw(uint256 amount) external onlyOwner {  // FIX: Add onlyOwner
    require(amount <= address(this).balance, "Insufficient balance");
    payable(owner).transfer(amount);
    emit Withdrawal(owner, amount);
}
```

**Additional recommendations:**
1. Consider using `WithdrawalPattern` with a two-step process for larger amounts
2. Add a timelock for withdrawals above a threshold
3. Implement an emergency pause mechanism

---

## Code4rena Severity Reference

| Severity | Description | Example |
|----------|-------------|---------|
| **HIGH** | Direct loss of user funds, protocol insolvency, broken key invariants | Unauthorized ETH withdrawal |
| **MEDIUM** | Indirect loss, griefing, broken non-critical invariants | Temporary DoS, incorrect fee calculation |
| **LOW** | Minor issues, code quality, gas optimizations | Unused variables, missing events |
| **GAS** | Pure gas optimizations (no security impact | Unnecessary SLOADs, loop optimizations) |
