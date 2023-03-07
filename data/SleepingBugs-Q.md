

# Low

| | Issue index |
| ----------- | ----------- |
| 1 | [New `tvlCap` can be less than current value locked](#new-`tvlcap`-can-be-less-than-current-value-locked) |
| 2 | [Use of `asserts()`](#use-of-`asserts()`) |
| 3 | [Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions](#upgradeable-contract-is-missing-a-`__gap[50]`-storage-variable-to-allow-for-new-storage-variables-in-later-versions) |
| 4 | [Missing important events emissions affects off-chain monitoring](#missing-important-events-emissions-affects-off-chain-monitoring) |
| 5 | [Critical address change should use two-step procedure](#critical-address-change-should-use-two-step-procedure) |
| 6 | [`updateCollateralRatios` is not safe without a two-step procedure](#`updatecollateralratios`-is-not-safe-without-a-two-step-procedure) |
| 7 | [`_safeTransfer()` should be use rather than `_transfer`](#`_safetransfer()`-should-be-use-rather-than-`_transfer`) |
| 8 | [`distributionPeriod` can be changed without anybody noticing and removing `rewardPerSecond` until changed back](#`distributionperiod`-can-be-changed-without-anybody-noticing-and-removing-`rewardpersecond`-until-changed-back) |
| 9 | [`_approve` can be frontrunned](#`_approve`-can-be-frontrunned) |
| 10 | [Wrong comment](#wrong-comment) |
| 11 | [Collateral is not checked to already exist, this leads to unexpected behaviors](#collateral-is-not-checked-to-already-exist,-this-leads-to-unexpected-behaviors) |
| 12 | [Missing `emit` keyword](#missing-`emit`-keyword) |
| 13 | [Single-step process for critical ownership transfer/renounce is risky](#single-step-process-for-critical-ownership-transfer/renounce-is-risky) |
## New `tvlCap` can be less than current value locked

Add a check to ensure new `tvlCap ` is not less than current value locked.

- [ReaperVaultV2.sol#L580](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L580)

## Use of `asserts()`
### Impact

From solidity docs: Properly functioning code should never reach a failing assert statement; if this happens there is a bug in your contract which you should fix.

With assert the user pays the gas and with require it doesn't. The ETH network gas isn't cheap. You have reachable asserts in the following locations (which should be replaced by `require` / are mistakenly left from development phase):

### Details

The Solidity `assert()` function is meant to assert invariants. Properly functioning code should never reach a failing assert statement. A reachable assertion can mean one of two things:

A bug exists in the contract that allows it to enter an invalid state;
The assert statement is used incorrectly, e.g. to validate inputs.

### References

SWC ID: 110

### Code Snippet

- [BorrowerOperations.sol#L128](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L128)
        assert(MIN_NET_DEBT > 0);
- [BorrowerOperations.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L197)
        assert(vars.compositeDebt > 0);
- [BorrowerOperations.sol#L301](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L301)
        assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
- [BorrowerOperations.sol#L331](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L331)
        assert(_collWithdrawal <= vars.coll); 
- [TroveManager.sol#L417](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L417)
            assert(_LUSDInStabPool != 0);
- [TroveManager.sol#L1219](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1219)
            * The following assert() holds true because:
- [TroveManager.sol#L1224](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1224)
            assert(totalStakesSnapshot[_collateral] > 0);
- [TroveManager.sol#L1279](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1279)
        assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
- [TroveManager.sol#L1342](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1342)
        assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
- [TroveManager.sol#L1348](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1348)
        assert(index <= idxLast);
- [TroveManager.sol#L1413](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1413)
        //assert(newBaseRate <= DECIMAL_PRECISION); // This is already enforced in the line above
- [TroveManager.sol#L1414](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1414)
        assert(newBaseRate > 0); // Base rate is always non-zero after redemption
- [TroveManager.sol#L1489](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1489)
        assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
- [StabilityPool.sol#L526](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L526)
        assert(_debtToOffset <= _totalLUSDDeposits);
- [StabilityPool.sol#L551](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L551)
        assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
- [StabilityPool.sol#L591](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L591)
        assert(newP > 0);
- [LUSDToken.sol#L312](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L312)
        assert(sender != address(0));
- [LUSDToken.sol#L313](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L313)
        assert(recipient != address(0));
- [LUSDToken.sol#L321](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L321)
        assert(account != address(0));
- [LUSDToken.sol#L329](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L329)
        assert(account != address(0));
- [LUSDToken.sol#L337](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L337)
        assert(owner != address(0));
- [LUSDToken.sol#L338](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L338)
        assert(spender != address(0));

### Recommendation

Substitute `asserts` with `require`/`revert`.

## Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions
### Impact 

For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments" (quote OpenZeppelin). Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts, potentially causing loss of user fund or cause the contract to malfunction completely. [See](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps).

For a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

### Vulnerability Details

In the following context of the upgradeable contracts they are expected to use gaps for avoiding collision:

- `UUPS`

However, none of these contracts contain storage gap. The storage gap is essential for upgradeable contract because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Refer to the bottom part of this [article](https://docs.openzeppelin.com/contracts/4.x/upgradeable).

If a contract inheriting from a base contract contains additional variable, then the base contract cannot be upgraded to include any additional variable, because it would overwrite the variable declared in its child contract. This greatly limits contract upgradeability.

### Code Snippet

- [ReaperBaseStrategyv4.sol#L17-L18](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L17-L18)

```solidity
    UUPSUpgradeable,
    AccessControlEnumerableUpgradeable
```

### Tools Used

Manual analysis

### Recommendation

Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. 

`uint256[50] private __gap;`

Reference OpenZeppelin [upgradeable contract templates](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable).


## Missing important events emissions affects off-chain monitoring

Add event emissions for important variables changes and actions.

- [LUSDToken.sol#L138](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L138)
- [LUSDToken.sol#L143](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L143)


## Critical address change should use two-step procedure

The critical procedures should be two step process.

- [LUSDToken.sol#L146](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L146)
- [LUSDToken.sol#L153](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L153)
- [LUSDToken.sol#L160](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L160)
Ethos-Core/contracts/CollateralConfig.sol

### Recommendation

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions and / or a timelock

## `updateCollateralRatios` is not safe without a two-step procedure

As notice at: "
        !!!PLEASE USE EXTREME CARE AND CAUTION. THIS IS IRREVERSIBLE!!!
        //
        // You probably don't want to do this unless a specific asset has proved itself through tough times.
        // Doing this irresponsibly can permanently harm the protocol."

This functions is critical. Therefore, using a two steps procedure would be optimal.

Consider adding a time before this change is completely done so it can be canceled, or do it in a 2 steps manner so it can be reviewed before setting it up.

- [CollateralConfig.solL85](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/CollateralConfig.solL85)

## `_safeTransfer()` should be use rather than `_transfer`

- [LUSDToken.sol#L198](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L198)
- [LUSDToken.sol#L203](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L203)

## `distributionPeriod` can be changed without anybody noticing and removing `rewardPerSecond` until changed back
 
`updateDistributionPeriod` doesn't use any kind of event, so if by error the owner sets the value to a high one, rewards per seconds at `fund` will not be distributed because of rounding down at `rewardPerSecond = amount.div(distributionPeriod);`

So this human fail can lead to an a not noticed change, due to missing the event, that will make rewards to be 0 until notice.

```solidity
        function fund(uint amount) external onlyOwner {
        require(amount != 0, "cannot fund 0");
        OathToken.transferFrom(msg.sender, address(this), amount);
        // roll any unissued OATH into new distribution
        if (lastIssuanceTimestamp < lastDistributionTime) {
            uint timeLeft = lastDistributionTime.sub(lastIssuanceTimestamp);
            uint notIssued = timeLeft.mul(rewardPerSecond);
            amount = amount.add(notIssued);
        }

        rewardPerSecond = amount.div(distributionPeriod); //@audit div by high value
        lastDistributionTime = block.timestamp.add(distributionPeriod);
        lastIssuanceTimestamp = block.timestamp;

        emit LogRewardPerSecond(rewardPerSecond);
    }

    // Owner-only function to update the distribution period
    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
        distributionPeriod = _newDistributionPeriod;
    }
```

- [CommunityIssuance.sol#L101-L121](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101-L121)

## `_approve` can be frontrunned

Consider to require to set allowance to 0 first as suggested [at](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/599), 

- [LUSDToken.sol#L336](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L336)



## Wrong comment
- [LUSDToken.sol#L210](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L210)

## Collateral is not checked to already exist, this leads to unexpected behaviors

- [ActivePool.sol#L112](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L112)

## Missing `emit` keyword

- [ActivePool.sol#L194](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L194)
- [ActivePool.sol#L201](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L201)
- [ActivePool.sol#L194](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L194)

## Single-step process for critical ownership transfer/renounce is risky
### Impact

Given that some contracts are derived from `Ownable`, the ownership management of this contract defaults to `Ownable` ’s `transferOwnership()` and `renounceOwnership()` methods which are not overridden here. 

Such critical address transfer/renouncing in one-step is very risky because it is irrecoverable from any mistakes

Scenario: If an incorrect address, e.g. for which the private key is not known, is used accidentally then it prevents the use of all the `onlyOwner()` functions forever, which includes the changing of various critical addresses and parameters. This use of incorrect address may not even be immediately apparent given that these functions are probably not used immediately. 

When noticed, due to a failing `onlyOwner()` function call, it will force the redeployment of these contracts and require appropriate changes and notifications for switching from the old to new address. This will diminish trust in the protocol and incur a significant reputational damage.

### Code Snippet

- `Ownable`

- [CollateralConfig.sol#L15](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/CollateralConfig.sol#L15)
contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {
- [BorrowerOperations.sol#L18](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L18)
contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOperations {
- [TroveManager.sol#L18](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L18)
contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager {
- [ActivePool.sol#L26](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L26)
contract ActivePool is Ownable, CheckContract, IActivePool {

- [StabilityPool.sol#L146](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L146)
contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {

- [CommunityIssuance.sol#L14](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L14)
contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMath {

- [LQTYStaking.sol#L18](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L18)
contract LQTYStaking is ILQTYStaking, Ownable, CheckContract, BaseMath {

### Recommendation

- Recommend using `Ownable2Step` from OpenZeppelin or `Owned` from Solmate

# Informational

| | Issue index |
| ----------- | ----------- |
| 1 | [Wrong place of underscore can lead to confusions](#wrong-place-of-underscore-can-lead-to-confusions) |
| 2 | [Constants should be defined rather than using magic numbers](#constants-should-be-defined-rather-than-using-magic-numbers) |
| 3 | [Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10 ** 18`)](#use-scientific-notation-(e.g.-`1e18`)-rather-than-exponentiation-(e.g.-`10-**-18`)) |
| 4 | [Mixed use of `type(uint).max` and `uint(-1)` affects consistency](#mixed-use-of-`type(uint).max`-and-`uint(-1)`-affects-consistency) |
| 5 | [Inconsistent spacing in comments](#inconsistent-spacing-in-comments) |
| 6 | [Non-assembly method available](#non-assembly-method-available) |
| 7 | [Wrong value of days slightly affects precision](#wrong-value-of-days-slightly-affects-precision) |
| 8 | [Missing indexed event parameters](#missing-indexed-event-parameters) |
| 9 | [Different versions of pragma](#different-versions-of-pragma) |
| 10 | [Use of `uint` and `uint256` for the same type affects consistency](#use-of-`uint`-and-`uint256`-for-the-same-type-affects-consistency) |
| 11 | [Missing error messages in require statements](#missing-error-messages-in-require-statements) |
| 12 | [Mixed styles as `mapping(` / `mapping (`, `for (` / `for(` are used without consistency, affecting searchability](#mixed-styles-as-`mapping(`-/-`mapping-(`,-`for-(`-/-`for(`-are-used-without-consistency,-affecting-searchability) |
| 13 | [Deprecated use of `now` rather than `block.timestamp`](#deprecated-use-of-`now`-rather-than-`block.timestamp`) |
| 14 | [Signature malleability of EVM's `ecrecover()`](#signature-malleability-of-evm's-`ecrecover()`) |
| 15 | [Missing idempotent check](#missing-idempotent-check) |
| 16 | [Use of `_renounceOwnership` can lead to unexpected behaviors](#use-of-`_renounceownership`-can-lead-to-unexpected-behaviors) |
| 17 | [Maximum line length exceeded](#maximum-line-length-exceeded) |
| 18 | [Names can be more specific](#names-can-be-more-specific) |
| 19 | [Unused dependency can be removed](#unused-dependency-can-be-removed) |
| 20 | [Strategy is not checked to be valid and therefore may not exist](#strategy-is-not-checked-to-be-valid-and-therefore-may-not-exist) |
## Wrong place of underscore can lead to confusions

Underscore is a valid symbol used for clarity while reading numbers, this symbol is used in general for separating hundreds (i.e: `4_000`) however, it is being used in a not typical position `40_00`.

- [ActivePool.sol#L52](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L52)
    uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
- [ActivePool.sol#L53](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L53)
    uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
- [ActivePool.sol#L54](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L54)
    uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS

## Constants should be defined rather than using magic numbers

- [ActivePool.sol#L127](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L127)
        require(_bps <= 10_000, "Invalid BPS value");
- [ActivePool.sol#L133](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L133)
        require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
- [ActivePool.sol#L145](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L145)
        require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
- [ActivePool.sol#L258](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L258)
        vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);
- [ActivePool.sol#L262](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L262)
        vars.finalYieldingAmount = vars.finalBalance.mul(vars.yieldingPercentage).div(10_000);
- [ActivePool.sol#L291](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L291)
                vars.treasurySplit = vars.profit.mul(yieldSplitTreasury).div(10_000);
- [ActivePool.sol#L296](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L296)
                vars.stakingSplit = vars.profit.mul(yieldSplitStaking).div(10_000);

- [StabilityPool.sol#L768](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L768)
        if (compoundedDeposit < initialDeposit.div(1e9)) {return 0;}

- [ActivePool.sol#L127](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L127)
        require(_bps <= 10_000, "Invalid BPS value");
- [ActivePool.sol#L145](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L145)
        require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");

## Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10 ** 18`)

- [ReaperVaultV2.sol#L40](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L40)
    uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
- [ReaperVaultV2.sol#L125](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L125)
        lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks

### Recommendation

Consider replacing `10 ** value` notation to scientific notation

## Mixed use of `type(uint).max` and `uint(-1)` affects consistency

Avoid using mixed ways of accessing max value as this affects consistency.

- [ReaperVaultV2.sol#L580](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L588)

- [ActivePool.sol#L258](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L258)

## Inconsistent spacing in comments

Some lines use `// x` and some use `//x`. Following the common style comment should be as follows:

```diff
-uint256 public genericDeclaration; //generic comment without space`
+uint256 public genericDeclaration; // generic comment with space`
```

- [TroveManager.sol#L1413](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1413)
        //assert(newBaseRate <= DECIMAL_PRECISION); // This is already enforced in the line above

## Non-assembly method available

Non assembly methods are available for this values

- `assembly{ id := chainid() }` => `uint256 id = block.chainid`,

- [LUSDToken.sol#L300](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L300)
            chainID := chainid()


## Wrong value of days slightly affects precision

Formula uses a hardcoded value of `365` (days) which would be wrong applied in a leap year (`366` days).

- [CommunityIssuance.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L25)
    * Minutes in one year: 60*24*365 = 525600
- [ReaperBaseStrategyv4.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L25)
    uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;

### Recommendation

- Consider that that each year is `365.2422 days` for maximum precision[(NASA value)](https://pumas.nasa.gov/sites/default/files/examples/04_21_97_1.pdf)

## Missing indexed event parameters

`NOTE`: None of these findings where found by [4naly3er output - NC](https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae)

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

- [TroveManager.sol#L210-L216](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L210-L216)

- [ReaperVaultV2.sol#L90-L100](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L90-L100)

### Recommendation

Consider which event parameters could be particularly useful to off-chain tools and should be indexed.

## Different versions of pragma

Some of the contracts include an unlocked pragma, e.g., `pragma solidity >=0.8.0`. 

Locking the pragma helps ensure that contracts are not accidentally deployed using an old compiler version with unfixed bugs.

- [ReaperVaultV2.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L3)
pragma solidity ^0.8.0;
- [ReaperVaultERC4626.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3)
pragma solidity ^0.8.0;
- [ReaperBaseStrategyv4.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3)
pragma solidity ^0.8.0;
- [ReaperStrategyGranarySupplyOnly.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3)
pragma solidity ^0.8.0;

### Recommendation

Lock pragmas to a specific Solidity version. 

## Use of `uint` and `uint256` for the same type affects consistency

All over the code `uint` and `uint256`, what are the same type, are used with no consistency even in the same [contract](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L146-L158).

Consider using only `uint` or `uint256` for clarity and consistency.

## Missing error messages in require statements

`NOTE`: None of these findings where found by [4naly3er output - NC](https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae)

`require`/`revert` statements should include error messages in order to help at monitoring the system.

- [ReaperStrategyGranarySupplyOnly.sol#L167](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167)
            require(step[0] != address(0));
- [ReaperStrategyGranarySupplyOnly.sol#L168](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L168)
            require(step[1] != address(0));

## Mixed styles as `mapping(` / `mapping (`, `for (` / `for(` are used without consistency, affecting searchability

All over the code there is no consistency when writting spacings and `(`. Not only as mentioned in the title, but `( variable` and `(variable` are used, etc. This really affects how easy is to maintain the code as it is hard to search at it. 

Better to follow Solidity guidelines as it would be following `for(variable` rather than `for ( variable` or other mixes of style.

As example:

- [LQTYStaking.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25)
`mapping( address => uint) public stakes;`

- [StabilityPool.sol#L223](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L223)
`mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;`

## Deprecated use of `now` rather than `block.timestamp`

- [LUSDToken.sol#L282](https://github.com/nicobevilacqua/2023-02-ethos/blob/2c41b21adf5db192d31eeda37c60169853b10fc0/Ethos-Core/contracts/LUSDToken.sol#L282)

## Signature malleability of EVM's `ecrecover()`

The function calls the Solidity `ecrecover()` function directly to verify the given signatures. However, the `ecrecover()` EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin’s ECDSA library.

- [LUSDToken.sol#L287](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L287)
        address recoveredAddress = ecrecover(digest, v, r, s);

### Recommendation

Use the `ecrecover` function from OpenZeppelin’s ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

## Missing idempotent check

- [ReaperVaultV2.sol#L580](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L580)

Add a not same value check like this:

`if (_newCoolAddress == newCoolAddress) revert ADDRESS_SAME();`

## Use of `_renounceOwnership` can lead to unexpected behaviors

Renouncing to the ownership of a contract can lead to unexpected behaviors. Method `setAddresses` can only be called once therefore, due to the call from the method to `_renounceOwnership`. Consider using a method to submit a proposal of new addresses, then another one to accept the proposal where `_renounceOwnership` is called at the end.

- [BorrowerOperations.sol#L167](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L167)
        _renounceOwnership();
- [StabilityPool.sol#L302](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L302)
        _renounceOwnership();
- [LQTYStaking.sol#L100](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L100)
        _renounceOwnership();

- [TroveManager.sol#L294](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L294)
        owner = address(0);    

## Names can be more specific

- [StabilityPool.sol#L180](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L180)
- [StabilityPool.sol#L181](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L181)
- [StabilityPool.sol#L195](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L195)

## Unused dependency can be removed

// import "./Dependencies/Ownable.sol";
- [TroveManager.sol#L14](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L14)

## Strategy is not checked to be valid and therefore may not exist

strategy is not validated (could not exist) and this function would return 0 on that case.

- [ReaperVaultV2.sol#L226](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L226)


## Maximum line length exceeded

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Solidity newer guidelines suggest 120 characters. 
Reference: Long lines should be wrapped to conform with Solidity Style [guidelines](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#maximum-line-length)

Following lines with more than 120:
 

- [BorrowerOperations.sol#L105](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L105)
- [BorrowerOperations.sol#L172](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L172)
- [BorrowerOperations.sol#L190](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L190)
- [BorrowerOperations.sol#L233](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L233)
- [BorrowerOperations.sol#L235](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L235)
- [BorrowerOperations.sol#L237](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L237)
- [BorrowerOperations.sol#L248](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L248)
- [BorrowerOperations.sol#L254](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L254)
- [BorrowerOperations.sol#L259](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L259)
- [BorrowerOperations.sol#L264](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L264)
- [BorrowerOperations.sol#L268](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L268)
- [BorrowerOperations.sol#L272](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L272)
- [BorrowerOperations.sol#L276](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L276)
- [BorrowerOperations.sol#L282](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L282)
- [BorrowerOperations.sol#L300](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L300)
- [BorrowerOperations.sol#L312](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L312)
- [BorrowerOperations.sol#L343](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L343)
- [BorrowerOperations.sol#L358](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L358)
- [BorrowerOperations.sol#L419](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L419)
- [BorrowerOperations.sol#L510](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L510)
- [BorrowerOperations.sol#L511](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L511)
- [BorrowerOperations.sol#L517](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L517)
- [BorrowerOperations.sol#L528](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L528)
- [BorrowerOperations.sol#L529](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L529)
- [BorrowerOperations.sol#L530](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L530)
- [BorrowerOperations.sol#L538](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L538)
- [BorrowerOperations.sol#L546](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L546)
- [BorrowerOperations.sol#L588](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L588)
- [BorrowerOperations.sol#L637](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L637)
- [BorrowerOperations.sol#L644](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L644)
- [BorrowerOperations.sol#L645](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L645)
- [BorrowerOperations.sol#L675](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L675)
- [BorrowerOperations.sol#L697](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L697)
- [TroveManager.sol#L58](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L58)
- [TroveManager.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L93)
- [TroveManager.sol#L97](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L97)
- [TroveManager.sol#L102](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L102)
- [TroveManager.sol#L114](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L114)
- [TroveManager.sol#L200](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L200)
- [TroveManager.sol#L201](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L201)
- [TroveManager.sol#L202](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L202)
- [TroveManager.sol#L338](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L338)
- [TroveManager.sol#L348](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L348)
- [TroveManager.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L351)
- [TroveManager.sol#L384](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L384)
- [TroveManager.sol#L393](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L393)
- [TroveManager.sol#L398](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L398)
- [TroveManager.sol#L404](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L404)
- [TroveManager.sol#L407](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L407)
- [TroveManager.sol#L416](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L416)
- [TroveManager.sol#L421](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L421)
- [TroveManager.sol#L428](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L428)
- [TroveManager.sol#L571](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L571)
- [TroveManager.sol#L572](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L572)
- [TroveManager.sol#L575](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L575)
- [TroveManager.sol#L618](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L618)
- [TroveManager.sol#L774](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L774)
- [TroveManager.sol#L775](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L775)
- [TroveManager.sol#L778](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L778)
- [TroveManager.sol#L819](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L819)
- [TroveManager.sol#L854](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L854)
- [TroveManager.sol#L887](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L887)
- [TroveManager.sol#L898](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L898)
- [TroveManager.sol#L902](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L902)
- [TroveManager.sol#L903](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L903)
- [TroveManager.sol#L908](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L908)
- [TroveManager.sol#L909](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L909)
- [TroveManager.sol#L915](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L915)
- [TroveManager.sol#L926](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L926)
- [TroveManager.sol#L934](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L934)
- [TroveManager.sol#L935](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L935)
- [TroveManager.sol#L937](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L937)
- [TroveManager.sol#L957](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L957)
- [TroveManager.sol#L995](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L995)
- [TroveManager.sol#L998](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L998)
- [TroveManager.sol#L1001](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1001)
- [TroveManager.sol#L1002](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1002)
- [TroveManager.sol#L1003](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1003)
- [TroveManager.sol#L1006](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1006)
- [TroveManager.sol#L1007](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1007)
- [TroveManager.sol#L1008](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1008)
- [TroveManager.sol#L1011](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1011)
- [TroveManager.sol#L1012](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1012)
- [TroveManager.sol#L1013](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1013)
- [TroveManager.sol#L1044](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1044)
- [TroveManager.sol#L1053](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1053)
- [TroveManager.sol#L1082](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1082)
- [TroveManager.sol#L1097](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1097)
- [TroveManager.sol#L1212](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1212)
- [TroveManager.sol#L1221](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1221)
- [TroveManager.sol#L1258](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1258)
- [TroveManager.sol#L1259](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1259)
- [TroveManager.sol#L1296](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1296)
- [TroveManager.sol#L1303](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1303)
- [TroveManager.sol#L1305](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1305)
- [TroveManager.sol#L1312](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1312)
- [TroveManager.sol#L1323](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1323)
- [TroveManager.sol#L1336](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1336)
- [TroveManager.sol#L1372](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1372)
- [TroveManager.sol#L1567](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1567)
- [TroveManager.sol#L1574](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1574)
- [TroveManager.sol#L1581](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1581)
- [TroveManager.sol#L1588](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1588)
- [ActivePool.sol#L44](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L44)
- [ActivePool.sol#L47](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L47)
- [ActivePool.sol#L144](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L144)
- [ActivePool.sol#L258](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L258)
- [ActivePool.sol#L264](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L264)
- [ActivePool.sol#L269](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L269)
- [ActivePool.sol#L282](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L282)
- [ActivePool.sol#L287](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L287)
- [ActivePool.sol#L288](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L288)
- [StabilityPool.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L25)
- [StabilityPool.sol#L27](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L27)
- [StabilityPool.sol#L28](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L28)
- [StabilityPool.sol#L31](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L31)
- [StabilityPool.sol#L34](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L34)
- [StabilityPool.sol#L42](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L42)
- [StabilityPool.sol#L45](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L45)
- [StabilityPool.sol#L46](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L46)
- [StabilityPool.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L50)
- [StabilityPool.sol#L52](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L52)
- [StabilityPool.sol#L55](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L55)
- [StabilityPool.sol#L58](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L58)
- [StabilityPool.sol#L59](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L59)
- [StabilityPool.sol#L65](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L65)
- [StabilityPool.sol#L68](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L68)
- [StabilityPool.sol#L69](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L69)
- [StabilityPool.sol#L71](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L71)
- [StabilityPool.sol#L74](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L74)
- [StabilityPool.sol#L75](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L75)
- [StabilityPool.sol#L80](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L80)
- [StabilityPool.sol#L83](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L83)
- [StabilityPool.sol#L89](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L89)
- [StabilityPool.sol#L92](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L92)
- [StabilityPool.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L93)
- [StabilityPool.sol#L94](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L94)
- [StabilityPool.sol#L99](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L99)
- [StabilityPool.sol#L101](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L101)
- [StabilityPool.sol#L103](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L103)
- [StabilityPool.sol#L109](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L109)
- [StabilityPool.sol#L130](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L130)
- [StabilityPool.sol#L136](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L136)
- [StabilityPool.sol#L142](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L142)
- [StabilityPool.sol#L189](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L189)
- [StabilityPool.sol#L205](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L205)
- [StabilityPool.sol#L217](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L217)
- [StabilityPool.sol#L220](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L220)
- [StabilityPool.sol#L319](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L319)
- [StabilityPool.sol#L361](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L361)
- [StabilityPool.sol#L434](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L434)
- [StabilityPool.sol#L474](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L474)
- [StabilityPool.sol#L547](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L547)
- [StabilityPool.sol#L553](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L553)
- [StabilityPool.sol#L626](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L626)
- [StabilityPool.sol#L637](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L637)
- [StabilityPool.sol#L657](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L657)
- [StabilityPool.sol#L659](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L659)
- [StabilityPool.sol#L670](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L670)
- [StabilityPool.sol#L671](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L671)
- [StabilityPool.sol#L673](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L673)
- [StabilityPool.sol#L762](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L762)
- [StabilityPool.sol#L763](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L763)
- [LQTYStaking.sol#L203](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L203)
- [LUSDToken.sol#L16](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L16)
- [LUSDToken.sol#L22](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L22)
- [LUSDToken.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L25)
- [LUSDToken.sol#L46](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L46)
- [LUSDToken.sol#L238](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L238)
- [LUSDToken.sol#L248](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L248)
- [ReaperVaultERC4626.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L50)
- [ReaperVaultERC4626.sol#L65](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L65)
- [ReaperVaultERC4626.sol#L71](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L71)
- [ReaperVaultERC4626.sol#L72](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L72)
- [ReaperVaultERC4626.sol#L76](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L76)
- [ReaperVaultERC4626.sol#L85](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L85)
- [ReaperVaultERC4626.sol#L86](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L86)
- [ReaperVaultERC4626.sol#L89](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L89)
- [ReaperVaultERC4626.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L93)
- [ReaperVaultERC4626.sol#L94](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L94)
- [ReaperVaultERC4626.sol#L103](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L103)
- [ReaperVaultERC4626.sol#L119](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L119)
- [ReaperVaultERC4626.sol#L128](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L128)
- [ReaperVaultERC4626.sol#L130](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L130)
- [ReaperVaultERC4626.sol#L136](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L136)
- [ReaperVaultERC4626.sol#L159](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L159)
- [ReaperVaultERC4626.sol#L160](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L160)
- [ReaperVaultERC4626.sol#L163](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L163)
- [ReaperVaultERC4626.sol#L180](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L180)
- [ReaperVaultERC4626.sol#L182](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L182)
- [ReaperVaultERC4626.sol#L207](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L207)
- [ReaperVaultERC4626.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L214)
