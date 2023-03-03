# Summary
## Low Risk
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|L-01       |Missing 0 checks in constructor|4|
|L-02       |Unspecific compiler version pragma|-|
|L-03       |Not every year equals 365 days|1|
|L-04       | Not enough checks on withdraw and deposit function | 2 |


## Non critical
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|N-01       | Use a newer solidity version | - |
|N-02       | Use a modifier instead of a function for prerequisites | 5 |
|N-03       | Event is missing indexed fields | 7 |
|N-04       | Make consistent use of comments in the contracts | - |
|N-05       | Use uint256 instead of uint | 1 |
|N-06       | Use scientific notation (1E18) rather than exponentation (10**18) | 2 |
|N-07       | Unused named return variables | 11 |
|N-08       | Formatting is not always correct | 3 |
|N-09       | Wrong comments | 1 |
|N-10       | Constant values such as a call to keccak256(), should be immutable rather than constant | 2 |
|N-11       | Only use assert when you check code that never should be false | 3 |
|N-12       | Functions should be grouped according to their visibility and ordered | 1 |

# Details
## Low Risk
## [L-01] Missing 0 checks in constructor
When you set addresses, it's a best practice to check for 0 addresses. This way a variable can't be accidently set to the 0 address. Roles are set in the constructor of ReaperVaultV2, without a check, ADMIN can be the 0 address.
- [ReaperVaultV2.sol#L111-L136](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136)
- [ReaperBaseStrategyv4.sol#L72-L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L72-L73)
- [ReaperBaseStrategyv4.sol#L82-L85](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L82-L85)
- [ReaperStrategyGranarySupplyOnly.sol#L68-L71](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L68-L71)
## [L-02] Unspecific compiler version pragma
Avoid floating or unspecific pragmas for non-library contracts.
Contracts should be deployed with the same compiler version that they have been tested with. When you lock the pragma, it ensures that the contract does not get accidently deployed with a compiler that has bugs.

This applies to the contracts in Ethos-Vault
## [L-03] Not every year equals 365 days
Due to leap years, not every year equals 365 days.
- [ReaperBaseStrategyv4.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L25)
## [L-04] Not enough checks on withdraw function
Although `_withdraw` is always called with msg.sender as owner and receiver, there are no require statements to check if this is true. 
To be safe, it's best to add a require statement in `_withdraw` to check if msg.sender equals owner and receiver. One change in the `withdraw` function where for example owner is set as a parameter and withdraw can be exploited.

[ReaperVaultV2.sol#L359-L412](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L359-L412)

The same applies to the `deposit` function

[ReaperVaultV2.sol#L319-L338](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L319-L338)
## Non critical
## [N-01] Use a newer solidity version
Although it will probably be a lot of refactoring work. It's better to use newer solidity versions. Old versions might contain bugs. This applies to the contract in Ethos-Core

## [N-02] Use a modifier instead of a function for prerequisites
It's more readable if you use a modifier instead of the functions for prerequisites. It's also possible to use multiple modifiers in one function and have modifiers with parameters so it's entirely possible.

```diff
ActivePool.sol
L159:
-   function getCollateral(address _collateral) external view override returns (uint) {
+   function getCollateral(address _collateral) external view override _requireValidCollateralAddress(_collateral) returns (uint) {
-       _requireValidCollateralAddress(_collateral);
        return collAmount[_collateral];
    }

L313:
-   function _requireValidCollateralAddress(address _collateral) internal view {
+   modifier _requireValidCollateralAddress(address _collateral){
        require(
            ICollateralConfig(collateralConfigAddress).isCollateralAllowed(_collateral),
            "Invalid collateral address"
        );
+       _;
    }
```

- [ActivePool.sol#L313-L342](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L313-L342)
- [BorrowerOperations.sol#L524-L656](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L524-L656)
- [LUSDToken.sol#L346-L395](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L346-L395)
- [StabilityPool.sol#L848-L875](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L848-L875)
- [TroveManager.sol#L1522-L1540](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1522-L1540)
## [N-03] Event is missing indexed fields
When an event is missing indexed fields it can be hard for off-chain scripts to efficiently filter those events.
- [ActivePool.sol#L58-L67](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L58-L67)
- [ReaperVaultV2.sol#L80-L100](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L80-L100)
- [ActivePool.sol#L58-L67](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L58-L67)
- [CollateralConfig.sol#L37-L38](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L37-L38)
- [StabilityPool.sol#L232-L234](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L232-L234)
- [StabilityPool.sol#L246-L250](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L246-L250)
- [TroveManager.sol#L203-L216](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L203-L216)
## [N-04] Make consistent use of comments in the contracts 
There are only a few functions that have comments. It is a good practice to use natspec comments. 
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

This is used sometimes in ethos-vault but not in ethos-core. It's best to be consistent with comments and use natspec in every contract.
## [N-05] Use uint256 instead of uint
For code readability it's better to make consistent use of uint256. Now both uint and uint256 are used and can lead to confusion.
- [BorrowerOperations.sol#L47-L78](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L47-L78)
## [N-06] Use scientific notation (1E18) rather than exponentation (10**18)
Scientific notation should be used for better code readability.
- [ReaperVaultV2.sol#L40](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40)
- [ReaperVaultV2.sol#L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125)
## [N-07] Unused named return variables
It's a good practice to not specify named return variables when they are never used. Instead of using named return variables, you can use natspec to keep readability.
- [ReaperVaultERC4626.sol#L29](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L29)
- [ReaperVaultERC4626.sol#L37](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L37)
- [ReaperVaultERC4626.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51)
- [ReaperVaultERC4626.sol#L66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66)
- [ReaperVaultERC4626.sol#L79](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79)
- [ReaperVaultERC4626.sol#L96](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L96)
- [ReaperVaultERC4626.sol#L122](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122)
- [ReaperVaultERC4626.sol#L165](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L165)
- [ReaperVaultERC4626.sol#L220](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L220)
- [ReaperVaultERC4626.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L240)
- [ReaperStrategyGranarySupplyOnly.sol#L104](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L104)
## [N-08] Formatting is not always correct
The formatting is not consistent across the different contracts.
A few examples:
- [ActivePool.sol#L108](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108): wrong spacing
```solidity
    for(uint256 i = 0; i < numCollaterals; i++) {
```
- [ActivePool.sol#L269](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L269): wrong spacing
- [BorrowerOperations.sol#L633](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L633)
- [TroveManager.sol#L1539](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539)

One click on format document should fix this
## [N-09] Wrong comments
The comment says it returns a uint256 with 18 decimals. However the function uses the decimal function which returns the correct amount of decimals of a token. Not every token has 18 decimals, BTC is used in the tests and only has 8 decimals. So the function `getPricePerFullShare()` returns a uint256 with 8 decimals during testing.
[ReaperVaultV2.sol#L293](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L293)
```solidity
    /**
     * @dev Function for various UIs to display the current value of one of our yield tokens.
     * Returns an uint256 with 18 decimals of how much underlying asset one vault share represents.
     */
    function getPricePerFullShare() public view returns (uint256) {
        return totalSupply() == 0 ? 10**decimals() : (_freeFunds() * 10**decimals()) / totalSupply();
    }
```
## [N-10] Constant values such as a call to keccak256(), should be immutable rather than constant
Constants are supposed to be used for literal values written into the code, and immutable variables should be used for expressions or values calculated in the constructor. While it doesn't save any gas, they should be used in their appropriate context.
- [ReaperVaultV2.sol#L73-L76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73-L76)
- [ReaperBaseStrategyv4.sol#L49-L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49-L52)
## [N-11] Only use assert when you check code that never should be false 
Require is used to validate inputs and conditions before execution. assert is used to check for code that should never be false. Failing assertion probably means that there is a bug. This will also save gas if the statement fails, assert uses up all the remaining gas. Require refund the remaining gas.

Some assert statements where the validation can be false without a bug and should be changed to require
- [LUSDToken.sol#L312](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312)
- [LUSDToken.sol#L338](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L338)
- [StabilityPool.sol#L526](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L526)
## [N-12] Functions should be grouped according to their visibility and ordered
According to the solidity style guide functions should be ordered like the following:
- constructor 
- receive function (if exists) 
- fallback function (if exists) 
- external 
- public 
- internal 
- private