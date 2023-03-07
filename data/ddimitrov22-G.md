## Summary

|        | Issues                                                                                                     | Contexts | Estimated Gas Saved |
| ------ | ---------------------------------------------------------------------------------------------------------- | -------- | ------------------- |
| [G-01] | Setting the constructor to `payable`                                                                       | 6        | -                   |
| [G-02] | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables                                     | 22       | -                   |
| [G-03] | Empty blocks should be removed or emit something                                                           | 2        | -                   |
| [G-04] | Use a more recent version of solidity                                                                      | 5        | -                   |
| [G-05] | Splitting `require()` statements that use `&&` saves gas                                                   | 5        | -                   |
| [G-06] | Instead of emitting multiple events with a single value, emit a single event with multiple values          | 38       | -                   |
| [G-07] | Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant | 8        | -                   |
| [G-08] | Use `require` instead of `assert`                                                                          | 20       | -                   |
| [G-09] | Internal/Private functions only called once can be inlined to save gas                                     | 22       | -                   |
| [G-10] | Structs can be packed into fewer storage slots                                                             | 8        | -                   |
| [G-11] | Checking `vars.compositeDebt` to not be zero is redundant                                                  | 1        | -                   |

## [G-01] Setting the constructor to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor `payable` eliminates the need for an initial check of `msg.value == 0` and saves 13 gas on deployment with no security risks.

**Context**

    6 results - 6 files

    Ethos-Core/contracts/LUSDToken.sol:
    84:     constructor

    Ethos-Core/contracts/TroveManager.sol:
    225:     constructor() public {

    Ethos-Core/contracts/LQTY/CommunityIssuance.sol:
    57:     constructor() public {

    Ethos-Vault/contracts/ReaperVaultERC4626.sol:
    16:     constructor(

    Ethos-Vault/contracts/ReaperVaultV2.sol:
    111:     constructor(

    Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
    61:     constructor() initializer {}

## [G-02] `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

    8 results - 2 files

    Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol:
    133:             toFree += profit;

    Ethos-Vault/contracts/ReaperVaultV2.sol:
    168:         totalAllocBPS += _allocBPS;
    196:         totalAllocBPS += _allocBPS;
    391:                     totalLoss += loss;
    450:         stratParams.losses += loss;
    505:             strategy.gains += vars.gain;
    520:             strategy.allocated += vars.credit;
    521:             totalAllocated += vars.credit;

    14 results - 3 files

    Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol:
    141:         roi -= int256(loss);

    Ethos-Vault/contracts/ReaperVaultV2.sol:
    194:         totalAllocBPS -= strategies[_strategy].allocBPS;
    214:         totalAllocBPS -= strategies[_strategy].allocBPS;
    390:                     value -= loss;
    395:                 strategies[stratAddr].allocated -= actualWithdrawn;
    396:                 totalAllocated -= actualWithdrawn;
    444:                 stratParams.allocBPS -= bpsChange;
    445:                 totalAllocBPS -= bpsChange;
    451:         stratParams.allocated -= loss;
    452:         totalAllocated -= loss;
    514:                 strategy.allocated -= vars.debtPayment;
    515:                 totalAllocated -= vars.debtPayment;
    516:                 vars.debt -= vars.debtPayment; // tracked for return value

    Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
    128:                 repayment -= uint256(-roi);

## [G-03] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be `abstract` and the function signatures be added without any default implementation.

    2 results - 2 files

    Ethos-Vault/contracts/ReaperVaultERC4626.sol:
    24:     ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}

    Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
    61:     constructor() initializer {}

## [G-04] Use a more recent version of solidity

Use a solidity version of at least 0.8 to get default underflow/overflow checks, use a solidity version of at least 0.8.2 to get simple compiler automatic inlining Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

For example, we can avoid using the library safeMath in the following files by using version 0.8+:

    5 results - 5 files

    Ethos-Core/contracts/ActivePool.sol:
    3: pragma solidity 0.6.11;

    Ethos-Core/contracts/LUSDToken.sol:
    3: pragma solidity 0.6.11;

    Ethos-Core/contracts/StabilityPool.sol:
    3: pragma solidity 0.6.11;

    Ethos-Core/contracts/LQTY/CommunityIssuance.sol:
    3: pragma solidity 0.6.11;

    Ethos-Core/contracts/LQTY/LQTYStaking.sol:
    3: pragma solidity 0.6.11;

## [G-05] Splitting `require()` statements that use `&&` saves gas

Instead of using the && operator in a single require statement to check multiple conditions,using multiple require statements with 1 condition per require statement will save 8 GAS per &&.
The gas difference would only be realized if the revert condition is realized(met).

    5 results - 3 files

    Ethos-Core/contracts/BorrowerOperations.sol:
    653:             require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,

    Ethos-Core/contracts/LUSDToken.sol:
    348:             _recipient != address(0) &&
    353:             !stabilityPools[_recipient] &&
    354:             !troveManagers[_recipient] &&

    Ethos-Core/contracts/TroveManager.sol:
    1539:         require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

## [G-06] Instead of emitting multiple events with a single value, emit a single event with multiple values

There are many events which are emitted only once inside a `setAddress()` function with a single address value. Those functions can be called only once because they have `onlyOwner` modifier and call `_renounceOwnership()` at the end of the function. Thus, such events can be combined into a single event which will save additional gas.

**Context**

    38 results - 4 files

    Ethos-Core/contracts/BorrowerOperations.sol:
    155:         emit CollateralConfigAddressChanged(_collateralConfigAddress);
    156:         emit TroveManagerAddressChanged(_troveManagerAddress);
    157:         emit ActivePoolAddressChanged(_activePoolAddress);
    158:         emit DefaultPoolAddressChanged(_defaultPoolAddress);
    159:         emit StabilityPoolAddressChanged(_stabilityPoolAddress);
    160:         emit GasPoolAddressChanged(_gasPoolAddress);
    161:         emit CollSurplusPoolAddressChanged(_collSurplusPoolAddress);
    162:         emit PriceFeedAddressChanged(_priceFeedAddress);
    163:         emit SortedTrovesAddressChanged(_sortedTrovesAddress);
    164:         emit LUSDTokenAddressChanged(_lusdTokenAddress);
    165:         emit LQTYStakingAddressChanged(_lqtyStakingAddress);

    Ethos-Core/contracts/StabilityPool.sol:
    293:         emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);
    294:         emit CollateralConfigAddressChanged(_collateralConfigAddress);
    295:         emit TroveManagerAddressChanged(_troveManagerAddress);
    296:         emit ActivePoolAddressChanged(_activePoolAddress);
    297:         emit LUSDTokenAddressChanged(_lusdTokenAddress);
    298:         emit SortedTrovesAddressChanged(_sortedTrovesAddress);
    299:         emit PriceFeedAddressChanged(_priceFeedAddress);
    300:         emit CommunityIssuanceAddressChanged(_communityIssuanceAddress);

    Ethos-Core/contracts/TroveManager.sol:
    280:         emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);
    281:         emit CollateralConfigAddressChanged(_collateralConfigAddress);
    282:         emit ActivePoolAddressChanged(_activePoolAddress);
    283:         emit DefaultPoolAddressChanged(_defaultPoolAddress);
    284:         emit StabilityPoolAddressChanged(_stabilityPoolAddress);
    285:         emit GasPoolAddressChanged(_gasPoolAddress);
    286:         emit CollSurplusPoolAddressChanged(_collSurplusPoolAddress);
    287:         emit PriceFeedAddressChanged(_priceFeedAddress);
    288:         emit LUSDTokenAddressChanged(_lusdTokenAddress);
    289:         emit SortedTrovesAddressChanged(_sortedTrovesAddress);
    290:         emit LQTYTokenAddressChanged(_lqtyTokenAddress);
    291:         emit LQTYStakingAddressChanged(_lqtyStakingAddress);
    292:         emit RedemptionHelperAddressChanged(_redemptionHelperAddress);

    Ethos-Core/contracts/LQTY/LQTYStaking.sol:
    93:         emit LQTYTokenAddressSet(_lqtyTokenAddress);
    94:         emit LQTYTokenAddressSet(_lusdTokenAddress);
    95:         emit TroveManagerAddressSet(_troveManagerAddress);
    96:         emit BorrowerOperationsAddressSet(_borrowerOperationsAddress);
    97:         emit ActivePoolAddressSet(_activePoolAddress);
    98:         emit CollateralConfigAddressSet(_collateralConfigAddress);

**Proof of Concept**

The following test was done in Remix to show the difference between calling a function which emits 3 events, and a function which emits 1 event with 3 passed values:

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract Events {

    address public add1;
    address public add2;
    address public add3;


    // gas 3 events 107290
    // gas 1 event 105390
    event Add1(address _add1, address _add2, address _add3);

    function setAdd(address _add1, address _add2, address _add3) public {

        add1 = _add1;
        add2 = _add2;
        add3 = _add3;

        emit Add1(_add1, _add2, _add3);

        }
    }

`950 gas is saved per event.

## [G-07] Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant

    8 results - 2 files

    Ethos-Vault/contracts/ReaperVaultV2.sol:
    73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
    74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
    75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
    76:     bytes32 public constant ADMIN = keccak256("ADMIN");

    Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
    49:     bytes32 public constant KEEPER = keccak256("KEEPER");
    50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
    51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
    52:     bytes32 public constant ADMIN = keccak256("ADMIN");

## [G-08] Use `require` instead of `assert`

The `assert()` and `require()` functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the `assert()` function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a `require()` function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

**Context**
20 results - 4 files

    Ethos-Core/contracts/BorrowerOperations.sol:
    128:         assert(MIN_NET_DEBT > 0);
    197:         assert(vars.compositeDebt > 0);
    301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
    331:         assert(_collWithdrawal <= vars.coll);

    Ethos-Core/contracts/LUSDToken.sol:
    312:         assert(sender != address(0));
    313:         assert(recipient != address(0));
    321:         assert(account != address(0));
    329:         assert(account != address(0));
    337:         assert(owner != address(0));
    338:         assert(spender != address(0));

    Ethos-Core/contracts/StabilityPool.sol:
    526:         assert(_debtToOffset <= _totalLUSDDeposits);
    551:         assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
    591:         assert(newP > 0);

    Ethos-Core/contracts/TroveManager.sol:
    417:             assert(_LUSDInStabPool != 0);
    1224:             assert(totalStakesSnapshot[_collateral] > 0);
    1279:         assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
    1342:         assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
    1348:         assert(index <= idxLast);
    1414:         assert(newBaseRate > 0); // Base rate is always non-zero after redemption
    1489:         assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0

## [G-09] Internal/Private functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L265
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L462
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L438
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L455
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L476
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L533
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L537
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L546
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L551
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L555
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L567
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L571
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L624
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L636
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L661
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1082
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L864
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L785
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L671
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L582
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L478
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1538

## [G-10] Structs can be packed into fewer storage slots

Few of the structs across the contracts can be packed in order to save gas. Check this [tweet](https://twitter.com/0xCygaar/status/1630660838460735489) for a reference.

**Context**

    8 results - 3 files

    Ethos-Core/contracts/BorrowerOperations.sol:
    47:      struct LocalVariables_adjustTrove {
    66:     struct LocalVariables_openTrove {
    80:     struct ContractsCache {

    Ethos-Core/contracts/StabilityPool.sol:
    646:     struct LocalVariables_getSingularCollateralGain {

    Ethos-Core/contracts/TroveManager.sol:
    129:     struct LocalVariables_OuterLiquidationFunction {
    146:     struct LocalVariables_LiquidationSequence {
    160:     struct LiquidationValues {
    172:     struct LiquidationTotals {

## [G-11] Checking `vars.compositeDebt` to not be zero is redundant

There is an instance where `assert` check on`vars.compositeDebt` to be above zero is redundant. The `composideDebt` is a sum of LUSD amount + LUSD borrowing fee + LUSD gas comp. and it is impossible to be zero. Furthermore, few lines above the `netDebt` which is part of the `compositeDebt` is checked to be above zero.

**Context**

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L197
