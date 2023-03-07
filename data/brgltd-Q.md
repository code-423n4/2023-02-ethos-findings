# [01] Calling `ReaperVaultV2.setWithdrawalQueue()` passing a `_withdrawalQueue` array with less items than `strategies` can accidentaly make strategy items useless.

## Proof of Concept

`ReaperVaultV2.addStrategy()` will add a new strategy and populate the `withdrawalQueue` array and the `strategies` mapping. 

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L158-L166

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L169

If the admin calls `ReaperVaultV2.setWithdrawalQueue()` with a `withdrawalQueue` containing less items than `strategies`, the "extra" strategies will be ignored while iterating the `withdrawalQueue` in `ReaperVaultV2._withdraw()`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L372-L397

The likelyhood is small (since it depends on the admin inputting bad values), but the impact can be considerable, since the some strategies won't be accounted while making withdrawals.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L385

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95-L103

## Impact

The strategy allocated value will be used to transfer funds from the strategy into the vault. These funds will be missing if the number of `withdrawalQueue` items is smaller than the number of items in `strategies`.

## Recommended Mitigation Steps

Consider adding a check on `ReaperVaultV2.setWithdrawalQueue()` to ensure the length of `_withdrawalQueue` matches the length of the existing `strategies`. This can be done by checking the length of the argument `_withdrawalQueue` with the current `withdrawalQueue`, since the current `strategies` mapping will have the same length as the current `withdrawalQueue` array.

```diff
diff --git a/ReaperVaultV2.sol.orig b/ReaperVaultV2.sol
index 87de400..3eb078e 100644
--- a/ReaperVaultV2.sol.orig
+++ b/ReaperVaultV2.sol
@@ -259,6 +259,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
         _atLeastRole(ADMIN);
         uint256 queueLength = _withdrawalQueue.length;
         require(queueLength != 0, "Queue must not be empty");
+        require(_withdrawalQueue.length == withdrawalQueue.length, "Invalid length");

         delete withdrawalQueue;
         for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
```

Alternatively, if setting a new `strategies` array with less items than the current `strategies` array is an expected functionality of the system, consider documenting this behavior in `ReaperVaultV2.setWithdrawalQueue()`.

# [02] `ReaperVaultV2.revokeStrategy()` won't remove the allocated value

## Impact

`ReaperVaultV2.revokeStrategy()` will remove the vault assets that should be allocated to the strategy (in BPS), but it won't remove the actual amount currently allocated.

The allocated value will still be used for withdrawals and for computing the capital available.

## Proof of Concept

`ReaperVaultV2.revokeStrategy()` sets `strategy.allocBPS` and `totalAlloBPS` to zero, but it doesn't change `strategy.allocated` or `totalAllocated`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L205-L217

`ReaperVaultV2._withdraw()` will use the allocated value for the strategie(s).

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L379-L385

The allocated value will also be used for retrieving the available capital, and for strategy reporting.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L225-L252

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L493-L560

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L432-L453

## Recommended Mitigation Steps

Update `strategies[_strategy].allocBPS` and `totalAllocated` when revoking a strategy.

Alternatively, if this behavior is intentional (not resetting `allocBPS` and decreasing `totalAllocated` when revoking a strategy), consider adding a comment on `ReaperVaultV2.revokeStrategy()`. The comment on [L202](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L202) indicates that `ReaperVaultV2.revokeStrategy()` will remove any allocation.

# [03] Incorrect bookkeeping for inflationary/deflationary/fee-on-trasnfer tokens

The function `ActivePool.sendCollateral()` will use `_amount` to make the transfer and to update the state variable `collAmount[_collateral]`. 

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L175

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L186

Some tokens might charge a fee on transfer or implement a inflationary/deflationary behavior. If a token with this behavior is used as collateral, the `_amount` transferred might not match the `_amount` being decreased on `collAmount[_collateral]`.

## Recommendation

Update the variable based on actual balance chages, e.g. calculate the real amount that was transferred before saving into the state variable. Note: this could break checks-effects-interactions, so it's also recommended to add a nonReentrant modifier on `ActivePool.sendCollateral()`.

# [04] Replace safeApprove with safeIncreaseAllowance

`ReaperVaultV2` is using OpenZeppelin for the ERC20 construction. OpenZeppelin current recommendation is to avoid safeApprove and replace with increaseAllowance. See [reference](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d00acef4059807535af0bd0dd0ddf619747a044b/contracts/token/ERC20/utils/SafeERC20.sol#L39-L45).

The risk related to safeApprove is related to an attacker being able to use the old and new approval. See OpenZeppelin [comment](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d00acef4059807535af0bd0dd0ddf619747a044b/contracts/token/ERC20/IERC20.sol#L57-L62). This doesn't seen to a risk currently on `ReaperVaultV2`, since it's going from 0 to max uint256. However, using safeIncreaseAllowance is still recommended.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74

# [05] `ReaperVaultV2.setWithdrawalQueue()` allows for duplicated strategies

`ReaperVaultV2.setWithdrawalQueue()` won't check if two or more strategies have the same address. This could cause a strategy debt to decrease more than expected.

Consider adding a check to prevent duplicated strategies.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L258-L271

# [06] Lack of checks-effects-interactions

Consider making external calls after state changes to follow the checks-effects-interactions pattern.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L525-L529

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L540-L541

# [07] Permit will accept owner with address zero

The return of `ecrecover` is not being checked in `LUSDToken.permit()`. If digest + vrs doesn't match, recoveredAddress will be zero. Thus, passing owner of address zero will not revert the Tx. Also, the comment on [L275](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L275) is unclear. Consider checking against address zero for the returned value of ecrecover and clarifying the comment on L275.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L262-L290

# [08] Usage of regular transfer|transferFrom can cause issues for non-standard tokens

Prefer the usage of the safe variants, since the non-safe version can cause issues regarding tokens that don't return a boolean (USDT, BNB) or tokens that don't revert on failure.

The only instances where the project is using the non-safe function is for lusd and oauth transfer, so it doesn't seem to be any direct issue. However, using the safe variant is still recommended and it will be more consistent throughout the codebase.

Consider converting from transfer|transferFrom to safeTransfer|safeTransferFrom on the following four instances.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L135

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L171

# [09] Avoid pragma float

All the contracts in Ethos-Vault are floating the pragma.

Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or a package.

# [10] Outdated OpenZeppelin version

Liquidy contracts that are based of OpenZeppelin contracts were using v3 of OpenZeppelin as an implementation reference. Consider updating/reviewing to the latest version of OpenZeppelin (v4) where possible possible, to ensure the contracts contain the latest security fixes that OpenZeppelin applied from v3 to v4.

# [11] Missing address zero checks on `ReaperVaultV2` constructor for `_treasury`, `_strategies` and `multisigRoles`

If one of these variable is configured with address zero, failure to immediately reset the value can result in unexpected behavior for the project.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136

# [12] Max allowance

The vault has unlimited approval to spend the strategy contract funds. If possible, consider allowing only necessary and limited amounts, to prevent spending/drailing all the strategy funds in one Tx.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74

# [13] Missing storage gap for upgradeable contracts

Consider adding a `__gap[]` storage variable to allow for new storage variables in later versions.

See this [link](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) for a description of this storage variable issue. Please, refer to this [example](https://github.com/code-423n4/2022-08-foundation/blob/main/contracts/mixins/shared/Gap10000.sol) for an implementation reference.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L13-L14

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L11-L12

# [14] Emit events before external calls

Multiple functions in the project emit an event as the last statement. Wherever possible, consider emitting events before external calls. In case of reentrancy, funds are not at risk (for external call + event ordering), however emitting events after external calls can damage frontends and monitoring tools in case of reentrancy attacks.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L319-L338

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71-L123

# [15] Missing check for multisigRoles in `ReaperVaultV2.sol`

`ReaperVaultV2.sol` will initialize the multisigRoles in the constructor and `ReaperBaseStrategyv4.sol` will initialize on `__ReaperBaseStrategy_init()`.

`ReaperBaseStrategyv4.sol` contains the following check `require(_multisigRoles.length == 3, "");`. This check is missing on the `ReaperVaultV2.sol` constructor. Consider adding the same check on `ReaperVaultV2.sol`, to validate the length of multisigRoles.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L81

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136

# [16] Critical changes should use two-step procedure

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

Consider adding a two-steps pattern on critical changes to avoid mistakenly changing the system state.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L603-L611

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L156-L160

# [17] Add a validation to check if steps tokens are not the same

`ReaperStrategyGranarySupplyOnly.setHarvestSteps()` will have the `steps` containing two tokes. Currently it validates that the addresses are not address zero. One improvement that can be made is to add a check that the tokens are not the same. E.g. add `require(step[0] != step[1])`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L165-L170

# [18] Missing events for parameter changes

Events will improve offchain monitoring and logging of the system changes.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L160-L171

# [19] Avoid stale values on setter functions

Consider validating that the received value is different than the current value to prevent triggering unecessary events.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L603-L611

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L617-L622

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L627-L631

# [20] Usage of return named variables and explicit values

Some functions return named variables, others return explicit values.

Following function returns an explicit value.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L418-L425

Following function return returns a named variable.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L319-L338

Consider adopting the same approach throughout the codebase to improve the explicitness and readability of the code.

# [21] Missing NATSPEC

Consider adding NATSPEC for function arguments and returned values.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L90-L103

Also, consider adding NATSPEC on all external/public functions.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L226-L236

# [22] Misleading comment

The number of decimals will depend on the token, e.g. wbtc will have 8 decimals. The comment on [L293](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L293) indicates it's always 18 decimals.

# [23] Order of functions

The solidity [documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) recommends the following order for functions:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private

The instances bellow show a public function between two external functions. If possible, consider updating the functions ordering to meet the solidity styleguide.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L205

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L225

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L258

# [24] Add a limit for the maximum number of characters per line

The solidity [documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) recommends a maximum of 120 characters.

Consider adding a limit of 120 characters or less to prevent large lines.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L238

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L248

# [25] Missing unit tests

It is crucial to write tests with the target of 100% coverage for smart contracts.

The following functions are not covered:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L178-L184

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L258-L271

It is recommended to write tests for all possible code flows/functions.

# [26] Interchangeable usage of uint and unit256

Consider using only approach, e.g. using only uint256.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L67-L77

# [27] Replace assert with require or custom error

As described on the solidity [documentation](https://docs.soliditylang.org/en/v0.8.15/control-structures.html#panic-via-assert-and-error-via-require). "The assert function creates an error of type Panic(uint256). … Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix.

Even if the code is expected to be unreacheable, consider using a require statement/custom error instead of assert.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L331

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312-L313

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L321

# [28] Repeated validation statements can be converted into a function modifier

For example, `require(!emergencyShutdown)` can be converted into a modifier `notEmergencyShutdown` to be used on the functions header.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L150

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L321

Similar pattern can be implemented for `require(_feeBPS <= PERCENT_DIVISOR / 5, "")` in `ReaperVaultV2.sol`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L181

# [29] Reduntant check

`VeloSolidMixin._swapVelo()` is already checking for amount zero. This check can be removed in `ReaperStrategyGranarySupplyOnly._harvestCore()`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L121-L123

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/mixins/VeloSolidMixin.sol#L24-L26

# [30] Can use a boolean flag instead of postponing the update for 100 years

The current logic to clear an update will delay 100 years. An alternative could be to use a boolean flag to enable and disable upgrades instead of postponing for 100 years. This could facilitate frontends/composability, e.g. show that updates are avaiable or not instead of showing avaiable in 2123.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L25

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L179-L182

# [31] Non-assembly method available

There are some automated tools that will flag a project as having higher complexity if there is inline-assembly, so it’s best to avoid using it where it’s not necessary

`assembly{ id := chainid() } => uint256 id = block.chainid;`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L298-L302

# [32] `ReaperVaultV2._withdraw()` can use the same argument for `_receiver` and `_owner`

`ReaperVaultV2.withdrawAll()` and `ReaperVaultV2.withdraw()` will pass `msg.sender` for `_receiver` and `_owner` of `ReaperVaultV2._withdraw()`. It would be possible to compose `_receiver` and `_owner` into the same argument for `ReaperVaultV2._withdraw()`.

# [33] Use scientific notation rather than exponentiation

Scientific notation can be used for better code readability, e.g. use 10e18 instead of `10**18`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125

# [34] Large multiples of ten should use scientific notation.

Using scientific notation for multiples of ten will improve code readability.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L41

# [35] Use memory instead of storage where possible

Memory can be used instead of storage for instantiating the step from the steps array in `ReaperStrategyGranarySupplyOnly_harvestCore()`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L118

# [36] Constant redefined elsewhere

Consider defining in only one contract so that values cannot become out of sync when only one location is updated.

A cheap way to store constants in a single location is to create an internal constant in a library. If the variable is a local cache of another contract’s value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers don’t get out of sync.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74-L76

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50-L52

# [37] Validation statements for function arguments should be place at the top of the function

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L81

# [38] Add BPS at the end of the variable name when the value is represented in BPS

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L52-L54

# [39] Named imports can be used

It's possible to name the imports to improve code readability. E.g. `import @openzeppelin/contracts/token/ERC20/ERC20.sol` can be rewritten as `import {ERC20} from @openzeppelin/contracts/token/ERC20/ERC20.sol`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L5-L14

# [40] Replace magic numbers with constants

Adding constants can improve code readability. E.g. for the instance bellow, 46 and 10**6 could be two constants.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125

# [41] OpenZeppelin new recommendation is to use `_disableInitializers()`

Instead of using an empty constructor with the `initializer` modifier, OpenZeppelin recommendation is to use `_disableInitializers()` inside the constructor.

See this [post](https://forum.openzeppelin.com/t/what-does-disableinitializers-function-mean/28730) for more information.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L61

# [42] Variable shadowing for lastReport and StrategyParams.lastReport in `ReaperVaultV2`

Consider renaming `lastReport` or `StrategyParams.lastReport` to avoid variable shadowing.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L32

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L46

# [43] Initializing a variable with the default value

For `ReaperBaseStrategyv4.withdraw()`, the variable amountFree doesn't need to be initialized to zero since uint256 will already have the default value of zero and the value will be overwriteen in `ReaperStrategyGranarySupplyOnly._liquidatePosition()`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L100-L101

# [44] Can use ternary

The [conditional](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L331-L335) on `ReaperVaultV2._deposit()` can be replace with:

```solidity
shares = totalSupply() == 0
    ? _amount
    : (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool";
```

# [45] Statements can be simplified

The [Math.min](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L244-L246) logic for `ReaperVaultV2.availableCapital()` can be simplified to the following:

```solidity
uint256 available = Math.min(
    Math.min(
        stratMaxAllocation - stratCurrentAllocation,
        vaultMaxAllocation - vaultCurrentAllocation
    );
    token.balanceOf(address(this))
)
```

# [46] Variable is not needed

The following instance:

```solidity
uint256 allocation = stratParams.allocated;
require(loss <= allocation, "Strategy loss cannot be greater than allocation");
```

Can be replaced with

```solidity
require(loss <= stratParams.allocated, "Strategy loss cannot be greater than allocation");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L435-L436

# [47] `ReaperBaseStrategyv4.sol` can be renamed to `ReaperBaseStrategyV4.sol`

`ReaperBaseStrategyV4.sol` will be more consistent with `ReaperVaultV2.sol`.
