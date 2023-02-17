### Issues Template
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |

| Total Found Issues | 678 |
|:--:|:--:|



### Non-Critical Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | Explicitly mark state variables visibility | 5 |
| [N-02] | Explicitly mark uint type size | 557 |
| [N-03] | Create your own import names instead of using the regular ones | 97 |


| Total Non-Critical Issues | 659 |
|:--:|:--:|

### Refactor Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | Use require instead of assert | 18 |
| [R-02] | Some number values can be refactored with _ | 1 |

| Total Refactor Issues | 19 |
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

```solidity
Ethos-Core/contracts/TroveManager.sol

uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;
```

Refactor to:

```solidity 
uint constant public MINUTE_DECAY_FACTOR = 999_037_758_833_783_000;
```
