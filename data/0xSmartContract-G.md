### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] | Remove the `initializer` modifier | 2 |
| [G-02] |Use hardcode address instead ``address(this)``| 36 |
| [G-03] |Structs can be packed into fewer storage slots |2 |
| [G-04] |Pack state variables | 4 |
| [G-05] | Remove or replace unused state variables |1|
| [G-06] | ``Deposit`` struct can be removed | 1 |
| [G-07] |Using ``delete``  instead of setting mapping/state variable ``0`` saves gas | 9 |
| [G-08] | Using ``delete`` instead of setting ``address(0)`` saves gas | 1 |
| [G-09] | Using ``delete`` to set ``bool`` values to their default values saves gas  | 3 |
| [G-10] |Using `storage` instead of `memory` for `structs/arrays` saves gas | 7 |
| [G-11] | Multiple ``address``/ID mappings can be combined into a single ``mapping`` of an ``address``/ID to a ``struct``, where appropriate |7 |
| [G-12] |Multiple accesses of a mapping/array should use a local variable cache | 30 |
| [G-13] |The result of function calls should be cached rather than re-calling the function  | 4 |
| [G-14] |Avoid using ``state variable`` in emit |1 |
| [G-15] |Superfluous event fields | 1 |
| [G-16] |Avoid contract existence checks by using low level calls |97 |
| [G-17] |``bytes`` constants are more eficient than ``string`` constans | 8 |
| [G-18] |Change ``public`` function visibility to ``external`` |3 |
| [G-19] |Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead |6|
| [G-20] |``internal`` functions only called once can be inlined to save gas | 45 |
| [G-21] |Do not calculate constants variables | 11 |
| [G-22] |Use ``assembly`` to write _address storage values_  | 18 |
| [G-23] |Setting the _constructor_ to `payable` | 6 |
| [G-24] |Add ``unchecked {}`` for subtractions where the operands cannot underflow because of a previous ``require`` or ``if`` statement | 10 |
| [G-25] |++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops | 13 |
| [G-26] |`x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables | 9 |
| [G-27] |Use ``require`` instead of ``assert``| 20 |
| [G-28] |Use nested if and, avoid multiple check combinations | 3 |
| [G-29] |Use ``double require`` instead of using ``&&``| 4 |
| [G-30] |Sort Solidity operations using short-circuit mode | 14 |
| [G-31] |Optimize names to save gas | All contracts |
| [G-32] |Use a more recent version of solidity | 12 |
| [G-33] |``>=`` costs less gas than ``>`` | 1 |
| [G-34] |Upgrade Solidity's optimizer|  |

Total 34 issues

### [G-01] Remove the `initializer` modifier

if we can just ensure that the `initialize()` function could only be called from within the constructor, we shouldn't need to worry about it getting called again. 

2 results - 2 files:
```solidity
Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol:

  67:     ) public initializer {

```


```solidity
Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:

  61:     constructor() initializer {}

```

In the EVM, the constructor's job is actually to return the bytecode that will live at the contract's address. So, while inside a constructor, your address `(address(this))` will be the deployment address, but there will be no bytecode at that address! So if we check `address(this).code.length` before the constructor has finished, even from within a delegatecall, we will get 0. So now let's update our `initialize()` function to only run if we are inside a constructor:


```diff

Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol:

- 67:     ) public initializer {
+        require(address(this).code.length == 0, 'not in constructor’);
```

Now the Proxy contract's constructor can still delegatecall initialize(), but if anyone attempts to call it again (after deployment) through the Proxy instance, or tries to call it directly on the above instance, it will revert because address(this).code.length will be nonzero. 

Also, because we no longer need to write to any state to track whether initialize() has been called, we can avoid the 20k storage gas cost. In fact, the cost for checking our own code size is only 2 gas, which means we have a 10,000x gas savings over the standard version. Pretty neat!


### [G-02] Use hardcode address instead ``address(this)``

Instead of ``address(this)``, it is more gas-efficient to pre-calculate and use the address with a hardcode. Foundry's ``script.sol`` and solmate````LibRlp.sol`` contracts can do this.

Reference:
https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824


36 results - 9 files:
```solidity
Ethos-Core\contracts\ActivePool.sol:

  210:    IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _amount);

  247:    vars.ownedShares = vars.yieldGenerator.balanceOf(address(this));

  280:    IERC4626(yieldGenerator[_collateral]).deposit(uint256(vars.netAssetMovement), address(this));

  282:    IERC4626(yieldGenerator[_collateral]).withdraw(uint256(-vars.netAssetMovement), address(this), address(this));
 
  288:    vars.profit = IERC20(_collateral).balanceOf(address(this)).add(vars.yieldingAmount).sub(collAmount[_collateral]);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L210


```solidity
Ethos-Core\contracts\BorrowerOperations.sol:

  231:    IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collAmount);

  497:    IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collChange);

  530:    require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L231


```solidity
Ethos-Core\contracts\LUSDToken.sol:

  305:    return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));

  349:    _recipient != address(this),

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L305


```solidity
Ethos-Core\contracts\StabilityPool.sol:

  605:    lusdToken.burn(address(this), _debtToOffset);
  606  
  607:    activePoolCached.sendCollateral(_collateral, address(this), _collToAdd);

  777:    lusdToken.sendToPool(_address, address(this), _amount);

  797:    lusdToken.returnFromPool(address(this), _depositor, LUSDWithdrawal);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L605


```solidity
Ethos-Core\contracts\LQTY\CommunityIssuance.sol:

  103:    OathToken.transferFrom(msg.sender, address(this), amount);

 ```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103


```solidity
Ethos-Core\contracts\LQTY\LQTYStaking.sol:

  128:    lqtyToken.safeTransferFrom(msg.sender, address(this), _LQTYamount);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L128


```solidity
Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol:

  120:    uint256 amount = startToken.balanceOf(address(this));

  127:    uint256 allocated = IVault(vault).strategies(address(this)).allocated;

  180:    ILendingPool(lendingPoolAddress).deposit(want, toReinvest, address(this), LENDER_REFERRAL_CODE_NONE);

  200:    ILendingPool(ADDRESSES_PROVIDER.getLendingPool()).withdraw(address(want), _withdrawAmount, address(this));

  227:    return IERC20Upgradeable(want).balanceOf(address(this));

  233:    address(this)

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L120


```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  153:    require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");

  246:    available = Math.min(available, token.balanceOf(address(this)));

  279:    return token.balanceOf(address(this)) + totalAllocated;

  327:    uint256 balBefore = token.balanceOf(address(this));

  328:    token.safeTransferFrom(msg.sender, address(this), _amount);

  329:    uint256 balAfter = token.balanceOf(address(this));

  368:    if (value > token.balanceOf(address(this))) {

  373:    vaultBalance = token.balanceOf(address(this));

  386:    uint256 actualWithdrawn = token.balanceOf(address(this)) - vaultBalance;
 
  399:    vaultBalance = token.balanceOf(address(this));

  528:    token.safeTransferFrom(vars.stratAddr, address(this), vars.freeWantInStrat - vars.credit);
 
  641:    uint256 amount = IERC20Metadata(_token).balanceOf(address(this));

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L153


```solidity
Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:

  159:    IVault(vault).revokeStrategy(address(this));

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L159

### [G-03] Structs can be packed into fewer storage slots

1. The "StrategyParams" construct can be packaged as suggested below. Here, ``uint64`` is suggested for ``uint256 activation`` and```uint256 lastReport`` considering that it will be valid for the next 532 years.

**1 slot win 1 * 2k  =  2k gas saved**

```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  25:     struct StrategyParams {
  26:         uint256 activation; // Activation block.timestamp
  27:         uint256 feeBPS; // Performance fee taken from profit, in BPS
  28:         uint256 allocBPS; // Allocation in BPS of vault's total assets
  29:         uint256 allocated; // Amount of capital allocated to this strategy
  30:         uint256 gains; // Total returns that Strategy has realized for Vault
  31:         uint256 losses; // Total losses that Strategy has realized for Vault
  32:         uint256 lastReport; // block.timestamp of the last time a report occured
  33:     }

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L25-L33

```diff
Ethos-Vault\contracts\ReaperVaultV2.sol:
  25:     struct StrategyParams {
- 26:         uint256 activation; // Activation block.timestamp
+ 26:         uint64 activation; // Activation block.timestamp
+ 32:         uint64 lastReport; // block.timestamp of the last time a report occured
  27:         uint256 feeBPS; // Performance fee taken from profit, in BPS
  28:         uint256 allocBPS; // Allocation in BPS of vault's total assets
  29:         uint256 allocated; // Amount of capital allocated to this strategy
  30:         uint256 gains; // Total returns that Strategy has realized for Vault
  31:         uint256 losses; // Total losses that Strategy has realized for Vault
- 32:         uint256 lastReport; // block.timestamp of the last time a report occured
  33:     }

```

2. The "Config" structure can be packaged as suggested below.

**1 slot win 1 * 2k  =  2k gas saved**

```solidity
Ethos-Core\contracts\CollateralConfig.sol:

  27:     struct Config {
  28:         bool allowed;
  29:         uint256 decimals;
  30:         uint256 MCR;
  31:         uint256 CCR;
  32:     }

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L27-L32

```diff
Ethos-Core\contracts\CollateralConfig.sol:

  27:     struct Config {
  28:         bool allowed;
- 29:         uint256 decimals;
+ 29:         uint128 decimals;  
  30:         uint256 MCR;
  31:         uint256 CCR;
  32:     }

```

### [G-04] Pack state variables

1. State variable packaging in **ReaperBaseStrategyv4.sol** contract  (**1 * 2k = 2k gas saved**)

```solidity
Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:

  31:     uint256 public lastHarvestTimestamp;

  33:     uint256 public upgradeProposalTime;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L31-L33

```diff
Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:

- 31:     uint256 public lastHarvestTimestamp;
+ 31:     uint64 public lastHarvestTimestamp;

- 33:     uint256 public upgradeProposalTime;
+ 33:     uint64 public upgradeProposalTime;

```


2. State variable packaging in **ActivePool.sol** contract (3 * 2.1k = **6.3k Gas saved**)

```solidity
Ethos-Core\contracts\ActivePool.sol:
   
  49:     uint256 public yieldingPercentageDrift = 100; // rebalance iff % is off by more than 100 BPS
  50: 
  51:     // Yield distribution params, must add up to 10k
  52:     uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
  53:     uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
  54:     uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L49-L54

The state variable ``yieldingPercentageDrift`` can be updated to a maximum value of 500 as seen in ``function setYieldingPercentageDrift``.

```solidity
Ethos-Core\contracts\ActivePool.sol:   
132 function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {   
133: require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");   
134: yieldingPercentageDrift = _driftBps;

```

On the other hand, `yieldSplitTreasury``, ``yieldSplitSP`` and  The ``yieldSplitStaking`` state variables can total up to 10k.

```solidity
51: // Yield distribution params, must add up to 10k

```

Given this information, these state variables can be tightly packed into 1 slot instead of 4 if updated as shown below.

```diff
Ethos-Core\contracts\ActivePool.sol:
     
- 49:     uint256 public yieldingPercentageDrift = 100; // rebalance iff % is off by more than 100 BPS
+ 49:     uint64 public yieldingPercentageDrift = 100; // rebalance iff % is off by more than 100 BPS
  
  51:     // Yield distribution params, must add up to 10k
- 52:     uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
+ 52:     uint64 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
- 53:     uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
+ 53:     uint64 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
- 54:     uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS
+ 54:     uint64 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS

```

3. State variable packaging in **ReaperVaultV2.sol** contract (3 * 2.1k = **6.3k Gas saved**)

```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:
 
  35:     mapping(address => StrategyParams) public strategies;

  38:     address[] public withdrawalQueue;

  42:     uint256 public tvlCap;
  43: 
  44:     uint256 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)
  45:     uint256 public totalAllocated; // Amount of tokens that have been allocated to all strategies
  46:     uint256 public lastReport; // block.timestamp of last report from any strategy

  49:     bool public emergencyShutdown;

  54:     // Max slippage(loss) allowed when withdrawing, in BPS (0.01%)
  55:     uint256 public withdrawMaxLoss = 1;
  56:     uint256 public lockedProfitDegradation; // rate per block of degradation. DEGRADATION_COEFFICIENT is 100% per block
  57:     uint256 public lockedProfit; // how much profit is locked and cant be withdrawn

  78:     address public treasury; // address to whom performance fee is remitted in the form of vault shares

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L35-L78

```diff
Ethos-Vault\contracts\ReaperVaultV2.sol:

  35:     mapping(address => StrategyParams) public strategies;

  38:     address[] public withdrawalQueue;

  42:     uint256 public tvlCap;

+ 49:     bool public emergencyShutdown;
- 44:     uint256 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)
+ 44:     uint192 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)
  45:     uint256 public totalAllocated; // Amount of tokens that have been allocated to all strategies
- 46:     uint256 public lastReport; // block.timestamp of last report from any strategy
+ 46:     uint64 public lastReport; // block.timestamp of last report from any strategy

- 49:     bool public emergencyShutdown;

- 55:     uint256 public withdrawMaxLoss = 1;
+ 55:     uint64 public withdrawMaxLoss = 1;
- 56:     uint256 public lockedProfitDegradation; // rate per block of degradation. DEGRADATION_COEFFICIENT is 100% per block
+ 56:     uint128 public lockedProfitDegradation; // rate per block of degradation. DEGRADATION_COEFFICIENT is 100% per block
  57:     uint256 public lockedProfit; // how much profit is locked and cant be withdrawn

  78:     address public treasury; // address to whom performance fee is remitted in the form of vault shares

```

4. State variable packaging in **ReaperBaseStrategyv4.sol** contract (2 * 2.1k = **4.2k Gas saved**)

```solidity
Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:

  28:     address public want;

  30:     bool public emergencyExit;
  31:     uint256 public lastHarvestTimestamp;
 
  33:     uint256 public upgradeProposalTime;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L28-L33

```diff
- 28:     address public want;

  30:     bool public emergencyExit;
- 31:     uint256 public lastHarvestTimestamp;
+ 31:     uint64 public lastHarvestTimestamp; 

- 33:     uint256 public upgradeProposalTime;
+ 33:     uint64 public upgradeProposalTime;
+ 28:     address public want;

```

### [G-05] Remove or replace unused state variables

Saves a storage slot. If the variable is assigned a non-zero value, saves Gsset (20000 gas). If it's assigned a zero value, saves Gsreset (2900 gas). If the variable remains unassigned, there is no gas savings unless the variable is public, in which case the compiler-generated non-payable getter deployment cost is saved. If the state variable is overriding an interface's public function, mark the variable as constant or immutable so that it does not use a storage slot

1 result - 1 file:
```solidity
Ethos-Core\contracts\StabilityPool.sol:

  160:     address public lqtyTokenAddress;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L160


### [G-06] ``Deposit`` struct can be removed

```solidity
Ethos-Core\contracts\StabilityPool.sol:
   174: struct Deposit {
   175: uint initialValue;
   176: }

   186: mapping (address => Deposit) public deposits; // depositor address -> deposit struct

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L174-L176


Using a direct type when accessing mapping values is more gas-optimized than using a single-element structure. Because the first requires a storage slot to be read or written. The second requires reading or writing two storage slots.
(one for the struct and one for the value inside the struct)

Here, removing ``struct Deposit`` and accessing directly from mapping is gas-optimized.

```diff
Ethos-Core\contracts\StabilityPool.sol:
- 174: struct Deposit {
- 175: uint initialValue;
- 176: }

- 186: mapping (address => Deposit) public deposits; // depositor address -> deposit struct
// address depositor -> uint256 initialValue
+ 186: mapping (address => uint256) public deposits;

```

### [G-07] Using ``delete``  instead of setting mapping/state variable ``0`` saves gas

9 results - 3 files:
```solidity
Ethos-Core\contracts\StabilityPool.sol:

  529:     lastLUSDLossError_Offset = 0;

  578:     currentScale = 0;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L529

```solidity
Ethos-Core\contracts\TroveManager.sol:

  1192:    Troves[_borrower][_collateral].stake = 0;

  1285:    Troves[_borrower][_collateral].coll = 0;

  1286:    Troves[_borrower][_collateral].debt = 0;

  1288:    rewardSnapshots[_borrower][_collateral].collAmount = 0;

  1289:    rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1192

```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  215:     strategies[_strategy].allocBPS = 0;

  537:     lockedProfit = 0;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L215


**Recommendation code:**
```diff
Ethos-Vault\contracts\ReaperVaultV2.sol:

- 215:     strategies[_strategy].allocBPS = 0;
+ 215:     delete strategies[_strategy].allocBPS;

```

### [G-08] Using ``delete`` instead of setting ``address(0)`` saves gas

1 result - 1 file:
```solidity
Ethos-Core\contracts\TroveManager.sol:

  294:         owner = address(0);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L294

**Proof Of Concepts:**
```solidity
contract Example {
    address public myAddress = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;

    //  use for empty value Slot:     23450 gas
    //  use for non empty value Slot: 21450 gas
    function reset() public {
        delete myAddress;
    }


    // use for empty value Slot:     23497 gas
    // use for non empty value Slot: 23497 gas
    function setToZero() public {
        myAddress = address(0);
    }
}
```


### [G-09] Using ``delete`` to set ``bool`` values to their default values saves gas

3 result - 2 files:
```solidity
Ethos-Core\contracts\LUSDToken.sol:

  143:         mintingPaused = false;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L143

```solidity
Ethos-Core\contracts\TroveManager.sol:

  602:         vars.backToNormalMode = false;

  804:         vars.backToNormalMode = false;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L602
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L804


### [G-10] Using `storage` instead of `memory` for `structs/arrays` saves gas

When fetching data from a storage location, assigning the data to a ``memory`` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the ``memory`` keyword, declaring the variable with the ``storage`` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a ``memory`` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires ``memory``, or if the array/struct is being read from another ``memory`` array/struct

7 results - 3 files:
```solidity

Ethos-Core\contracts\BorrowerOperations.sol:

  175:    ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);

  283:    ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L175


```solidity
Ethos-Core\contracts\StabilityPool.sol:

  687:    Snapshots memory snapshots = depositSnapshots[_depositor];

  722:    Snapshots memory snapshots = depositSnapshots[_depositor];

  806:    address[] memory collaterals = collateralConfig.getAllowedCollaterals();

  857:    address[] memory collaterals = collateralConfig.getAllowedCollaterals();

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L687

```solidity
Ethos-Core\contracts\LQTY\LQTYStaking.sol:

  226:    address[] memory collaterals = collateralConfig.getAllowedCollaterals();

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L226


### [G-11] Multiple ``address``/ID mappings can be combined into a single ``mapping`` of an ``address``/ID to a ``struct``, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.


7 results 2 files:
```solidity
Ethos-Core\contracts\LUSDToken.sol:

  62:     mapping (address => bool) public troveManagers;
  63:     mapping (address => bool) public stabilityPools;
  64:     mapping (address => bool) public borrowerOperations;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L62-L64

```solidity
Ethos-Core\contracts\TroveManager.sol:

  104:     mapping (address => uint) public L_Collateral;
  105:     mapping (address => uint) public L_LUSDDebt;


  119:     mapping (address => uint) public lastCollateralError_Redistribution;
  120:     mapping (address => uint) public lastLUSDDebtError_Redistribution;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L104-L105


### [G-12] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata.

30 results - 4 files:
```solidity
Ethos-Core\contracts\ActivePool.sol:

  // @audit LUSDDebt[_collateral]
  
  193:    LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount);

  194:    ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);

  200:    LUSDDebt[_collateral] = LUSDDebt[_collateral].sub(_amount);

  201:    ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);


// @audit yieldGenerator[_collateral] 

  245:    vars.yieldGenerator = IERC4626(yieldGenerator[_collateral]);

  278:    IERC20(_collateral).safeIncreaseAllowance(yieldGenerator[_collateral], uint256(vars.netAssetMovement));

  279:    IERC4626(yieldGenerator[_collateral]).deposit(uint256(vars.netAssetMovement), address(this));

  281:    IERC4626(yieldGenerator[_collateral]).withdraw(uint256(-vars.netAssetMovement), address(this), address(this));

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L193


```solidity
Ethos-Core\contracts\StabilityPool.sol:

  // @audit depositSnapshots[_depositor]

  825:    depositSnapshots[_depositor].P = currentP;

  826:    depositSnapshots[_depositor].G = currentG;

  827:    depositSnapshots[_depositor].scale = currentScaleCached;

  828:    depositSnapshots[_depositor].epoch = currentEpochCached;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L825-L828


```solidity
Ethos-Core\contracts\TroveManager.sol:

  // @audit Troves[_borrower][_collateral]

  969:    Troves[_borrower][_collateral].debt = _newDebt;

  970:    Troves[_borrower][_collateral].coll = _newColl;

  977:    Troves[_borrower][_collateral].stake,


  //@audit Troves[_borrower][_collateral]

  1091:   Troves[_borrower][_collateral].coll = Troves[_borrower][_collateral].coll.add(pendingCollateralReward);

  1092:   Troves[_borrower][_collateral].debt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward);

  1102:   Troves[_borrower][_collateral].debt,

  1103:   Troves[_borrower][_collateral].coll,

  1104:   Troves[_borrower][_collateral].stake,


  // @audit Troves[_borrower][_collateral]

  1284:   Troves[_borrower][_collateral].status = closedStatus;

  1285:   Troves[_borrower][_collateral].coll = 0;

  1286:   Troves[_borrower][_collateral].debt = 0;


  // @audit rewardSnapshots[_borrower][_collateral]

  1288:   rewardSnapshots[_borrower][_collateral].collAmount = 0;

  1289:   rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L969-L970


```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  // @audit strategies[_strategy]

  180:    require(strategies[_strategy].activation != 0, "Invalid strategy address");

  182:    strategies[_strategy].feeBPS = _feeBPS;

  183:    emit StrategyFeeBPSUpdated(_strategy, _feeBPS);


  // @audit strategies[_strategy]

  210:    if (strategies[_strategy].allocBPS == 0) {

  214:    totalAllocBPS -= strategies[_strategy].allocBPS;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L180-L183


### [G-13] The result of function calls should be cached rather than re-calling the function

4 result - 1 file:
```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  //@audit token.balanceOf()
  368:     if (value > token.balanceOf(address(this))) {
  
  //@audit token.balanceOf()
  373:     vaultBalance = token.balanceOf(address(this));
  
  //@audit token.balanceOf()
  386:     uint256 actualWithdrawn = token.balanceOf(address(this)) - vaultBalance;
 
  //@audit token.balanceOf()
  399:     vaultBalance = token.balanceOf(address(this));

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L368


### [G-14] Avoid using ``state variable`` in emit

Using the current memory value here will be more gas-optimized.

1 result - 1 file:

```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  578:     function updateTvlCap(uint256 _newTvlCap) public {
  579:         _atLeastRole(ADMIN);
  580:         tvlCap = _newTvlCap;
  581:         emit TvlCapUpdated(tvlCap);
  582:     }

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L581

```diff
Ethos-Vault\contracts\ReaperVaultV2.sol:

  578:     function updateTvlCap(uint256 _newTvlCap) public {
  579:         _atLeastRole(ADMIN);
  580:         tvlCap = _newTvlCap;
- 581:         emit TvlCapUpdated(tvlCap);
+ 581:         emit TvlCapUpdated(_newTvlCap);
  582:     }

```

### [G-15] Superfluous event fields

``block.timestamp`` and ``block.number`` are added to event information by default so adding them manually wastes gas.

1 result - 1 file:
```solidity
Ethos-Core\contracts\TroveManager.sol:

  1500     function _updateLastFeeOpTime() internal {
  1501         uint timePassed = block.timestamp.sub(lastFeeOperationTime);
  1502 
  1503         if (timePassed >= SECONDS_IN_ONE_MINUTE) {
  1504             lastFeeOperationTime = block.timestamp;
  1505:             emit LastFeeOpTimeUpdated(block.timestamp);
  1506         }
  1507     }

  ```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1505

```diff
Ethos-Core\contracts\TroveManager.sol:

 1500     function _updateLastFeeOpTime() internal {
 1501         uint timePassed = block.timestamp.sub(lastFeeOperationTime);
 1502 
 1503         if (timePassed >= SECONDS_IN_ONE_MINUTE) {
 1504             lastFeeOperationTime = block.timestamp;
-1505:             emit LastFeeOpTimeUpdated(block.timestamp);
+ 1505:             emit LastFeeOpTimeUpdated(lastFeeOperationTime);
 1506         }
 1507     }

  ```

### [G-16] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including ``EXTCODESIZE`` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

97 results - 9 files:
```solidity
Ethos-Core\contracts\BorrowerOperations.sol:

  //@audit getCollateralCCR()
  178:      vars.collCCR = collateralConfig.getCollateralCCR(_collateral);
  286:      vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

  //@audit getCollateralMCR()
  179:      vars.collMCR = collateralConfig.getCollateralMCR(_collateral);
  287:      vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

  //@audit getCollateralDecimals()
  180:      vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  288:      vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  382:      uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

  //@audit getTroveColl()
  388:      uint coll = troveManagerCached.getTroveColl(msg.sender, _collateral);

  //@audit getTroveDebt()
  389:      uint debt = troveManagerCached.getTroveDebt(msg.sender, _collateral);
 
  //@audit getBorrowingFee()
  421:      uint LUSDFee = _troveManager.getBorrowingFee(_LUSDAmount);

  //@audit increaseTroveColl(), decreaseTroveColl()
  468:      uint newColl = (_isCollIncrease) ? _troveManager.increaseTroveColl(_borrower, _collateral, _collChange)
  469:                                     : _troveManager.decreaseTroveColl(_borrower, _collateral, _collChange);
  470:      uint newDebt = (_isDebtIncrease) ? _troveManager.increaseTroveDebt(_borrower, _collateral, _debtChange)
  471:                                     : _troveManager.decreaseTroveDebt(_borrower, _collateral, _debtChange);

  //@audit isCollateralAllowed()
  525:      require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");

  //@audit balanceOf()
  529:      require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");
  645:      require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");
  
  //@audit allowance()
  530:      require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");

  //@audit getTroveStatus()
  542:      uint status = _troveManager.getTroveStatus(_borrower, _collateral);
  547:      uint status = _troveManager.getTroveStatus(_borrower, _collateral);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L178


```solidity
Ethos-Core\contracts\TroveManager.sol:

  //@audit getCollateralDecimals()
  420:      uint collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  522:      vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  596:      vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  725:      vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  798:      vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  1048:     uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  1061:     uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  1131:     uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  1146:     uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  1362:     uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
  1368:     uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

  //@audit getCollateralMCR()
  523:      vars.collMCR = collateralConfig.getCollateralMCR(_collateral);
  598:      vars.collMCR = collateralConfig.getCollateralMCR(_collateral);
  727:      vars.collMCR = collateralConfig.getCollateralMCR(_collateral);
  800:      vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

  //@audit fetchPrice()
  524:      vars.price = priceFeed.fetchPrice(_collateral);
  728:      vars.price = priceFeed.fetchPrice(_collateral);
  
  //@audit getCollateralCCR()
  521:      vars.collCCR = collateralConfig.getCollateralCCR(_collateral);
  597:      vars.collCCR = collateralConfig.getCollateralCCR(_collateral);
  726:      vars.collCCR = collateralConfig.getCollateralCCR(_collateral);
  799:      vars.collCCR = collateralConfig.getCollateralCCR(_collateral);
  
  //@audit getLast()
  606:      vars.user = _sortedTroves.getLast(_collateral);
  691:      vars.user = sortedTrovesCached.getLast(_collateral);

  //@audit getFirst()
  607:      address firstUser = _sortedTroves.getFirst(_collateral);

  //@audit getPrev()
  610:      address nextUser = _sortedTroves.getPrev(_collateral, vars.user);  

   //@audit getTotalLUSDDeposits()
  729:      vars.LUSDInStabPool = stabilityPoolCached.getTotalLUSDDeposits();

  //@audit getCollateral()
  1308:     uint activeColl = _activePool.getCollateral(_collateral);
  1309:     uint liquidatedColl = defaultPool.getCollateral(_collateral);

  //@audit getSize()
  1539:     require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L420


```solidity
Ethos-Core\contracts\ActivePool.sol:

  //@audit getAllowedCollaterals()
  105:      address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();

  //@audit asset()
  111:      require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");

  //@audit deposit()
  280:      IERC4626(yieldGenerator[_collateral]).deposit(uint256(vars.netAssetMovement), address(this));
  
  //@audit withdraw()
  282:      IERC4626(yieldGenerator[_collateral]).withdraw(uint256(-vars.netAssetMovement), address(this), address(this));

  //@audit balanceOf()
  288:      vars.profit = IERC20(_collateral).balanceOf(address(this)).add(vars.yieldingAmount).sub(collAmount[_collateral]);

  //@audit isCollateralAllowed()
  315:      ICollateralConfig(collateralConfigAddress).isCollateralAllowed(_collateral),

  //@audit redemptionHelper()
  328:      address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L105


```solidity
Ethos-Core\contracts\StabilityPool.sol:

  //@audit redemptionHelper()
  328:      address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());

  //@audit issueOath()
  417:      uint LQTYIssuance = _communityIssuance.issueOath();

  //@audit getAllowedCollaterals()
  638:      assets = collateralConfig.getAllowedCollaterals();
  806:      address[] memory collaterals = collateralConfig.getAllowedCollaterals();
  857:      address[] memory collaterals = collateralConfig.getAllowedCollaterals();

  //@audit getCollateralDecimals()
  664:      vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

  //@audit fetchPrice()
  861:      uint price = priceFeed.fetchPrice(collateral);

  //@audit getLast()
  862:      address lowestTrove = sortedTroves.getLast(collateral);

  //@audit getCollateralMCR()
  863:      uint256 collMCR = collateralConfig.getCollateralMCR(collateral);

  //@audit getCurrentICR()
  864:      uint ICR = troveManager.getCurrentICR(lowestTrove, collateral, price);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L328

```solidity
Ethos-Core\contracts\LQTY\CommunityIssuance.sol:

  //@audit transferFrom()
  103:      OathToken.transferFrom(msg.sender, address(this), amount);

  //@audit transfer()
  127:      OathToken.transfer(_account, _OathAmount);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103

```solidity
Ethos-Core\contracts\LQTY\LQTYStaking.sol:

  //@audit transfer()
  135:      lusdToken.transfer(msg.sender, LUSDGain);
  171:      lusdToken.transfer(msg.sender, LUSDGain);

  //@audit getAllowedCollaterals()
  204:      assets = collateralConfig.getAllowedCollaterals();
  226:      address[] memory collaterals = collateralConfig.getAllowedCollaterals();

  //@audit redemptionHelper()
  252:      address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L135

```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  //@audit vault()
  153:      require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");
  
  //@audit want()
  154:      require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");

  //@audit balanceOf()
  246:      available = Math.min(available, token.balanceOf(address(this)));
  279:      return token.balanceOf(address(this)) + totalAllocated;
  303:      _deposit(token.balanceOf(msg.sender), msg.sender);
  327:      uint256 balBefore = token.balanceOf(address(this));
  329:      uint256 balAfter = token.balanceOf(address(this));
  368:      if (value > token.balanceOf(address(this))) {
  373:      vaultBalance = token.balanceOf(address(this));
  386:      uint256 actualWithdrawn = token.balanceOf(address(this)) - vaultBalance;
  556:      return IStrategy(vars.stratAddr).balanceOf();
  641:      uint256 amount = IERC20Metadata(_token).balanceOf(address(this));

  //@audit withdraw()
  385:      uint256 loss = IStrategy(stratAddr).withdraw(Math.min(remaining, strategyBal));

  //@audit decimals()
  651:      return token.decimals();

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L153

```solidity
Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:

  //@audit availableCapital()
  111:      int256 availableCapital = IVault(vault).availableCapital();

  //@audit report()
  134:      debt = IVault(vault).report(roi, repayment);

Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol:

  //@audit UNDERLYING_ASSET_ADDRESS()
  69:       want = _gWant.UNDERLYING_ASSET_ADDRESS();

  //@audit balanceOf()
  120:      uint256 amount = startToken.balanceOf(address(this));
  227:      return IERC20Upgradeable(want).balanceOf(address(this));

  //@audit strategies()
  127:      uint256 allocated = IVault(vault).strategies(address(this)).allocated;

  //@audit getLendingPool()
  178:      address lendingPoolAddress = ADDRESSES_PROVIDER.getLendingPool();
  200:      ILendingPool(ADDRESSES_PROVIDER.getLendingPool()).withdraw(address(want), _withdrawAmount, address(this));

  //@audit claimAllRewardsToSelf()
  207:      IRewardsController(REWARDER).claimAllRewardsToSelf(rewardClaimingTokens);

  //@audit getUserReserveData(
  231:      (uint256 supply, , , , , , , , ) = IAaveProtocolDataProvider(DATA_PROVIDER).getUserReserveData(

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L111



### [G-17] ``bytes`` constants are more eficient than ``string`` constans

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.


8 results - 6 files:
```solidity
Ethos-Core\contracts\ActivePool.sol:

  30:     string constant public NAME = "ActivePool";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30

```solidity
Ethos-Core\contracts\BorrowerOperations.sol:

  21:     string constant public NAME = "BorrowerOperations";

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21

```solidity
Ethos-Core\contracts\LUSDToken.sol:

  32:     string constant internal _NAME = "LUSD Stablecoin";

  33:     string constant internal _SYMBOL = "LUSD";

  34:     string constant internal _VERSION = "1";

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L32-L34

```solidity
Ethos-Core\contracts\StabilityPool.sol:

  150:    string constant public NAME = "StabilityPool";

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150

```solidity
Ethos-Core\contracts\LQTY\CommunityIssuance.sol:

  19:     string constant public NAME = "CommunityIssuance";

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19

```solidity
Ethos-Core\contracts\LQTY\LQTYStaking.sol:

  23:     string constant public NAME = "LQTYStaking";

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23


### [G-18] Change ``public`` function visibility to ``external``

Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

3 results - 2 files:
```solidity
Ethos-Core\contracts\TroveManager.sol:

  1045:     function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

  1440:     function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1045


```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  295:     function getPricePerFullShare() public view returns (uint256) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L295


### [G-19] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html 

Use a larger size then downcast where needed.

6 results 3 files:
```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  // @audit uint8 decimals()
  651:     return token.decimals();

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L651

```solidty
Ethos-Core\contracts\TroveManager.sol:

  // @audit uint128 index
  1329:    index = uint128(TroveOwners[_collateral].length.sub(1));
  1332:    return index;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1329

```solidty
Ethos-Core\contracts\StabilityPool.sol:

  // @audit uint128 currentScaleCached
  585:     currentScale = currentScaleCached.add(1);

  // @audit uint128 currentEpochCached
  576:     currentEpoch = currentEpochCached.add(1);

  // @audit uint128 scaleSnapshot
  745:     uint128 scaleDiff = currentScale.sub(scaleSnapshot);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L585


### [G-20] ``internal`` functions only called once can be inlined to save gas

Not inlining costs **20** to **40** gas because of two extra ``JUMP`` instructions and additional stack operations needed for function calls.

45 results - 9 files:
```solidity
Ethos-Core\contracts\ActivePool.sol:

  320:    function _requireCallerIsBorrowerOperationsOrDefaultPool() internal view {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L320

```solidity
Ethos-Core\contracts\BorrowerOperations.sol:

  524:    function _requireValidCollateralAddress(address _collateral) internal view {

  533:    function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {

  537:    function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {

  546:    function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {

  551:    function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {
  
  555     function _requireNotInRecoveryMode(
  556         address _collateral,
  557         uint _price,
  558         uint256 _CCR,
  559         uint256 _collateralDecimals
  560:     ) internal view {
  
  567:    function _requireNoCollWithdrawal(uint _collWithdrawal) internal pure {
  
  571     function _requireValidAdjustmentInCurrentMode 
  572     (
  573         bool _isRecoveryMode,
  574         address _collateral,
  575         uint _collWithdrawal,
  576         bool _isDebtIncrease, 
  577         LocalVariables_adjustTrove memory _vars
  578     ) 
  579:         internal 
  
  624:    function _requireNewICRisAboveOldICR(uint _newICR, uint _oldICR) internal pure {

  636:    function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure {

  640:    function _requireCallerIsStabilityPool() internal view {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L524

```solidity
Ethos-Core\contracts\LUSDToken.sol:

  320:    function _mint(address account, uint256 amount) internal {

  328:    function _burn(address account, uint256 amount) internal {

  360:    function _requireCallerIsBorrowerOperations() internal view {

  365:    function _requireCallerIsBOorTroveMorSP() internal view {

  375:    function _requireCallerIsStabilityPool() internal view {

  380:    function _requireCallerIsTroveMorSP() internal view {

  393:    function _requireMintingNotPaused() internal view {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L320


```solidity
Ethos-Core\contracts\StabilityPool.sol:

  421:    function _updateG(uint _LQTYIssuance) internal {

  439:    function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {

  597:    function _moveOffsetCollAndDebt(address _collateral, uint _collToAdd, uint _debtToOffset) internal {

  637:    function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {
  
  693:    function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {
  
  776:    function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {

  794:    function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {

  848:    function _requireCallerIsActivePool() internal view {

  852:    function _requireCallerIsTroveManager() internal view {

  856:    function _requireNoUnderCollateralizedTroves() internal {

  869:    function _requireUserHasDeposit(uint _initialDeposit) internal pure {

  873:    function _requireNonZeroAmount(uint _amount) internal pure {

```


```solidity
Ethos-Core\contracts\TroveManager.sol:

  1082:   function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {
  
  1213:   function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {
  
  1321:   function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
  
  1339:   function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {
  
  1516:   function _minutesPassedSinceLastFeeOp() internal view returns (uint) {
  
  1538:   function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1082

```solidity
Ethos-Core\contracts\LQTY\LQTYStaking.sol:
  
  251:    function _requireCallerIsTroveManagerOrActivePool() internal view {
  
  261:    function _requireCallerIsBorrowerOperations() internal view {
  
  265:    function _requireUserHasStake(uint currentStake) internal pure {  
  
  269:    function _requireNonZeroAmount(uint _amount) internal pure {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L251

```solidity
Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol:
  
  176:    function _deposit(uint256 toReinvest) internal {
  
  206:    function _claimRewards() internal {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176


```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:
  
  462:    function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {

Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:
  
  250:    function _liquidateAllPositions() internal virtual returns (uint256 amountFreed);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L462


### [G-21] Do not calculate constants variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

11 results - 3 files:
```solidity
Ethos-Core\contracts\TroveManager.sol:

  54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
  
  55:     uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%

```


```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
  
  74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
  
  75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
  
  76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```



```solidity
Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:
  
  25:     uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;
  
  49:     bytes32 public constant KEEPER = keccak256("KEEPER");
  
  50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
  
  51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
  
  52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```
**Recommendation Code:**
```diff

Ethos-Core\contracts\TroveManager.sol:

- 54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%

+            // REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
+ 54:     uint constant public override REDEMPTION_FEE_FLOOR = 5e15;

Ethos-Vault\contracts\ReaperVaultV2.sol:

- 73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
+ 73:     bytes32 public constant DEPOSITOR = 0xe16b3d8fc79140c62874442c8b523e98592b429e73c0db67686a5b378b29f336;

```


### [G-22] Use ``assembly`` to write _address storage values_ 

18 results - 6 files:
```solidity
Ethos-Core\contracts\LUSDToken.sol:

   84     constructor
  102:         troveManagerAddress = _troveManagerAddress;
  106:         stabilityPoolAddress = _stabilityPoolAddress;
  110:         borrowerOperationsAddress = _borrowerOperationsAddress;
  114:         governanceAddress = _governanceAddress;
  117:         guardianAddress = _guardianAddress;

  146     function updateGovernance(address _newGovernanceAddress) external {
  149:         governanceAddress = _newGovernanceAddress;

  153     function updateGuardian(address _newGuardianAddress) external {
  156:         guardianAddress = _newGuardianAddress;

  160     function upgradeProtocol(
  170:         troveManagerAddress = _newTroveManagerAddress;
  174:         stabilityPoolAddress = _newStabilityPoolAddress;
  178:         borrowerOperationsAddress = _newBorrowerOperationsAddress;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L102

```solidity
Ethos-Core\contracts\TroveManager.sol:
  
  232     function setAddresses(
  266:         borrowerOperationsAddress = _borrowerOperationsAddress;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L266

```solidity
Ethos-Core\contracts\LQTY\CommunityIssuance.sol:

  61     function setAddresses
  75:         stabilityPoolAddress = _stabilityPoolAddress;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L75

```solidity
Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol:

  62:    function initialize(
  68:         gWant = _gWant;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L68


```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  111     constructor(
  123:         tvlCap = _tvlCap;
  124:         treasury = _treasury;

  627     function updateTreasury(address newTreasury) external {
  630:         treasury = newTreasury;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L123-L124


```solidity
Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:
 
  63     function __ReaperBaseStrategy_init(
  72:         vault = _vault;
  73:         want = _want;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L72-L73


**Recommendation Code:**
```diff

Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:
 
  63     function __ReaperBaseStrategy_init(
- 72:         vault = _vault;
+                  assembly {                      
+                      sstore(vault.slot, _vault )
+                  }                               
          
```


### [G-23] Setting the _constructor_ to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ```msg.value == 0``` and saves ```13 gas``` on deployment with no security risks.

6 results - 6 files:
```solidity
Ethos-Core\contracts\LUSDToken.sol:

  84:     constructor

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L84

```solidity
Ethos-Core\contracts\TroveManager.sol:

  225:     constructor() public {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L225

```solidity
Ethos-Core\contracts\LQTY\CommunityIssuance.sol:

  57:     constructor() public {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L57

```solidity
Ethos-Vault\contracts\ReaperVaultERC4626.sol:

  16:     constructor(

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L16

```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  111:     constructor(

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111

```solidity
Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:

  61:     constructor() initializer {}

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L61

**Recommendation:**
Set the constructor to ```payable```

### [G-24] Add ``unchecked {}`` for subtractions where the operands cannot underflow because of a previous ``require`` or ``if`` statement

```
require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a } 
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
```
This will stop the check for overflow and underflow so it will save gas.

10 results - 2 files:
```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  144     function addStrategy(
  156         require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");
  168:         totalAllocBPS += _allocBPS;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L156

```solidity
Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol:

   99         if (_amountNeeded > liquidatedAmount) {
  100:             loss = _amountNeeded - liquidatedAmount;

  131         if (totalAssets > allocated) {
  132:             uint256 profit = totalAssets - allocated;
  135         } else if (totalAssets < allocated) {
  136:             roi = -int256(allocated - totalAssets);
  137         }


  225     function availableCapital() public view returns (int256) {
  234         if (stratCurrentAllocation > stratMaxAllocation) {
  235:             return -int256(stratCurrentAllocation - stratMaxAllocation);
  236:         } else if (stratCurrentAllocation < stratMaxAllocation) {
  244:             uint256 available = stratMaxAllocation - stratCurrentAllocation;


  359     function _withdraw(
  374                 if (value <= vaultBalance) {
  384:                 uint256 remaining = value - vaultBalance;


  493     function report(int256 _roi, uint256 _repayment) external returns (uint256) {
  534         if (vars.lockedProfitBeforeLoss > vars.loss) {
  535:             lockedProfit = vars.lockedProfitBeforeLoss - vars.loss;


  109     function harvest() external override returns (int256 roi) {
  120             if (amountFreed < debt) {
  121:                 roi = -int256(debt - amountFreed);
  122             } else if (amountFreed > debt) {
  123:                 roi = int256(amountFreed - debt);
  124             }

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L100


### [G-25] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops

In the use of for loops, this structure, which will reduce gas costs. This saves 30-40 gas per loop.

13 results - 3 files:
```solidity

Ethos-Core\contracts\StabilityPool.sol:

  351:    for (uint i = 0; i < numCollaterals; i++) {

  397:    for (uint i = 0; i < numCollaterals; i++) {

  640:    for (uint i = 0; i < assets.length; i++) {

  810:    for (uint i = 0; i < collaterals.length; i++) {

  831:    for (uint i = 0; i < collaterals.length; i++) {

  859:    for (uint i = 0; i < numCollaterals; i++) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351

```solidity
Ethos-Core\contracts\TroveManager.sol:

  608:    for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {

  690:    for (vars.i = 0; vars.i < _n; vars.i++) {

  808:    for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

  882:    for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608

```solidity
Ethos-Core\contracts\LQTY\LQTYStaking.sol:

  206:    for (uint i = 0; i < assets.length; i++) {

  228:    for (uint i = 0; i < collaterals.length; i++) {

  240:    for (uint i = 0; i < numCollaterals; i++) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206


**Recommendation Code:**
```diff
- 206:    for (uint i = 0; i < assets.length; i++) {
+ 206:    for (uint i = 0; i < assets.length; i = unchecked_inc(i)) {

+ function unchecked_inc(uint i) internal pure returns (uint) {
+       unchecked {
+           return i + 1;
+        }
+    }

```

### [G-26] `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables

9 results - 1 file:
```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  168:    totalAllocBPS += _allocBPS;

  194:    totalAllocBPS -= strategies[_strategy].allocBPS;

  196:    totalAllocBPS += _allocBPS;

  214:    totalAllocBPS -= strategies[_strategy].allocBPS;

  396:    totalAllocated -= actualWithdrawn;

  445:    totalAllocBPS -= bpsChange;

  452:    totalAllocated -= loss;
  
  515:    totalAllocated -= vars.debtPayment;

  521:    totalAllocated += vars.credit;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168

`x += y (x -= y)` costs more gas than `x = x  + y (x = x - y)` for state variables.


### [G-27] Use ``require`` instead of ``assert``

The assert() and require() functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made.

20 results - 4 files:
```solidity
Ethos-Core\contracts\BorrowerOperations.sol:

  128:     assert(MIN_NET_DEBT > 0);

  197:     assert(vars.compositeDebt > 0);

  301:     assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

  331:     assert(_collWithdrawal <= vars.coll); 

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L128


```solidity
Ethos-Core\contracts\LUSDToken.sol:

  312:     assert(sender != address(0));

  313:     assert(recipient != address(0));

  321:     assert(account != address(0));

  329:     assert(account != address(0));

  337:     assert(owner != address(0));

  338:     assert(spender != address(0));

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312


```solidity
Ethos-Core\contracts\StabilityPool.sol:

  526:     assert(_debtToOffset <= _totalLUSDDeposits);

  551:     assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);

  591:     assert(newP > 0);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L526


```solidity
Ethos-Core\contracts\TroveManager.sol:

   417:    assert(_LUSDInStabPool != 0);

  1224:    assert(totalStakesSnapshot[_collateral] > 0);

  1279:    assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

  1342:    assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

  1348:    assert(index <= idxLast);

  1414:    assert(newBaseRate > 0); // Base rate is always non-zero after redemption

  1489:    assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L417


### [G-28] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

3 results - 2 files:
```solidity
Ethos-Core\contracts\ActivePool.sol:

  264:         if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L264

```solidity
Ethos-Core\contracts\BorrowerOperations.sol:

  311:         if (_isDebtIncrease && !isRecoveryMode) { 

  337:         if (!_isDebtIncrease && _LUSDChange > 0) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L311


**Recomendation Code:**
```diff
Ethos-Core\contracts\BorrowerOperations.sol:

- 311:         if (_isDebtIncrease && !isRecoveryMode) {
+                 if (_isDebtIncrease) { 
+                 if (!isRecoveryMode) { 
+              }
+          }

```

### [G-29] Use ``double require`` instead of using ``&&``

Using double require instead of operator && can save more gas
When having a require statement with 2 or more expressions needed, place the expression that cost less gas first.

4 results - 3 files:
```solidity
Ethos-Core\contracts\BorrowerOperations.sol:

  653:     require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653

```solidity
Ethos-Core\contracts\LUSDToken.sol:

  347:     require(
  348:         _recipient != address(0) && 
  349:         _recipient != address(this),
  350:         "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
  351:     );

  352:     require(
  353:         !stabilityPools[_recipient] &&
  354:         !troveManagers[_recipient] &&
  355:         !borrowerOperations[_recipient],
  356:         "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
  357:     );

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347


```solidity
Ethos-Core\contracts\TroveManager.sol:

  1539:    require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539


**Recommendation Code:**
 ```diff
Ethos-Core\contracts\TroveManager.sol:

- 1539:    require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
+              require (TroveOwnersArrayLength > 1);
+              require (sortedTroves.getSize(_collateral) > 1);

```

### [G-30] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses ```OR/AND``` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
14 results - 5 files:
```solidity
Ethos-Core\contracts\TroveManager.sol:

   616:    if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { break; }

   817:    if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { continue; }
   
  1539:   require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

  1531:   require(msg.sender == borrowerOperationsAddress || msg.sender == address(redemptionHelper));

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L616


```solidity
Ethos-Vault\contracts\ReaperVaultERC4626.sol:

   52:      if (totalSupply() == 0 || _freeFunds() == 0) return assets;

   80:      if (emergencyShutdown || balance() >= tvlCap) return 0;

  123:     if (emergencyShutdown || balance() >= tvlCap) return 0;

  184:     if (totalSupply() == 0 || _freeFunds() == 0) return 0;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L52


```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:

  227:     if (totalAllocBPS == 0 || emergencyShutdown) {

  555:     if (strategy.allocBPS == 0 || emergencyShutdown) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L227


```solidity
Ethos-Core\contracts\BorrowerOperations.sol:

  311:     if (_isDebtIncrease && !isRecoveryMode) { 

  337:     if (!_isDebtIncrease && _LUSDChange > 0) {

  653:     require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L311


```solidity
Ethos-Core\contracts\LUSDToken.sol:

  347:     require(
  348         _recipient != address(0) && 
  349         _recipient != address(this),

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347


### [G-31] Optimize names to save gas (22 gas)

Contracts most called functions could simply save gas by function ordering via ```Method ID```. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because ```22 gas``` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

**Context:** 
All Contracts


**Recommendation:** 
Find a lower ```method ID``` name for the most called functions for example Call() vs. Call1() is cheaper by ```22 gas```
For example, the function IDs in the ```ActivePool.sol``` contract will be the most used; A lower method ID may be given.

**Proof of Consept:**
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

ActivePool.sol function names can be named and sorted according to METHOD ID

```js
Sighash   |   Function Signature
========================
31585161  =>  manualRebalance(address,uint256)
c707f625  =>  setAddresses(address,address,address,address,address,address,address,address,address[])
8fca46f4  =>  setYieldingPercentage(address,uint256)
6dc85da8  =>  setYieldingPercentageDrift(uint256)
2bc75d35  =>  setYieldClaimThreshold(address,uint256)
46a80441  =>  setYieldDistributionParams(uint256,uint256,uint256)
9b56d6c9  =>  getCollateral(address)
4b14fa87  =>  getLUSDDebt(address)
715aa61b  =>  sendCollateral(address,address,uint256)
5fc7b5e8  =>  increaseLUSDDebt(address,uint256)
3aacc815  =>  decreaseLUSDDebt(address,uint256)
eb13cd10  =>  pullCollateralFromBorrowerOperationsOrDefaultPool(address,uint256)
e07a32fa  =>  _rebalance(address,uint256)
a388c48f  =>  _requireValidCollateralAddress(address)
cd0a39b5  =>  _requireCallerIsBorrowerOperationsOrDefaultPool()
9cfa3bca  =>  _requireCallerIsBOorTroveMorSP()
8daa3501  =>  _requireCallerIsBOorTroveM()
```

### [G-32] Use a more recent version of solidity

Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath
Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value
In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

12 results - 12 files:
```solidity
Ethos-Core\contracts\ActivePool.sol:
 
  3: pragma solidity 0.6.11;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3

```solidity
Ethos-Core\contracts\BorrowerOperations.sol:

  3: pragma solidity 0.6.11;
 
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3

```solidity
Ethos-Core\contracts\CollateralConfig.sol:
 
  3: pragma solidity 0.6.11;
 
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3

```solidity
Ethos-Core\contracts\LUSDToken.sol:
 
  3: pragma solidity 0.6.11;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3

```solidity
Ethos-Core\contracts\StabilityPool.sol:
 
  3: pragma solidity 0.6.11;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3


```solidity
Ethos-Core\contracts\TroveManager.sol:

  3: pragma solidity 0.6.11;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3

  
```solidity
Ethos-Core\contracts\LQTY\CommunityIssuance.sol:
 
  3: pragma solidity 0.6.11;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3
 
```solidity
Ethos-Core\contracts\LQTY\LQTYStaking.sol:
 
  3: pragma solidity 0.6.11;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3

 ```solidity
Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol:
 
  3: pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3
 
```solidity
Ethos-Vault\contracts\ReaperVaultERC4626.sol:
 
  3: pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3
  
```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol:
 
  3: pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3
 
```solidity
Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol:
  
  3: pragma solidity ^0.8.0;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3


### [G-33] ``>=`` costs less gas than ``>``

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

1 result - 1 file:
```solidity
Ethos-Core\contracts\LQTY\CommunityIssuance.sol:

  87:             uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L87


### [G-34] Upgrade Solidity's optimizer

Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.

```js
Ethos-Core\hardhat.config.js:
  38:     solidity: {
  39:         compilers: [
  40:             {
  41:                 version: "0.4.23",
  42:                 settings: {
  43:                     optimizer: {
  44:                         enabled: true,
  45:                         runs: 100
  46:                     }
  47:                 }
  48:             },
  49:             {
  50:                 version: "0.5.17",
  51:                 settings: {
  52:                     optimizer: {
  53:                         enabled: true,
  54:                         runs: 100
  55:                     }
  56:                 }
  57:             },
  58:             {
  59:                 version: "0.6.11",
  60:                 settings: {
  61:                     optimizer: {
  62:                         enabled: true,
  63:                         runs: 100
  64:                     }
  65:                 }
  66:             },
  67:         ]
  68:     },

```

```js
Ethos-Vault\hardhat.config.js:
  10: module.exports = {
  11:   solidity: {
  12:     compilers: [
  13:       {
  14:         version: "0.8.11",
  15:         settings: {
  16:           optimizer: {
  17:             enabled: true,
  18:             runs: 200,
  19:           },
  20:         },
  21:       },
  22:     ],
  23:   },
  24:   networks: {
  25:     mainnet: {
  26:       url: `https://rpc.ftm.tools`,
  27:       chainId: 250,
  28:       accounts: [`0x${PRIVATE_KEY}`],
  29:     },
  30:     testnet: {
  31:       url: `https://rpcapi-tracing.testnet.fantom.network`,
  32:       chainId: 4002,
  33:       accounts: [`0x${PRIVATE_KEY}`],
  34:     },
  35:   },
  36:   etherscan: {
  37:     apiKey: FTMSCAN_KEY,
  38:   },
  39:   mocha: {
  40:     timeout: 1200000,
  41:   },
  42:   gasReporter: {
  43:       enabled: true
  44:   }
  45: };

```