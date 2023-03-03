# [L-01] Dependence on predictable environment variable 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1501
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1504
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1505
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1516
 
 The block.timestamp environment variable is used to determine a control flow decision. The  the value of variable block.imestamp is predictable and can be manipulated by a malicious miner.

# [N-01] Use a more recent version of Solidity
The use of the more recent version of solidity can help in preventing Integer Arithmetic Bugs (>0.8.0)

# [N-02] Function writing that does not comply with the Solidity Style Guide

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

Functions should be grouped according to their visibility and ordered:

-   constructor
-   receive function (if exists)
-   fallback function (if exists)
-   external
-   public
-   internal
-   private
-   within a grouping, place the view and pure functions last

This style guide is proposed by solidity community. It  intended to provide coding conventions for writing Solidity code
https://docs.soliditylang.org/en/v0.8.17/style-guide.html

# [N-03] require()/revert() statements should have descriptive reason strings
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L179
```require(totals.totalCollateralDrawn > 0);```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L304
``` require(_maxFeePercentage >= troveManager.REDEMPTION_FEE_FLOOR() && _maxFeePercentage <= DECIMAL_PRECISION);```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L309
```         require(block.timestamp >= systemDeploymentTime.add(BOOTSTRAP_PERIOD));```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L313

```require(_getTCR(_collateral, _price, _collDecimals) >= _MCR);```


https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L317

   ``` require(_amount > 0);```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L321
```       require(_lusdToken.balanceOf(_redeemer) >= _amount);```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L250
```     require(msg.sender == owner);```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L551

``` require(totals.totalDebtInSequence > 0);```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L716

``` require(_troveArray.length != 0);```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754

```        require(totals.totalDebtInSequence > 0);```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1450

``` require(redemptionFee < _collateralDrawn);```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1523

``` require(msg.sender == borrowerOperationsAddress); ```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1527

```         require(msg.sender == address(redemptionHelper)); ```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1531

  ```require(msg.sender == borrowerOperationsAddress || msg.sender == address(redemptionHelper));```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1535

```   require(Troves[_borrower][_collateral].status == Status.active);```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539

```        require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167

```  require(step[0] != address(0)); ```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L168

``` require(step[1] != address(0));```
# [N-04] For modern and more readable code; update import usages

Description
Another aspect of our Solidity code that you might not have seen is the struct Point. Prior to now, we imported it via global import, but we didn't use it. The Point struct contaminated the source code with an extra object that was not needed and that we were not using. Just import what you need is a modularity and modular programming rule that was broken here. We can use this rule more effectively when using specific imports using curly brackets.

Recommendation
import {contract1 , contract2} from "filename.sol"; 

# [N‑05] Lines are too long
Maximum suggested line length is 120 characters (https://docs.soliditylang.org/en/v0.8.17/style-guide.html)

Recommendation
- Wrapped lines should conform to the following guidelines.
- The first argument should not be attached to the opening parenthesis.
- One, and only one, indent should be used.
- Each argument should fall on its own line.
- The terminating element, );, should be placed on the final line by itself.

