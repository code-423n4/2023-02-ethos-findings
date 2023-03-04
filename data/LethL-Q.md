## Low Issues
|  | Issue | Instances |
|---|---|---|
| Low-1 | Lack of checks-effects-interactions | 1 |
| Low-2 | Loss of precision due to rounding | 1 |
| Low-3 | Outdated Compiler Version | 8 |

### [Low-1] Lack of checks-effects-interactions
External calls should be executed after state changes to avoid reetrancy bugs.
Consider moving the external calls after the state changes in the following instances:

Instances(1):
```
File: Ethos-Core/contracts/LUSDToken.sol

237:        _transfer(sender, recipient, amount);
238:        _approve(sender, msg.sender, _allowances[sender][msg.sender].sub(amount, "ERC20: transfer amount exceeds allowance"));
```
Link to Code: https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L237-L238

### [Low-2] Loss of precision due to rounding
Add scalars so roundings are negligible.

Instances(1):
```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

463:        uint256 performanceFee = (gain * strategies[strategy].feeBPS) / PERCENT_DIVISOR;
```
Link to Code: https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L463

### [Low-3] Outdated Compiler Version
Using an outdated compiler version (0.6.11) can be problematic if there are known bugs and issues with the current compiler version. It is recommended to use a latest stable version of the Solidity compiler (https://blog.soliditylang.org/2020/07/22/Solidity-0612-release-announcement/).

Instances(8):
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3

## Non Critical Issues
|  | Issue | Instances |
|---|---|---|
| NC-1 | Constants should be defined rather than using magic numbers | 1 |
| NC-2 | Using `block.timestamp` instead of `now` | 1 |

### [NC-1] Constants should be defined rather than using magic numbers
Rather than relying on magic numbers, constants should be defined.

Instances(1):
```
File: Ethos-Core/contracts/StabilityPool.sol

768:        if (compoundedDeposit < initialDeposit.div(1e9)) {return 0;}
```
Link to Code: https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L768

### [NC-2] Using `block.timestamp` instead of `now`
`block.timestamp` and `now` can be used interchangeably. However, it's generally considered best practice to use `block.timestamp` instead of `now`.

Instances(1):
```
File: Ethos-Core/contracts/LUSDToken.sol

282:        require(deadline >= now, 'LUSD: expired deadline');
```
Link to Code: https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#282