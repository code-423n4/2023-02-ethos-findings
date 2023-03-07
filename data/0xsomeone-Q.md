## Ethos Reserve Q&A Report

This submission serves as a Q&A report of the Ethos Reserve codebase. The findings that were identified with a low / non-critical security impact are as follows:

## `CommunityIssuance.sol`

### QIE-01L: Inexistent Evaluation of Reward Per Second (Affected Lines: [L112](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L112))

An abnormally large `distributionPeroid` coupled with a low amount of funds in a `CommunityIssuance::fund` call will result in no rewards being distributed per second, causing fund loss that at most equates to `distributionPeriod` in the token's denomination.

We advise any unaccounted-for reward to not be transferred from the `msg.sender` to the contract by calculating the `OathToken` to transfer as `rewardPerSecond.mul(distributionPeriod)`.

### QIE-02L: Inexistent Sanitization of Distribution Period (Affected Lines: [L120-L122](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120-L122))

The distribution period of the contract is not sanitized during a `CommunityIssuance::updateDistributionPeriod` call.

We advise a non-zero validation to be performed as otherwise the contract's `CommunityIssuance::fund` function will be inoperable until the misconfiguration is corrected.

## `CollateralConfig.sol`

### CCG-01L: Inexistent Validation of System Conditions (Affected Lines: [L66-L70](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L66-L70))

The `CCR` configured for a particular market should at all times be greater than the `MCR` as otherwise, the market will never achieve its critical-state, preventing the system's failsafe mechanisms from kicking in.

We advise the `_CCRs[i]` value to be mandated as greater-than-or-equal-to the `_MCRs[i]` value of a particular collateral. Depending on how this finding is viewed, it can be considered medium-risk as a misconfigured asset will never trigger the failsafe mechanisms of the protocol and thus never recover from a critical state.

## `ReaperBaseStrategyv4.sol`

### RBS-01L: Incorrect Usage of Deprecated Method (Affected Lines: [L74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74))

The referenced line invokes `SafeERC20::safeApprove` which can cause upgrades to fail as the `ReaperBaseStrategyv4` contract represents an upgradeable contract. 

If the contract's logic is adjusted and attempted to be upgraded, a second `SafeERC20::safeApprove` invocation that will be performed during the contract's re-initialization will fail due to [the custom and deprecated logic present in the function](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol#L50).

We advise a direct `approve` instruction to be utilized instead, potentially by evaluating the yielded `bool` opportunistically similarly to how `SafeERC20` achieves this.

### RBS-02L: Unsafe Value Negations (Affected Lines: [L114](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L114), [L121](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L121), [L128](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L128))

The referenced negation operations (`-a`) are performed unsafely as they do not apply any bound checks to the `a` value. 

As all linked negations are meant to convert a value from a negative to a positive value, an edge case manifests whereby the minimum `int256` (`type(int256).min`) will overflow when negated by one unit as the negative value ranges have one more member in signed integer types.

We advise the code to use a wrapper library that negates the value and applies a bound check to ensure that the negation will never result in an underflow. 

## `ReaperStrategyGranarySupplyOnly.sol`

### RGO-01L: Unsafe Casting Operations (Affected Lines: [L134](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L134), [L141](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L141))

The referenced casting operations are performed unsafely, potentially resulting in an uncaught casting overflow to occur.

We advise safe wrapper libraries to be utilized, such as OpenZeppelin's [`SafeCast`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol) library. To note, Solidity's built-in safe arithmetic post-0.8 **does not cover casting overflows**.

### RGO-02L: Unsafe Value Negation (Affected Lines: [L136](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L136))

The referenced negation operations (`-a`) are performed unsafely as they do not apply any bound checks to the `a` value. 

As all linked negations are meant to convert a value from a negative to a positive value, an edge case manifests whereby the minimum `int256` (`type(int256).min`) will overflow when negated by one unit as the negative value ranges have one more member in signed integer types.

We advise the code to use a wrapper library that negates the value and applies a bound check to ensure that the negation will never result in an underflow. 

## `ReaperVaultV2.sol`

### RVT-01L: Inconsistent Input Sanitization (Affected Lines: [L133-L135](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L133-L135))

The referenced lines of code utilize the `_multisigRoles` members without validating the array's length.

We advise sanitization to be imposed similarly to [`ReaperBaseStrategyv4::__ReaperBaseStrategy_init`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L81) to prevent improper role initializations.

### RVT-02L: Unsafe Casting Operations (Affected Lines: [L228](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L228), [L235](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L235), [L248](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L248))

The referenced casting operations are performed unsafely, potentially resulting in an uncaught casting overflow to occur.

We advise safe wrapper libraries to be utilized, such as OpenZeppelin's [`SafeCast`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol) library. To note, Solidity's built-in safe arithmetic post-0.8 **does not cover casting overflows**.

### RVT-03L: Inexistent Validation of Duplicate Entries (Affected Lines: [L258-L271](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L258-L271))

The `ReaperVaultV2::setWithdrawalQueue` function does not evaluate whether duplicate entries exist in the queue.

While the finding will have no impact as it will only result in a redundant increase in gas within `ReaperVaultV2::_withdraw`, we still advise duplicate entries to be prohibited.

### RVT-04L: Inexistent Sanitization of Withdrawal Queue (Affected Lines: [L258-L271](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L258-L271))

The active withdrawal queue of the system should at all times represent the strategies from which funds can be withdrawn in the system.

We advise the `length` of the queue to be mandated as exactly equal to the number of active strategies, ensuring that an incorrect withdrawal queue cannot be specified.

### RVT-05L: Unsafe Value Negations (Affected Lines: [L500](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L500), [L510](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L510))

The referenced negation operations (`-a`) are performed unsafely as they do not apply any bound checks to the `a` value. 

As all linked negations are meant to convert a value from a negative to a positive value, an edge case manifests whereby the minimum `int256` (`type(int256).min`) will overflow when negated by one unit as the negative value ranges have one more member in signed integer types.

We advise the code to use a wrapper library that negates the value and applies a bound check to ensure that the negation will never result in an underflow. 

## `ReaperVaultERC4626.sol`

### RVE-01L: Misleading Function Naming (Affected Lines: [L202-L210](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L202-L210))

The `ReaperVaultERC4626::withdraw` function meant to conform to the EIP standard accepts the input `uint256` argument denominated in `assets`, however, the `ReaperVaultV2::withdraw` functions accept the input `uint256` argument denominated in `shares`.

We advise the codebase to become consistent, utilizing a different naming convention for `ReaperVaultV2` (i.e. `deposit`).

## `ActivePool.sol`

### APL-01L: Unsafe Casting Operations (Affected Lines: [L277](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L277))

The referenced casting operations are performed unsafely, potentially resulting in an uncaught casting overflow to occur.

We advise safe wrapper libraries to be utilized, such as OpenZeppelin's [`SafeCast`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol) library. To note, Solidity's built-in safe arithmetic post-0.8 **does not cover casting overflows**.

### APL-02L: Unsafe Value Negations (Affected Lines: [L282](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L282))

The referenced negation operations (`-a`) are performed unsafely as they do not apply any bound checks to the `a` value. 

As all linked negations are meant to convert a value from a negative to a positive value, an edge case manifests whereby the minimum `int256` (`type(int256).min`) will overflow when negated by one unit as the negative value ranges have one more member in signed integer types.

We advise the code to use a wrapper library that negates the value and applies a bound check to ensure that the negation will never result in an underflow. 

### APL-03L: Misleading Error Message (Affected Lines: [L334](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L334))

The referenced error message of the `require` check within `ActivePool::_requireCallerIsBOorTroveMorSP` contains a misleading error message which does not list the `redemptionHelper`.

We advise this message to be corrected, representing the actual condition evaluated by the `require` check.

## `BorrowerOperations.sol`

### BOS-01L: Incorrect Relayed Borrower (Affected Lines: [L365](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L365))

The `BorrowerOperations::_adjustTrove` function will relay an incorrect "borrower" to its internal `_moveTokensAndCollateralfromAdjustment` call when `BorrowerOperations::moveCollateralGainToTrove` is invoked by the stability pool.

While it is inconsequential in the current code (no LUSD is withdrawn / repaid and the transaction represents a `_isCollIncrease`), it is still advisable to relay the correct argument via a ternary (`a ? b : c`) conditional.

## `StabilityPool.sol`

### SPL-01L: Unconditional Loss Calculation (Affected Lines: [L339](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L339), [L384](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L384))

The referenced calculations meant to assess the LUSD loss incurred by the deposit are unconditionally executed, potentially causing an underflow if the position somehow increases.

We evaluated the codebase and the linked calculation can never underflow in the current implementation, however, the code should evaluate whether any loss exists prior to calculating it via a conditional, prohibiting an underflow from halting deposits / withdrawals.

## `LUSDToken.sol`

### LTN-01L: Permittence of Invalid Signatures (Affected Lines: [L287-L288](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287-L288))

The `ecrecover` mechanism within `LUSDToken::permit` allows an invalid signature to result in the zero address incorrectly.

While an approval will still be prevented by how the `LUSDToken::_approve` function disallows the `from` address from being zero, we still advise invalid signatures to emit a proper error message rather than failing with an "approval from zero" error message.

### LTN-02L: Bypass of Valid Recipient Check (Affected Lines: [L188](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L188), [L203](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L203))

The `LUSDToken::returnFromPool` as well as the `LUSDToken::mint` functions represent instances whereby a non-zero LUSD balance can be transmitted to a target recipient without the `LUSDToken::_requireValidRecipient` check.

We advise both instances to properly apply the check, preventing LUSD tokens from being minted to incorrect addresses. The present use of the codebase disallows this as no code path will cause the check to fail, however, the check should be present regardless.