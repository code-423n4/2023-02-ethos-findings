# QA Report
## Finding Summary

[**Low Severity**](#Low-Severity)
1. [**Unchecked Cast May Overflow**](#1-Unchecked-Cast-May-Overflow)
2. [**Consider a More Recent Solidity Version**](#2-Consider-a-More-Recent-Solidity-Version)
3. [**Code Function Does Not Match Comment**](#3-Code-Function-Does-Not-Match-Comment)
4. [**Low Coverage**](#4-Low-Coverage)
5. [**assert Used Over require**](#5-assert-Used-Over-require)
6. [**Drastic Compiler Version Variation**](#6-Drastic-Compiler-Version-Variation)
7. [**ERC20 Decimals Assumption**](#7-ERC20-Decimals-Assumption)
8. [**batchLiquidateTroves Ignores Checks In liquidate**](#8-batchLiquidateTroves-Ignores-Checks-In-liquidate)
9. [**LUSD Minting To Restricted Address**](#9-LUSD-Minting-To-Restricted-Address)
10. [**No Rebasing Token Support**](#10-No-Rebasing-Token-Support)

[**Non-Critical**](#Non-Critical)
1. [**Use of ecrecover**](#1-Use-of-ecrecover)
2. [**Consider Avoiding Sensitive Terms**](#2-Consider-Avoiding-Sensitive-Terms)
3. [**Explicit Variable Not Used**](#3-Explicit-Variable-Not-Used)
4. [**Variable Scopes Not Explicit**](#4-Variable-Scopes-Not-Explicit)
5. [**chainid Non-Assembly Function Available**](#5-chainid-Non-Assembly-Function-Available)
6. [**Use of deprecated now**](#6-Use-of-deprecated-now)
7. [**bytes.concat() Can Be Used Over abi.encodePacked()**](#7-bytesconcat-can-be-used-over-abiencodepacked)
8. [**Contracts Lacking NatSpec Headers**](#8-Contracts-Lacking-NatSpec-Headers)
9. [**Odd Name State**](#9-Odd-Name-State)
10. [**Be Explicitly Honest About Risk**](#10-Be-Explicitly-Honest-About-Risk)

[**General Style**](#General-Style)
1. [**Underscore Notation Not Used or Not Used Consistently**](#1-Underscore-Notation-Not-Used-or-Not-Used-Consistently)
2. [**Power of Ten Literal Not In Scientific Notation**](#2-Power-of-Ten-Literal-Not-In-Scientific-Notation)
3. [**Use of Exponentiation Over Scientific Notation**](#3-Use-of-Exponentiation-Over-Scientific-Notation)
4. [**Inconsistent Named Returns**](#4-Inconsistent-Named-Returns)
5. [**Spelling Mistakes**](#5-Spelling-Mistakes)
6. [**Commented Code**](#6-Commented-Code)
7. [**Inconsistent Comment Capitalization**](#7-Inconsistent-Comment-Capitalization)
8. [**Inconsistent Trailing Period**](#8-Inconsistent-Trailing-Period)
9. [**Needless Comments For Devs**](#9-Needless-Comments-For-Devs)
10. [**Time Keyword Not Used**](#10-Time-Keyword-Not-Used)
11. [**Inconsistent Brace Style**](#11-Inconsistent-Brace-Style)
12. [**Inconsistent Quote Style In Comments**](#12-Inconsistent-Quote-Style-In-Comments)
13. [**Space Between License and Pragma**](#13-Space-Between-License-and-Pragma)
14. [**Extra Spacing**](#14-Extra-Spacing)
15. [**Header Inconsistency**](#15-Header-Inconsistency)
16. [**Inconsistent Function Declaration Style**](#16-Inconsistent-Function-Declaration-Style)
17. [**Variable Names Too General**](#17-Variable-Names-Too-General)
18. [**Functions Lacking NatSpec**](#18-Functions-Lacking-NatSpec)
19. [**State Lacking NatSpec**](#19-State-Lacking-NatSpec)
20. [**Inconsistent Exponentiation Style**](#20-Inconsistent-Exponentiation-Style)
21. [**Differing Ownable Style**](#21-Differing-Ownable-Style)
22. [**Return Variables With Return Statements**](#22-Return-Variables-With-Return-Statements)
23. [**Overflow Used For Max Int Type**](#23-overflow-used-for-max-int-type)

[**Style Guide Violations**](#Style-Guide-Violations)
1. [**Maximum Line Length**](#1-Maximum-Line-Length)
2. [**Order of Functions**](#2-Order-of-Functions)
3. [**Whitespace in Expressions**](#3-Whitespace-in-Expressions)
4. [**Control Structures structs**](#4-control-structures-struct)
5. [**Control Structures if else if else**](#5-Control-Structures-if-else-if-else)
6. [**Function Declaration Style Short**](#6-Function-Declaration-Style-Short)
7. [**Function Declaration Style Long**](#7-Function-Declaration-Style-Long)
8. [**Function Declaration Modifier Order**](#8-Function-Declaration-Modifier-Order)
9. [**Mappings**](#9-Mappings)
10. [**Single Quote String**](#10-Single-Quote-String)
11. [**Order of Layout**](#11-Order-of-Layout)
12. [**NatSpec**](#12-Natspec)

# Low Severity

## 1. Unchecked Cast May Overflow

As of Solidity 0.8 overflows are handled automatically; however, not for casting. Also, many files in the codebase use Solidity 0.6.11 which does not have overflow protection. For example `uint32(4294967300)` will result in `4` without reversion. Consider using OpenZepplin's SafeCast library.

*/Ethos-Core/contracts/TroveManager.sol*

```solidity
1329:	index = uint128(TroveOwners[_collateral].length.sub(1));
```

*/Ethos-Core/contracts/ActivePool.sol*

```solidity
277:	vars.netAssetMovement = int256(vars.toDeposit) - int256(vars.toWithdraw) - int256(vars.profit);
```

*/Ethos-Vault/contracts/ReaperVaultV2.sol*

```solidity
228:	return -int256(strategies[stratAddr].allocated);
235:	return -int256(stratCurrentAllocation - stratMaxAllocation);
248:	return int256(available);
```

*/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol*

```solidity
121:	roi = -int256(debt - amountFreed);
123:	roi = int256(amountFreed - debt);
```

*/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol*

```solidity
134:	roi = int256(profit);
136:	roi = -int256(allocated - totalAssets);
141:	roi -= int256(loss);
```

## 2. Consider a More Recent Solidity Version

Ethos Reserve uses a fork of the Liquity Protocol which uses a solidity version of `0.6.11`. It may be better for security to use battle-tested software; however, underflow and overflow oversights can potentially be a large problem, which is fixed post Solidity `0.8.0` (automatic reversion). Consider consulting a Liquity dev to help upgrade past Solidity version `0.8.0` or upgrade yourself depending on your confidence level with the system.

## 3. Code Function Does Not Match Comment

There is a comment in [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L665) that reads `// break if the loop reaches a Trove with ICR >= MCR`. The comment is not exactly true. The `else` block that is commented is triggered if `vars.backToNormalMode` AND `!vars.backToNormalMode` OR `vars.ICR >= vars.collMCR`. Over simplifying comments may lead to users making uninformed decisions when reading the code functionality.

In [BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L651) there is a require statement that reads `Max fee percentage must less than or equal to 100%` which is checked if the system is in recovery mode. If the system is not in recovery another require statement reads `Max fee percentage must be between 0.5% and 100%`. Both statements check if the max fee percentage falls within **inclusive** ranges, yet the second require message uses the partially ambiguous (usually said for non-inclusivity) word `between`. Consider changing [this](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L654) error message from `Max fee percentage must be between 0.5% and 100%` to `Max fee percentage must be between 0.5% and 100% inclusively`.

## 4. Low Coverage

The code coverage of the files in scope is not 100%. Consider a test coverage of 100%.

## 5. assert Used Over require

`assert` should only be used in tests. Consider changing all occurrences of `assert` to `require`. Prior to Solidity `0.8.0` `require` will refund all remaining gas whereas `assert` will not. Even after Solidity 0.8 [`assert` will result in a panic](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#panic-via-assert-and-error-via-require) which should not occur in production code. As stated in the Solidity Documentation: "[p]roperly functioning code should never create a Panic".

*/Ethos-Core/contracts/BorrowerOperations.sol*

```solidity
128:	assert(MIN_NET_DEBT > 0);
197:	assert(vars.compositeDebt > 0);
301:	assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
331:	assert(_collWithdrawal <= vars.coll);
```

*/Ethos-Core/contracts/TroveManager.sol*

```solidity
417:	assert(_LUSDInStabPool != 0);
1224:	assert(totalStakesSnapshot[_collateral] > 0);
1279:	assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
1342:	assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
1348:	assert(index <= idxLast);
1414:	assert(newBaseRate > 0);
1489:	assert(decayedBaseRate <= DECIMAL_PRECISION);
```

*/Ethos-Core/contracts/StabilityPool.sol*

```solidity
526:	assert(_debtToOffset <= _totalLUSDDeposits);
551:	assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
591:	assert(newP > 0);
```

*/Ethos-Core/contracts/LUSDToken.sol*

```solidity
312:	assert(sender != address(0));
313:	assert(recipient != address(0));
321:	assert(account != address(0));
329:	assert(account != address(0));
337:	assert(owner != address(0));
338:	assert(spender != address(0));
```

## 6. Drastic Compiler Version Variation

Ethos-Core and Ethos-Vault have drastically different compiler versions. One uses `0.6.11` (which is forked from Liquity) and the other uses `^0.8.0`. One major difference between the compiler versions is automatic overflow and underflow checks. In future development or third-party development of the system the two parts are going to need different attention to detail that may introduce bugs. It is best to keep the system as a whole on a similar compiler version. 

## 7. ERC20 Decimals Assumption

[EIP20](https://eips.ethereum.org/EIPS/eip-20#decimals) states in regard to the `decimals` method: "OPTIONAL - This method can be used to improve usability, but interfaces and other contracts MUST NOT expect these values to be present". The protocol assumes the `decimals` method is present which may affect the system if whole tokens are used.

*/Ethos-Core/contracts/CollateralConfig.sol*

```Solidity
uint256 decimals = IERC20(collateral).decimals();
```

## 8. batchLiquidateTroves Ignores Checks In liquidate

*This issue is from a [previous audit](https://www.coinspect.com/liquity-audit/) of the Liquity Protocol that has not been fixed in Ethos*

The [`liquidate`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L310) function of [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol) checks if a trove is active while [`batchLiquidateTroves`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L715) does not. Both are publically callable functions.

Please see finding ID: `CI-LQY-02` in the [2021 Coinspect Security Audit](https://www.coinspect.com/liquity-audit/) for more information.

## 9. LUSD Minting To Restricted Address

*This issue is from a [previous audit](https://github.com/trailofbits/publications/blob/master/reviews/Liquity.pdf) of the Liquity Protocol that has not been fixed in Ethos*

When transferring LUSD there is a check done ([`_requireValidRecipient`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L346)) to make sure tokens cannot be sent to an invalid address. The [`mint`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L185) is missing this check.

Please see finding ID: `TOB-LQT-003` in the [2021 Liquity Protocol Trail of Bits Audit](https://github.com/trailofbits/publications/blob/master/reviews/Liquity.pdf) for more information.

## 10. No Rebasing Token Support

In the protocol [overview video](https://www.loom.com/share/dc3f31b93aae412697eb105724a8d327) at time 2:40, Justin states, "liquity is designed to only support ETH, and we support **anything**".

Rebasing tokens are not checked for ([ex](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L497)). Balances must be adjusted for during transfers. 

Please see [this](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/167) issue utilizing similar code from a previous code4rena audit.

# Non-Critical

## 1. Use of ecrecover

Consider using OpenZeppelin's ECDSA contract instead of raw `ecrecover` to avoid signature malleability. 

*/Ethos-Core/contracts/LUSDToken.sol*

```solidity
287:	address recoveredAddress = ecrecover(digest, v, r, s);
```

The [2022 Topshelf Protocol PeckShield Audit](https://app.topshelf.finance/PeckShield-Audit-Report-Topshelf-v1.0.pdf) which uses the Liquity Protocol code also suggests checking the owner is not the zero address. See finding ID: `PVE-001`.

## 2. Consider Avoiding Sensitive Terms

Consider using the terms allowlist / denylist instead of whitelist / blacklist.

*/Ethos-Core/contracts/CollateralConfig.sol*

```solidity
37:	event CollateralWhitelisted(address _collateral, uint256 _decimals, uint256 _MCR, uint256 _CCR);
72:	emit CollateralWhitelisted(collateral, decimals, _MCRs[i], _CCRs[i]);
```

## 3. Explicit Variable Not Used

As described in the [Solidity documentation](https://docs.soliditylang.org/en/v0.4.21/types.html#integers), "`uint` and `int` are aliases for `uint256` and `int256`, respectively". There are moments in the codebase where `uint` / `int` is used instead of the explicit `uint256` / `int256`. It is best to be explicit with variable names to lessen confusion. Consider replacing instances of `uint` / `int` with `uint256` / `int256`.

*/Ethos-Core/contracts/BorrowerOperations.sol*
* [51-53](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L51-L53), [55-63](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L55-L63), [70-77](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L70-L77), [104-106](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L104-L106), [172](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L172), [206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L206), [242](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L242), [248](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L248), [254](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L254), [259](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L259), [264](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L264), [268](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268), [282](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L282), [347](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L347), [383](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L383), [388-389](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L388-L389), [393](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L393), [419](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L419), [421](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L421), [432-433](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L432-L433), [439-440](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L439-L440), [444](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L444), [460](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L460), [462](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L462), [466](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L466), [468](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L468), [470](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L470), [482](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L482), [484](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L484), [486](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L486), [505](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L505), [511](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L511), [517](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L517), [528](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L528), [533](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L533), [537](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L537), [542](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L542), [547](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L547), [551](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L551), [557](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L557), [567](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L567), [575](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L575), [616](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L616), [620](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L620), [624](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L624), [628](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L628), [632](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L632), [636](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L636), [644](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L644), [648](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L648), [663-665](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L663-L665), [667](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L667), [673](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L673), [675](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L675), [677](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L677), [684-686](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L684-L686), [688](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L688), [690](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L690), [695](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L695), [697](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L697), [699](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L699), [704-706](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L704-L706), [708](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L708), [715-716](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L715-L716), [727](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L727), [729](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L729), [731](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L731), [736](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L736), [738-739](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L738-L739), [744](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L744), [748](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L748).

*/Ethos-Core/contracts/TroveManager.sol*
* [48](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L48), [53-55](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L53-L55), [61](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L61), [63](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L63), [66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L66), [78-80](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L78-L80), [88](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L88), [91](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L91), [94](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L94), [104](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L104), [105](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L105), [112](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L112), [119-120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L119-L120), [133-134](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L133-L134), [136-137](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L136-L137), [141-143](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L141-L143), [147-150](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L147-L150), [153-154](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L153-L154), [161-169](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L161-L169), [173-181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L173-L181), [200-209](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L200-L209), [212-215](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L212-L215), [299](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L299), [303](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L303), [326](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L326), [343](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L343), [362-365](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L362-L365), [420](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L420), [444-446](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L444-L446), [450](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L450), [480-482](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L480-L482), [484](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L484), [492](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L492), [513](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L513), [588-590](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L588-L590), [676](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L676), [678-679](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L678-L679), [790-791](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L790-L791), [819](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L819), [869](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L869), [871](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L871), [915](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L915), [985-988](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L985-L988), [1018](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1018), [1022-1024](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1022-L1024), [1045](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1045), [1046](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1046), [1048](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1048), [1049](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1049), [1057-1059](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1057-L1059-L1059), [1061-1062](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1061), [1066-1068](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1066-L1068), [1070-1071](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1070-L1071), [1087-1088](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1087-L1088), [1123](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1123), [1124-1125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1124-L1125), [1129](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1129), [1132](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1132), [1138-1140](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1138-L1140), [1144](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1144), [1147](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1147), [1171](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1171), [1190](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1190), [1195](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1195), [1201-1203](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1201-L1203), [1213-1214](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1213-L1214), [1234-1235](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1234-L1235), [1251-1256](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1251-L1256), [1281](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1281), [1305](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1305), [1308-1309](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1308-L1309), [1316](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1316), [1339](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1339), [1345-1346](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1345-L1346), [1361](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1361), [1366](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1366), [1374-1376](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1374-L1376), [1384](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1384), [1398-1399](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1398-L1399), [1401-1402](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1401-L1402), [1404](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1404), [1408](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1408), [1411](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1411), [1425](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1425), [1429](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1429), [1433](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1433), [1440](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1440), [1444](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1444), [1448-1449](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1448-L1449), [1456](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1456), [1460](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1460), [1464](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1464), [1471](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1471), [1475](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1475), [1479](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1479), [1488](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1488), [1501](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1501), [1509-1511](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1509-L1511), [1516](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1516), [1538](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1538), [1544-1545](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1544-L1545), [1548](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1548), [1152](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1552), [1556](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1556), [1562](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1562), [1567](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1567), [1569](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1569), [1574](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1574), [1576](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1576), [1581](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1581), [1583](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1583), [1588](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1588), [1590](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1590).

*/Ethos-Core/contracts/ActivePool.sol*
* [62-63](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L62-L63), [159](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L159), [164](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L164), [171](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L171), [190](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L190), [197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L197), [204](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L204).

*/Ethos-Core/contracts/StabilityPool.sol*
* [179-181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L179-L181), [195](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L195), [197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L197), [214](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L214), [223](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L223), [226](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L226), [229](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L229), [233-234](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L233-L234), [246-248](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L246-L248), [252-257](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L252-L257), [255-257](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L255-L257), [307](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L307), [311](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L311), [323](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L323), [326](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L326), [333](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L333), [339](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L339), [346](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L346), [350-351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L350-L351), [353](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L353), [367](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L367), [369](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L369), [376-384](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L376-L384), [392](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L392), [396-397](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L396-L397), [399](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L399), [410](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L410), [417](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L417), [421-422](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L421-L422), [430](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L430), [433](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L433), [439](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L439), [451](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L451), [453](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L453), [466](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L466), [473-474](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L473-L474), [486](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L486), [488](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L488), [493](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L493), [497](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L497), [506-508](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L506-L508), [511](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L511), [524](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L524), [531](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L531), [547-549](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L547-L549), [556](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L556), [560](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L560), [569-570](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L569-570), [597](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L597), [608](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L608), [613-614](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L613-L614), [626-627](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L626-L627), [637](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L637), [639-640](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L639-L640), [650-654](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L650-L654), [657](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L657), [683-684](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L683-L684), [688](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L688), [693](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L693), [701-707](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L701-L707), [718-719](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L718-L719), [724](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L724), [730](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L730), [735](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L735), [737](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L737), [744](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L744), [776](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L776), [778](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L778), [783](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L783), [785](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L785), [794](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L794), [803](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L803), [807](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L807), [810](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810), [819-822](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L819-L822), [831](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831), [833](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L833), [841](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L841), [858-859](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L858-L859), [861](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L861), [864](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L864), [869](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L869), [873](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L873).

*/Ethos-Core/contracts/LQTY/CommunityIssuance.sol*
* [15](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L15), [41-46](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L41-L46), [51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L51), [53](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L53), [84](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L84), [101](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101), [107-108](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L107-L108), [124](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L124).

*/Ethos-Core/contracts/LQTY/LQTYStaking.sol*
* [20](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L20), [25-29](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25-L29), [35-36](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L35-L36), [56-62](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L56-L62), [104](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L104), [107](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L107), [110-111](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L110-L111), [120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L120), [142-143](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L142-L143), [147-148](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L147-L148), [153-155](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L153-L155), [177](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L177), [179](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L179), [189](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L189), [199](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L199), [203](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L203), [206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206), [208](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L208), [213](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L213), [217-219](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L217-L219), [227-228](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L227-L228), [233](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L233), [238-240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L238-L240).

*/Ethos-Core/contracts/LUSDToken.sol*
* [75](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L75), [266-267](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L266-L267).

## 4. Variable Scopes Not Explicit

There is an inconsistency with the explicitness of variable scopes. Variable scopes should be explicit and not left out if `public` (the default). Most variables are explicitly scoped ([ex1](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L25), [ex2](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L26), ...); however, not all. The following are not explicitly deemed `public`: 

*/Ethos-Core/contracts/BorrowerOperations.sol*
* [28](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L28), [30](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L30), [32](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L32).

*/Ethos-Core/contracts/TroveManager.sol*

* [31](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L31), [33](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L33).

## 5. chainid Non-Assembly Function Available

If the Solidity version of [LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol) is upgraded to `0.8.0`, the complexity brought by assembly can be avoided by using built in functions. `assembly { chainID := chainid() }` can be replaced by `uint256 chainID = block.chainid`.

*/Ethos-Core/contracts/LUSDToken.sol*

```solidity
300:	chainID := chainid()
```

## 6. Use of deprecated now

`now` is deprecated, consider using `block.timestamp` (as used in [other parts](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L87) of the codebase).

*/Ethos-Core/contracts/LUSDToken.sol*

```solidity
282:	require(deadline >= now, 'LUSD: expired deadline');
```

## 7. bytes.concat() Can Be Used Over abi.encodePacked()

If the contracts are upgraded to Solidity version >= `0.8.4` consider using `bytes.concat()` instead of `abi.encodePacked()`.

*/Ethos-Core/contracts/LUSDToken.sol*

```solidity
283:	bytes32 digest = keccak256(abi.encodePacked('\x19\x01',
```

## 8. Contracts Lacking NatSpec Headers

No contracts in scope have a fully annotated NatSpec contract header (`@title`, `@author`, etc. see [here](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html) for reference). Some contracts have a partial NatSpec header like [this](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L17), some have [none](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L13), and others have [non-NatSpec header comments](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L8-L11). Consider adding properly annotated NatSpec headers.

## 9. Odd Name State

There is a `NAME` state variable in [BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21), [ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30), [LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23), and [CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19) that doesn't seem needed. The same `NAME` state variable is commented out of [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L19).

If developers want to know the title of the contract they can reference the source contract name. If there isn't an explicit reason to keep the `NAME` state variables otherwise, consider removing them.

## 10. Be Explicitly Honest About Risk

Even if risk seems infinitesimally small it should never be stated as impossible. 

For example, in the context of a properly functioning EVM the following line will never overflow (no risk):

```solidity
uint256 foo = 32;
```

The next example can overflow in modern Solidity if `someArray.length` > `type(uint32).max` even if there are social or intended code concepts keeping the length within a proper length:

```solidity
uint32 foo = uint32(someArray.length);
```

[L1322-L1223](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1322-L1323) of `TroveManager.sol` reads: `"No risk of overflow, since troves have minimum LUSD debt of liquidation reserve plus MIN_NET_DEBT. 3e30 LUSD dwarfs the value of all wealth in the world ( which is < 1e15 USD)"`. This assumes the code keeping the minumum LUSD debt works as described and also assumes the US dollar never experiences extreme hyperinflation. This may be an infinitesimally small risk, but not zero.

Consider changing the line to `"Little to no risk of overflow, since troves have minimum LUSD debt of liquidation reserve plus MIN_NET_DEBT. 3e30 LUSD dwarfs the value of all wealth in the world ( which is < 1e15 USD)"`.

# General Style

## 1. Underscore Notation Not Used or Not Used Consistently

Consider using underscore notation to help with contract readability (Ex. `23453` -> `23_453`).

*/Ethos-Core/contracts/TroveManager.sol*

```solidity
53:	uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;
```

## 2. Power of Ten Literal Not In Scientific Notation

Power of ten literals > `1e2` are easier to read when expressed in scientific notation. Consider expressing large powers of ten in scientific notation.

*/Ethos-Core/contracts/TroveManager.sol*

```solidity
54:	uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5;
```

*/Ethos-Vault/contracts/ReaperVaultV2.sol*

```solidity
41:	uint256 public constant PERCENT_DIVISOR = 10000;
```

*/Ethos-Core/contracts/ActivePool.sol*

```solidity
145:	require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
```

## 3. Use of Exponentiation Over Scientific Notation

For better style and less computation consider replacing any power of 10 exponentiation (`10**3`) with its equivalent scientific notation (`1e3`).

*/Ethos-Vault/contracts/ReaperVaultV2.sol*

```solidity
40:	uint256 public constant DEGRADATION_COEFFICIENT = 10**18;
125:	lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6;
```

## 4. Inconsistent Named Returns

Some functions use named returns and others do not. It is best for code clearity to keep a consistent style.

1. The following contracts only have named returns (EX. `returns(uint256 foo)`): [CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol).
2. The following contracts only have non-named returns (EX. `returns(uint256)`): [CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol), and [ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol).
3. The following contracts have both: [BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol), [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol), [StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol), [LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol), [LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol), [ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol), [ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol), [ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol), and [ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol).

## 5. Spelling Mistakes

There are some spelling mistakes throughout the codebase. Consider fixing all spelling mistakes. The following were not included in the [automated findings](https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae#nc-7-typos):

*/Ethos-Core/contracts/CollateralConfig.sol*

* The word `lengths` is misspelled as [`lenghts`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L54).

*/Ethos-Core/contracts/TroveManager.sol*

* The word `elsewhere` is misspelled as [`elswhere`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L936).
* The word `borrowers'` is misspelled as [`borrowers's`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1081).

## 6. Commented Code 

There are a few commented code lines in [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol) ([L14](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L14), [L18](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L18), [L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L19), [L1413](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1413)). Commented code should be removed prior to production as they no longer serve a purpose and only add bulk or distraction. 

## 7. Inconsistent Comment Capitalization

There is an inconsistency in the capitalization of comments. For example, [this line](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L413) is not capitilized while - like most - [this line](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L681) is. Given almost all comments are capitilized, consider capitilizing all comments.

## 8. Inconsistent Trailing Period

It appears that the single line comments in the codebase generally follow the following format: 
1. If the comment is a single sentence, then a trailing `.` is left out ([ex1](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L107), [ex2](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L226), [ex3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L382), ...)
3. If the comment has more than one sentence, then a  trailing `.` is added ([ex1](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L309), [ex2](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1044), [ex3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1053), ...)

There are few, but some comments void this general style:
1. [ex1](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L93), [ex2](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L356), ...

Consider sticking to a single comment style for a less distracting developer experience.

## 9. Needless Comments For Devs

More documentation is almost always better than none at all; however, there is no need to document basic programming knowledge. Anyone who would want to read the Ethos Reserve code should have basic programming knowledge (knowledge of `if` / `else` control statements). There are times in the codebase where basic programming knowledge is commented. In some case like [L665](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L665), the comment may aid in understanding certain abstraction (a more complex case comparison, although [L665](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L665) does not tell its full story [see L3]), these are not included. The following comments should be considered for removal in [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol): [539](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L539), [708](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L708), [742](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L742).

## 10. Time Keyword Not Used

Time keywords like `hours` or `days` are used in the codebase ([ex1](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L24), [ex2](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L25)) yet not used [here](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L48). Consider changing `SECONDS_IN_ONE_MINUTE = 60;` to `SECONDS_IN_ONE_MINUTE = 1 minutes;` for consistency.

## 11. Inconsistent Brace Style

Some single line code does not include braces ([ex1](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L665), [ex2](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L860), ...) and others do ([ex1](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L616), [ex2](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L811), [ex3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L817), [ex4](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1142), ...). Some single line brace code also have differing spacing ([Type1](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L720), [Type2](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1158)). In other parts of the codebase single line braces are expanded ([ex](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L210)).

Consider sticking to a single line brace style.

## 12. Inconsistent Quote Style In Comments

[Some comments](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L130) use a single quote for quotations and [others](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L763) use a double quote. The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#other-recommendations) states that "[s]trings should be quoted with double-quotes instead of single-quotes". Although comments are not seen as active string (most likely does not void the Style Guide), it is best to stick to double quotes for consistency.

## 13. Space Between License and Pragma

All contracts in scope have a space between the license statement and pragma ([ex](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L2)). Although it does not void the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html), all example contracts in the Style Guide do not have a space between the license statement and pragma ([ex](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#blank-lines)). This format can also be seen throughout the Solidy Documentation ([ex](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html)). Consider removing the needless space.

## 14. Extra Spacing

There are times in [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol) where extra spaces or new lines are present when they are not needed: 

* [476](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L476) (double space multi comment line, [others](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L511) are single spaced), [665](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L665) (an extra space between `else if` brace and `else`), [742](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L742) (extra space after `else` brace and after `//`), [852](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L852) (2 extra new lines), [1323](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1323) (space after the opening parenthesis).

## 15. Header Inconsistency

[CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol) has only basic [structural headers](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L17) (describes basic elements of the contract) which can be removed if `_requireCallerIsStabilityPool` were converted to a modifier and the contract followed the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) layout.

Some contracts like [ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol) have meaningful headers that help developers understand the structure of the code. Some contracts like [CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol) have no headers at all. 

There is no need for [CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol) to have headers if properly formatted. Consider removing the headers or add headers to all contracts. 

## 16. Inconsistent Function Declaration Style

Some function declarations like [this](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L555) do not drop the opening parenthesis on a new line, but [some](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L476) do. 

This does not void the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#function-declaration) which states: "[f]or long function declarations, it is recommended to drop each argument onto its own line at the same indentation level as the function body. The closing parenthesis and opening bracket should be placed on their own line as well at the same indentation level as the function declaration". There is no comment on the placement of the opening parenthesis; however, consider sticking to a single style.

## 17. Variable Names Too General

There are [times](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L176) in the codebase where a variable of the name `vars` is used to hold local variables of a function. Although the type of these variables are explicitly named like `LocalVariables_openTrove` it is best to not use general names like `vars` as it could confuse the reader. Consider being more specific in naming variables.

## 18. Functions Lacking NatSpec

In order to understand variables in the codebase the [Liquity Documentation](https://github.com/liquity/dev) was needed. For example, in the [`openTrove`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L172) function, the use of the `_maxFeePercentage` variable was unclear. Consider adding a NatSpec header to all functions which include `@param` tags detailing the use of each variable. 

## 19. State Lacking NatSpec

Similarily to functions all state variables should be documented with NatSpec tags. To understand what [`baseRate`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L63) was, the [Liquity Documentation](https://github.com/liquity/dev) was needed. It is not good to force readers to reference forked code documentation to understand variable use. Consider documenting state varaibles with NatSpec.

## 20. Inconsistent Exponentiation Style

Non-symmetric spacing voids the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#other-recommendations); however, it is not clear if exponentiation should have a trailing and leading space or no spacing (`a**b` vs `a ** b`). Nevertheless, spacing in exponentiation should be consistant. 

In [this](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L433) example the `a**b` style is used, but in [this](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L494) example the `a ** b` style is used. 

Consider sticking to a single style.

## 21. Differing Ownable Style

Some contracts in the codebase are ownable; however, they use different styles of ownership. For example [BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol) uses an Ownable contract, whereas [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L21) uses a single variable `owner`. It is understood that the single variable use in [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L21) is "to circumvent contract size limit". Nonetheless, the same style should be used to avoid possible security confusion and future implementation overlooks.

Also consider using battle-tested ownable contracts like [Openzepplin's](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol).

## 22. Return Variables With Return Statements

Some functions in the codebase use named returns but still return a variable. This can confuse a reader.

*/Ethos-Core/contracts/TroveManager.sol*

* [329](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L329), [369](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L369), [899](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L899), [1321](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1321).

*/Ethos-Core/contracts/StabilityPool.sol*

* [511](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L511), [626](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L626).

*/Ethos-Vault/contracts/ReaperVaultERC4626.sol*

* [29](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L29), [37](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L37), [51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51), [66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66), [79](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79), [96](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L96), [122](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122), [165](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L165), [220](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L220), [240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L240).

*/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol*

* [104](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L104).

## 23. Overflow Used For Max Int Type

`uint256(-1)` is used to get the max value of a `uint256`. Since [Solidity `0.6.8`](https://docs.soliditylang.org/en/v0.6.8/types.html#integers) `type(uint256).max` may be used instead of forcing an overflow. Solidity `0.8.0` introduced automatic overflow detection, forcing non-unchecked instances of `uint256(-1)` to revert. Future developers may be confused by the use of `uint256(-1)`, consider using `type(uint256).max`.

*/Ethos-Core/contracts/ActivePool.sol*

```solidity
258:	vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);
```

# Style Guide Violations

## 1. Maximum Line Length

Lines with greater length than 120 characters are used. The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) suggests that all lines should be 120 characters or less in width.

The following lines are longer than 120 characters, it is suggested to shorten these lines:

*/Ethos-Core/contracts/BorrowerOperations.sol*
*  [105](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L105), [172](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L172), [190](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L190), [233](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L233), [235](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L235), [237](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L237), [248](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L248), [254](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L254), [259](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L259), [264](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L264), [268](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L268), [272](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L272), [276](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L276), [282](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L282), [300](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L300), [312](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L312), [343](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L343), [358](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L358), [419](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L419), [510](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L510), [511](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L511), [517](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L517), [528](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L528), [529](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L529), [530](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L530), [538](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L538), [546](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L546), [588](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L588), [637](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L637), [644](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L644), [645](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L645), [675](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L675), [697](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L697). 

*/Ethos-Core/contracts/TroveManager.sol*
*  [58](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L58), [97](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L97), [102](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L102), [114](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L114), [200](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L200), [201](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L201), [202](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L202), [338](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L338), [348](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L348), [351](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L351), [384](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L384), [393](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L393), [398](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L398), [404](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L404), [407](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L407), [416](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L416), [421](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L421), [428](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L428), [571](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L571), [572](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L572), [575](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L575), [618](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L618), [774](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L774), [775](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L775), [778](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L778), [819](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L819), [854](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L854), [887](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L887), [898](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L898), [902](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L902), [903](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L903), [915](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L915), [926](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L926), [934](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L934), [935](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L935), [937](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L937), [957](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L957), [995](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L995), [998](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L998), [1001](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1001), [1002](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1002), [1003](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1003), [1006](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1006), [1007](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1007), [1008](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1008), [1011](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1011), [1012](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1012), [1013](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1013), [1044](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1044), [1053](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1053), [1082](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1082), [1097](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1097), [1221](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1221), [1258](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1258), [1259](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1259), [1296](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1296), [1303](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1303), [1305](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1305), [1312](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1312), [1323](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1323), [1336](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1336), [1372](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1372), [1567](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1567), [1574](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1574), [1581](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1581), [1588](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1588). 

*/Ethos-Core/contracts/ActivePool.sol*
*  [47](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L47), [144](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L144), [258](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L258), [264](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L264), [269](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L269), [282](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L282), [287](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L287), [288](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L288). 

*/Ethos-Core/contracts/StabilityPool.sol*
*  [25](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L25), [27](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L27), [28](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L28), [31](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L31), [34](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L34), [42](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L42), [45](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L45), [46](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L46), [50](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L50), [52](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L52), [55](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L55), [58](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L58), [59](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L59), [65](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L65), [68](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L68), [69](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L69), [71](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L71), [74](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L74), [75](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L75), [80](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L80), [83](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L83), [89](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L89), [92](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L92), [93](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L93), [94](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L94), [99](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L99), [101](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L101), [103](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L103), [109](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L109), [130](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L130), [136](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L136), [142](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L142), [189](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L189), [205](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L205), [217](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L217), [220](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L220), [319](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L319), [361](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L361), [474](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L474), [547](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L547), [553](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L553), [626](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L626), [637](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L637), [657](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L657), [659](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L659), [671](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L671), [673](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L673), [762](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L762), [763](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L763). 

*/Ethos-Core/contracts/LQTY/LQTYStaking.sol*
*  [203](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L203). 

*/Ethos-Core/contracts/LUSDToken.sol*
*  [16](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L16), [22](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L22), [25](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L25), [238](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L238), [248](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L248). 

*/Ethos-Vault/contracts/ReaperVaultERC4626.sol*
*  [50](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L50), [65](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L65), [71](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L71), [76](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L76), [85](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L85), [86](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L86), [89](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L89), [93](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L93), [94](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L94), [103](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L103), [119](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L119), [128](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L128), [130](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L130), [136](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L136), [159](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L159), [160](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L160), [163](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L163), [180](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L180), [182](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L182), [207](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L207), [214](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L214). 

## 2. Order of Functions

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) suggests the following function order: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private.

The following contracts are not compliant (examples are only to prove the functions are out of order NOT a full description): 

* [BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol): external functions are positioned after internal functions.
* [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol): external functions are positioned after internal functions.
* [StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol): external functions are positioned after internal functions.
* [LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol): external functions are positioned after internal functions.
* [LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol): external functions are positioned after public functions.
* [ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol): external functions are positioned after public functions.
* [ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol): external functions are positioned after public functions.
* [ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol): external functions are positioned after internal functions.
* [ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol): external functions are positioned after internal functions.

## 3. Whitespace in Expressions

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#whitespace-in-expressions) recommends to *"[a]void extraneous whitespace [i]mmediately inside parenthesis, brackets or braces, with the exception of single line function declarations"*.

*/Ethos-Core/contracts/StabilityPool.sol*

```solidity
368:	if (_amount !=0) {_requireNoUnderCollateralizedTroves();}
849:	require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");
```

*/Ethos-Core/contracts/TroveManager.sol*

```solidity
1127:	if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }
1142:	if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }
```

## 4. Control Structures struct

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#control-structures) states that structs should "close on **their own line** at the same indentation level as the beginning of the declaration". The struct seen [here](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L112) is defined using a single line which voids the second Style Guide requirement.

## 5. Control Structures if else if else

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#control-structures) states that "[f]or if blocks which have an else or else if clause, the else should be placed on the same line as the if’s closing brace". This is followed perfectly [here](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol) and in all contracts except [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol): [651](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L651), [853](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L853).

## 6. Function Declaration Style Short

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#function-declaration) states that "[f]or short function declarations, it is recommended for the opening brace of the function body to be kept on the same line as the function declaration". It is not clear what length is exactly meant by "short" or "long". The [maximum line length](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) of 120 characters may be an indication, and in that case any expanded function declaration under 120 characters should be on a single line.

The following functions in their respective contracts are not compliant by this standard: 

*/Ethos-Core/contracts/TroveManager.sol*
* `getCurrentICR`. 

*/Ethos-Core/contracts/LQTY/CommunityIssuance.sol*
* `setAddresses`. 

*/Ethos-Vault/contracts/ReaperVaultV2.sol*
* `addStrategy`. 

*/Ethos-Vault/contracts/ReaperVaultERC4626.sol*
* `withdraw`, `redeem`. 

*/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol*
* `updateVeloSwapPath`.

## 7. Function Declaration Style Long

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#function-declaration) states that "[f]or long function declarations, it is recommended to drop each argument onto its own line at the same indentation level as the function body. The closing parenthesis and opening bracket should be placed on their own line as well at the same indentation level as the function declaration."

[This](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L898) function violates the description.

## 8. Function Declaration Modifier Order

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#function-declaration) suggests the following modifer order:  Visibility, Mutability, Virtual, Override, Custom modifiers.

*/Ethos-Core/contracts/CollateralConfig.sol*

* `getAllowedCollaterals` ('external', 'override', 'view'): view (Mutability) positioned after override (Override).
* `isCollateralAllowed` ('external', 'override', 'view'): view (Mutability) positioned after override (Override).
* `getCollateralDecimals` ('external', 'override', 'view', 'Custom'): view (Mutability) positioned after override (Override).
* `getCollateralMCR` ('external', 'override', 'view', 'Custom'): view (Mutability) positioned after override (Override).
* `getCollateralCCR` ('external', 'override', 'view', 'Custom'): view (Mutability) positioned after override (Override).

*/Ethos-Core/contracts/BorrowerOperations.sol*

* `_getNewNominalICRFromTroveChange` ('pure', 'internal'): internal (Visibility) positioned after pure (Mutability).
* `_getNewICRFromTroveChange` ('pure', 'internal'): internal (Visibility) positioned after pure (Mutability).

*/Ethos-Core/contracts/StabilityPool.sol*

* `depositSnapshots_S` ('external', 'override', 'view'): view (Mutability) positioned after override (Override).

*/Ethos-Core/contracts/LQTY/CommunityIssuance.sol*

* `setAddresses` ('external', 'onlyOwner', 'override'): override (Override) positioned after onlyOwner (Custom Modifiers).

*/Ethos-Core/contracts/LQTY/LQTYStaking.sol*

* `setAddresses` ('external', 'onlyOwner', 'override'): override (Override) positioned after onlyOwner (Custom Modifiers).

## 9. Mappings

Mapping whitespace not in accordance with the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#mappings). There are many voiding whitespaces (see link).

*/Ethos-Core/contracts/CollateralConfig.sol*

```solidity
35:	mapping (address => Config) public collateralConfig;
```

*/Ethos-Core/contracts/TroveManager.sol*

```solidity
86:	mapping (address => mapping (address => Trove)) public Troves;
88:	mapping (address => uint) public totalStakes;
91:	mapping (address => uint) public totalStakesSnapshot;
94:	mapping (address => uint) public totalCollateralSnapshot;
104:	mapping (address => uint) public L_Collateral;
105:	mapping (address => uint) public L_LUSDDebt;
109:	mapping (address => mapping (address => RewardSnapshot)) public rewardSnapshots;
116:	mapping (address => address[]) public TroveOwners;
119:	mapping (address => uint) public lastCollateralError_Redistribution;
120:	mapping (address => uint) public lastLUSDDebtError_Redistribution;
```

*/Ethos-Core/contracts/ActivePool.sol*

```solidity
41:	mapping (address => uint256) internal collAmount;
42:	mapping (address => uint256) internal LUSDDebt;
44:	mapping (address => uint256) public yieldingPercentage;
45:	mapping (address => uint256) public yieldingAmount;
46:	mapping (address => address) public yieldGenerator;
47:	mapping (address => uint256) public yieldClaimThreshold;
```

*/Ethos-Core/contracts/StabilityPool.sol*

```solidity
167:	mapping (address => uint256) internal collAmounts;
179:	mapping (address => uint) S;
186:	mapping (address => Deposit) public deposits;
187:	mapping (address => Snapshots) public depositSnapshots;
214:	mapping (uint128 => mapping(uint128 => mapping (address => uint))) public epochToScaleToSum;
223:	mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;
228:	mapping (address => uint) public lastCollateralError_Offset;
```

*/Ethos-Core/contracts/LQTY/LQTYStaking.sol*

```solidity
25:	mapping( address => uint) public stakes;
28:	mapping (address => uint) public F_Collateral;
32:	mapping (address => Snapshot) public snapshots;
35:	mapping (address => uint) F_Collateral_Snapshot;
```

*/Ethos-Core/contracts/LUSDToken.sol*

```solidity
54:	mapping (address => uint256) private _nonces;
57:	mapping (address => uint256) private _balances;
58:	mapping (address => mapping (address => uint256)) private _allowances;
62:	mapping (address => bool) public troveManagers;
63:	mapping (address => bool) public stabilityPools;
64:	mapping (address => bool) public borrowerOperations;
```

## 10. Single Quote String

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#other-recommendations) states that "[s]trings should be quoted with double-quotes instead of single-quotes". Consider using double-quotes to be compliant.

*/Ethos-Core/contracts/ActivePool.sol*

```solidity
5:	import './Interfaces/IActivePool.sol';
7:	import './Interfaces/IDefaultPool.sol';
```

*/Ethos-Core/contracts/StabilityPool.sol*

```solidity
5:	import './Interfaces/IBorrowerOperations.sol';
7:	import './Interfaces/IStabilityPool.sol';
8:	import './Interfaces/IBorrowerOperations.sol';
9:	import './Interfaces/ITroveManager.sol';
10:	import './Interfaces/ILUSDToken.sol';
11:	import './Interfaces/ISortedTroves.sol';
870:	require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');
874:	require(_amount > 0, 'StabilityPool: Amount must be non-zero');
```

*/Ethos-Core/contracts/LQTY/LQTYStaking.sol*

```solidity
266:	require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');
270:	require(_amount > 0, 'LQTYStaking: Amount must be non-zero');
```

*/Ethos-Core/contracts/LUSDToken.sol*

```solidity
280:	revert('LUSD: Invalid s value');
282:	require(deadline >= now, 'LUSD: expired deadline');
283:	bytes32 digest = keccak256(abi.encodePacked('\x19\x01',
283:	bytes32 digest = keccak256(abi.encodePacked('\x19\x01',
288:	require(recoveredAddress == owner, 'LUSD: invalid signature');
```

## 11. Order of Layout

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) suggests the following contract layout order: Type declarations, State variables, Events, Modifiers, Functions.

The following contracts are not compliant (examples are only to prove the layout are out of order NOT a full description): 

* [CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol): Modifiers appear after Functions.
* [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol): State (enum) appears after Events.

## 12. NatSpec

The [NatSpec notation](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html#documentation-example) requires the use of `///` or `/**` to differentiate from regular comments. There is an instance in the codebase where a `//` is used instead of `///` and where a `/*` is used instead of `/**`.

*/Ethos-Core/contracts/LQTY/CommunityIssuance.sol*

```solidity
83:    // @dev issues a set amount of Oath to the stability pool
```

```solidity
97:	/*
98:	 @dev funds the contract and updates the distribution
99:	 @param amount: amount of $OATH to send to the contract
100:	*/
```