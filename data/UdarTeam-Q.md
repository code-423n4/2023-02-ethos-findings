### Issues Template
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |

| Total Found Issues | 707 |
|:--:|:--:|


### Non-Critical Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | Explicitly mark state variables visibility | 5 |
| [N-02] | Explicitly mark uint type size | 557 |
| [N-03] | Create your own import names instead of using the regular ones | 97 |
| [N-04] | Insufficient coverage | - |


| Total Non-Critical Issues | 659 |
|:--:|:--:|

### Refactor Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | Use require instead of assert | 18 |
| [R-02] | Some number values can be refactored with _ | 2 |
| [R-03] | Validate input before storage reads | 1 |
| [R-04] | Remove unused variables | 2 |
| [R-05] | Function state mutability can be restricted to pure | 1 |

| Total Refactor Issues | 24 |
|:--:|:--:|

### Ordinary Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01] | Remove TODOs in code | 2 |
| [O-02] | Commented out code | 3 |
| [O-03] | Use a more recent pragma version | 15 |
| [O-04] | Locking pragma | 4 |


| Total Ordinary Issues | 24 |
|:--:|:--:|

### [N-01] Explicitly mark state variables visibility

Explicitly marking state variables visibility improves code readability and consistency.

```solidity
Ethos-Core/contracts/BorrowerOperations.sol

28: address stabilityPoolAddress;
30: address gasPoolAddress;
32: ICollSurplusPool collSurplusPool;
```

```solidity
Ethos-Core/contracts/TroveManager.sol

31: address gasPoolAddress;
33: ICollSurplusPool collSurplusPool;
```

### [N-02] Explicitly mark uint type size

Explicitly marking uint type size improves code readability and consistency.

```solidity
Ethos-Core/contracts/BorrowerOperations.sol    128 Instances
Ethos-Core/contracts/TroveManager.sol    233 Instances
Ethos-Core/contracts/ActivePool.sol    8 Instances
Ethos-Core/contracts/StabilityPool.sol    135 Instances
Ethos-Core/contracts/LQTY/LQTYStaking.sol    49 Instances
Ethos-Core/contracts/LQTY/LQTYStaking.sol    3 Instances
Ethos-Vault/contracts/ReaperVaultV2.sol    1 Instance
```

### [N-03] Create your own import names instead of using the regular ones

For better readability, you should name the imports instead of using the regular ones.

Example:
```solidity
5: import {IBorrowerOperations} from "./Interfaces/IBorrowerOperations.sol";
```

### [N-04] Insufficient code coverage

Best practice is to come as close to 100% as possible for smart contracts.

According to documentation, coverage is 93%.

### [R-01] Use require instead of assert
The Solidity assert() function is meant to assert invariants. Properly functioning code should never reach a failing assert statement.

Instances:

```solidity
Ethos-Core/contracts/BorrowerOperations.sol

128: assert(MIN_NET_DEBT > 0);
197: assert(vars.compositeDebt > 0);
301: assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
331: assert(_collWithdrawal <= vars.coll);

Ethos-Core/contracts/TroveManager.sol

417:  assert(_LUSDInStabPool != 0);
1224: assert(totalStakesSnapshot[_collateral] > 0);
1279: assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
1342: assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
1348: assert(index <= idxLast);
1414: assert(newBaseRate > 0);
1489: assert(decayedBaseRate <= DECIMAL_PRECISION);

Ethos-Core/contracts/StabilityPool.sol

526: assert(_debtToOffset <= _totalLUSDDeposits);
551: assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
591: assert(newP > 0);

Ethos-Core/contracts/LUSDToken.sol

312: assert(sender != address(0));
313: assert(recipient != address(0));
321: assert(account != address(0));
329: assert(account != address(0));
337: assert(owner != address(0));
338: assert(spender != address(0));
```

### [R-02] Some number values can be refactored with _

#### Ethos-Core/contracts/TroveManager.sol

```solidity
uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;
```

Refactor to:

```solidity 
uint constant public MINUTE_DECAY_FACTOR = 999_037_758_833_783_000;
```

#### Ethos-Vault/contracts/ReaperVaultV2.sol

```solidity
41:    uint256 public constant PERCENT_DIVISOR = 10000;
```

Refactor to:

```solidity
41:    uint256 public constant PERCENT_DIVISOR = 10_000;
```

### [R-03] Validate input before storage reads

If input parameters are validated by comparing with constants or immutables they should be done first to avoid storage reads and spend gas.

```solidity
Ethos-Core/contracts/CollateralConfig.sol

85: function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
    ) external onlyOwner checkCollateral(_collateral) {
        Config storage config = collateralConfig[_collateral];
        require(_MCR <= config.MCR, "Can only walk down the MCR");
        require(_CCR <= config.CCR, "Can only walk down the CCR");

        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
        config.MCR = _MCR;

        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
        config.CCR = _CCR;
        emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
    }
```

Above function can be refactored to:

```solidity    
85: function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
    ) external onlyOwner checkCollateral(_collateral) {
        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");

        Config storage config = collateralConfig[_collateral];
        require(_MCR <= config.MCR, "Can only walk down the MCR");
        require(_CCR <= config.CCR, "Can only walk down the CCR");
        config.MCR = _MCR;
        config.CCR = _CCR;
        emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
    }
```

### [R-04] Remove unused variables

```solidity 
Ethos-Core/contracts/StabilityPool.sol

339: uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit);
384: uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit);
```

### [R-05] Function state mutability can be restricted to pure

Functions can be declared pure in which case they promise not to read from or modify the state.

```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol

659:    function _cascadingAccessRoles() internal view override returns (bytes32[] memory)
```

### [O-01] Remove TODOs in code

```solidity 
Ethos-Core/contracts/StabilityPool.sol

380:     /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
         * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
         * If needed could create a separate event just to report this.
         */
335:     /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
         * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
         * If needed could create a separate event just to report this.
         */
```

### [O-02] Commented out code

```solidity
Ethos-Core/contracts/TroveManager.sol

18:    contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager
19:    // string constant public NAME = "TroveManager";
1413:    //assert(newBaseRate <= DECIMAL_PRECISION); // This is already enforced in the line above
```

### [O-03] Use a more recent pragma version

One of the most important best practices for writing secure Solidity code is to always use the latest version of the language. New versions of Solidity may include security updates and bug fixes that can help protect your code from potential vulnerabilities.

```solidity
Ethos-Core/contracts/ActivePool.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/BorrowerOperations.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/CollateralConfig.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/CollSurplusPool.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/DefaultPool.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/GasPool.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/HintHelpers.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/LUSDToken.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/Migrations.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/MultiTroveGetter.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/PriceFeed.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/RedemptionHelper.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/SortedTroves.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/StabilityPool.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/TroveManager.sol

3:    pragma solidity 0.6.11;
```

### [O-04] Locking pragma

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

#### Instances

- Ethos-Vault/contracts/ReaperVaultV2.sol
- Ethos-Vault/contracts/ReaperVaultERC4626.sol
- Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
- Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
