## Summary<a name="Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | `abi.encode()` is less efficient than `abi.encodepacked()` | 2 | 200 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Setting the `constructor` to `payable` | 4 | 52 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Do not calculate constants | 2 | - |
| [GAS&#x2011;4](#GAS&#x2011;4) | Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function | 6 | 168 |
| [GAS&#x2011;5](#GAS&#x2011;5) | Using fixed bytes is cheaper than using `string` | 9 | - |
| [GAS&#x2011;6](#GAS&#x2011;6) | Using `delete` statement can save gas | 19 | - |
| [GAS&#x2011;7](#GAS&#x2011;7) | Use `assembly` to write address storage values | 2 | - |
| [GAS&#x2011;8](#GAS&#x2011;8) | Use hardcoded address instead `address(this)` | 34 | - |
| [GAS&#x2011;9](#GAS&#x2011;9) | `internal` functions only called once can be inlined to save gas | 4 | 88 |
| [GAS&#x2011;10](#GAS&#x2011;10) | Optimize names to save gas | 11 | 242 |
| [GAS&#x2011;11](#GAS&#x2011;11) | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 21 | - |
| [GAS&#x2011;12](#GAS&#x2011;12) | Structs can be packed into fewer storage slots by editing time variables and uint variables | 1 | 8000 |
| [GAS&#x2011;13](#GAS&#x2011;13) | Public Functions To External | 28 | - |
| [GAS&#x2011;14](#GAS&#x2011;14) | `require()` Should Be Used Instead Of `assert()` | 20 | - |
| [GAS&#x2011;15](#GAS&#x2011;15) | Save gas with the use of specific import statements | 96 | - |
| [GAS&#x2011;16](#GAS&#x2011;16) | Shorten the array rather than copying to a new one | 4 | - |
| [GAS&#x2011;17](#GAS&#x2011;17) | State variables can be packed into fewer storage slots | 1 | 2000 |
| [GAS&#x2011;18](#GAS&#x2011;18) | Help The Optimizer By Saving A Storage Variable’s Reference Instead Of Repeatedly Fetching It | 9 | - |
| [GAS&#x2011;19](#GAS&#x2011;19) | Superfluous event fields | 1 | - |
| [GAS&#x2011;20](#GAS&#x2011;20) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 30 | - |
| [GAS&#x2011;21](#GAS&#x2011;21) | `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops | 15 | 525 |
| [GAS&#x2011;22](#GAS&#x2011;22) | Using `unchecked` blocks to save gas | 6 | 120 |
| [GAS&#x2011;23](#GAS&#x2011;23) | Use v4.8.1 OpenZeppelin contracts | 3 | - |
| [GAS&#x2011;24](#GAS&#x2011;24) | Use solidity version 0.8.19 to gain some gas boost | 11 | 968 |
| [GAS&#x2011;25](#GAS&#x2011;25) | Using `storage` instead of `memory` saves gas | 5 | 1750 |
| [GAS&#x2011;26](#GAS&#x2011;26) | Using `10**X` for constants isn't gas efficient  | 2 | - |

Total: 346 contexts over 26 issues

## Gas Optimizations

### <a href="#Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


```solidity
284: domainSeparator(), keccak256(abi.encode(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L284

```solidity
305: return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L305











### <a href="#Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Setting the `constructor` to `payable`

Saves ~13 gas per instance

#### <ins>Proof Of Concept</ins>

```solidity
225: constructor() public
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L225

```solidity
57: constructor() public
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L57

```solidity
16: constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles)
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L16

```solidity
111: constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) ERC20(string(_name), string(_symbol))
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111





### <a href="#Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

#### <ins>Proof Of Concept</ins>


```solidity
54: uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L54

```solidity
55: uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L55






#### <a href="#Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function

Saves deployment costs

#### <ins>Proof Of Concept</ins>

```solidity
551: require(totals.totalDebtInSequence > 0);
754: require(totals.totalDebtInSequence > 0);
```


https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L551

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754


```solidity
155: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
181: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
180: require(strategies[_strategy].activation != 0, "Invalid strategy address");
193: require(strategies[_strategy].activation != 0, "Invalid strategy address");

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L181

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L180

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L193








### <a href="#Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Using fixed bytes is cheaper than using `string`

As a rule of thumb, use `bytes` for arbitrary-length raw byte data and string for arbitrary-length `string` (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of `bytes1` to `bytes32` because they are much cheaper.

#### <ins>Proof Of Concept</ins>


```solidity
30: string constant public NAME = "ActivePool";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30

```solidity
21: string constant public NAME = "BorrowerOperations";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21

```solidity
32: string constant internal _NAME = "LUSD Stablecoin";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L32

```solidity
33: string constant internal _SYMBOL = "LUSD";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L33

```solidity
34: string constant internal _VERSION = "1";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L34

```solidity
150: string constant public NAME = "StabilityPool";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150

```solidity
19: // string constant public NAME = "TroveManager";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L19

```solidity
19: string constant public NAME = "CommunityIssuance";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19

```solidity
23: string constant public NAME = "LQTYStaking";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23






### <a href="#Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

```solidity
253: vars.profit = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L253

```solidity
529: lastLUSDLossError_Offset = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L529

```solidity
578: currentScale = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L578

```solidity
756: compoundedDeposit = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L756

```solidity
387: singleLiquidation.debtToOffset = 0;
388: singleLiquidation.collToSendToSP = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L387-L388

```solidity
468: debtToOffset = 0;
469: collToSendToSP = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L468-L469

```solidity
505: singleLiquidation.debtToRedistribute = 0;
506: singleLiquidation.collToRedistribute = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L505-L506

```solidity
1192: Troves[_borrower][_collateral].stake = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1192

```solidity
1285: Troves[_borrower][_collateral].coll = 0;
1286: Troves[_borrower][_collateral].debt = 0;
1288: rewardSnapshots[_borrower][_collateral].collAmount = 0;
1289: rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1285

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1286

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1288

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1289



```solidity
215: strategies[_strategy].allocBPS = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L215

```solidity
369: uint256 totalLoss = 0;
371: uint256 vaultBalance = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L369

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L371



```solidity
537: lockedProfit = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L537





### <a href="#Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Use `assembly` to write address storage values

#### <ins>Proof Of Concept</ins>

```solidity
123: _HASHED_NAME = hashedName;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L123

```solidity
124: _HASHED_VERSION = hashedVersion;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L124




### <a href="#Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry's script.sol and solmate's `LibRlp.sol` contracts can help achieve this.

References: 
https://book.getfoundry.sh/reference/forge-std/compute-create-address 

https://twitter.com/transmissions11/status/1518507047943245824

#### <ins>Proof Of Concept</ins>

```solidity
210: IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _amount);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L210

```solidity
247: vars.ownedShares = vars.yieldGenerator.balanceOf(address(this));
280: IERC4626(yieldGenerator[_collateral]).deposit(uint256(vars.netAssetMovement), address(this));
282: IERC4626(yieldGenerator[_collateral]).withdraw(uint256(-vars.netAssetMovement), address(this), address(this));
288: vars.profit = IERC20(_collateral).balanceOf(address(this)).add(vars.yieldingAmount).sub(collAmount[_collateral]);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L247

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L280

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L282

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L288



```solidity
231: IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collAmount);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L231

```solidity
497: IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collChange);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L497

```solidity
530: require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L530

```solidity
305: return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L305

```solidity
349: _recipient != address(this),

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L349

```solidity
605: lusdToken.burn(address(this), _debtToOffset);
607: activePoolCached.sendCollateral(_collateral, address(this), _collToAdd);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L605

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L607



```solidity
777: lusdToken.sendToPool(_address, address(this), _amount);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L777

```solidity
797: lusdToken.returnFromPool(address(this), _depositor, LUSDWithdrawal);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L797

```solidity
103: OathToken.transferFrom(msg.sender, address(this), amount);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103

```solidity
128: lqtyToken.safeTransferFrom(msg.sender, address(this), _LQTYamount);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L128

```solidity
120: uint256 amount = startToken.balanceOf(address(this));
127: uint256 allocated = IVault(vault).strategies(address(this)).allocated;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L120

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L127



```solidity
180: ILendingPool(lendingPoolAddress).deposit(want, toReinvest, address(this), LENDER_REFERRAL_CODE_NONE);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L180

```solidity
200: ILendingPool(ADDRESSES_PROVIDER.getLendingPool()).withdraw(address(want), _withdrawAmount, address(this));

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L200

```solidity
227: return IERC20Upgradeable(want).balanceOf(address(this));

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L227

```solidity
233: address(this)

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L233

```solidity
153: require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L153

```solidity
246: available = Math.min(available, token.balanceOf(address(this)));

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L246

```solidity
279: return token.balanceOf(address(this)) + totalAllocated;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L279

```solidity
327: uint256 balBefore = token.balanceOf(address(this));
328: token.safeTransferFrom(msg.sender, address(this), _amount);
329: uint256 balAfter = token.balanceOf(address(this));

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L327-L329

```solidity
368: if (value > token.balanceOf(address(this))) {
373: vaultBalance = token.balanceOf(address(this));
386: uint256 actualWithdrawn = token.balanceOf(address(this)) - vaultBalance;
373: vaultBalance = token.balanceOf(address(this));

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L368

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L373

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L386

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L373



```solidity
528: token.safeTransferFrom(vars.stratAddr, address(this), vars.freeWantInStrat - vars.credit);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L528

```solidity
641: uint256 amount = IERC20Metadata(_token).balanceOf(address(this));

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L641



#### <ins>Recommended Mitigation Steps</ins>

Use hardcoded `address`







### <a href="#Summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> `internal` functions only called once can be inlined to save gas

#### <ins>Proof Of Concept</ins>

```solidity
320: function _requireCallerIsBorrowerOperationsOrDefaultPool

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L320


```solidity
432: function _getUSDValue

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L432

```solidity
455: function _updateTroveFromAdjustment
    

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L455

```solidity
476: function _moveTokensAndCollateralfromAdjustment
    

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L476



### <a href="#Summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

All in-scope contracts

#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.



### <a href="#Summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

#### <ins>Proof Of Concept</ins>


```solidity
133: toFree += profit;
141: roi -= int256(loss);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L133

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L141



```solidity
168: totalAllocBPS += _allocBPS;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168

```solidity
194: totalAllocBPS -= strategies[_strategy].allocBPS;
196: totalAllocBPS += _allocBPS;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L196



```solidity
214: totalAllocBPS -= strategies[_strategy].allocBPS;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L214

```solidity
390: value -= loss;
391: totalLoss += loss;
395: strategies[stratAddr].allocated -= actualWithdrawn;
396: totalAllocated -= actualWithdrawn;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L390

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L391

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L395

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L396



```solidity
444: stratParams.allocBPS -= bpsChange;
445: totalAllocBPS -= bpsChange;
450: stratParams.losses += loss;
451: stratParams.allocated -= loss;
452: totalAllocated -= loss;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L444

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L445

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L450

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L451

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L452



```solidity
505: strategy.gains += vars.gain;
514: strategy.allocated -= vars.debtPayment;
515: totalAllocated -= vars.debtPayment;
516: vars.debt -= vars.debtPayment;
520: strategy.allocated += vars.credit;
521: totalAllocated += vars.credit;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L505

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L514

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L515

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L516

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L520

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L521







### <a href="#Summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Structs can be packed into fewer storage slots by editing time variables and uint variables

The following structs contain time variables that are unlikely to ever reach max uint256 then it is possible to set them as uint64, saving gas slots.

#### <ins>Proof Of Concept</ins>

```solidity
25: struct StrategyParams {
        uint256 activation; // Activation block.timestamp
        uint256 feeBPS; // Performance fee taken from profit, in BPS
        uint256 allocBPS; // Allocation in BPS of vault's total assets
        uint256 allocated; // Amount of capital allocated to this strategy
        uint256 gains; // Total returns that Strategy has realized for Vault
        uint256 losses; // Total losses that Strategy has realized for Vault
        uint256 lastReport; // block.timestamp of the last time a report occured
    }

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L25

Can be packed into uint64 saving 1 storage slot:

```solidity
25: struct StrategyParams {
        uint64 activation; // Activation block.timestamp
        uint64 lastReport; // block.timestamp of the last time a report occured
        uint256 feeBPS; // Performance fee taken from profit, in BPS
        uint256 allocBPS; // Allocation in BPS of vault's total assets
        uint256 allocated; // Amount of capital allocated to this strategy
        uint256 gains; // Total returns that Strategy has realized for Vault
        uint256 losses; // Total losses that Strategy has realized for Vault
    }

```

It is potentially possible to lower the rest of the uint256 variable aswell since it is unlikely for them to ever reach max uint256. Allowing for saving about 4 storage slots.
```solidity
25: struct StrategyParams {
        uint64 activation; // Activation block.timestamp
        uint64 lastReport; // block.timestamp of the last time a report occured
        uint128 feeBPS; // Performance fee taken from profit, in BPS
        uint128 allocBPS; // Allocation in BPS of vault's total assets
        uint128 allocated; // Amount of capital allocated to this strategy
        uint128 gains; // Total returns that Strategy has realized for Vault
        uint128 losses; // Total losses that Strategy has realized for Vault
    }

```


### <a href="#Summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### <ins>Proof Of Concept</ins>


```solidity
function domainSeparator() public view override returns (bytes32) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L254

```solidity
function getDepositorCollateralGain(address _depositor) public view override returns (address[] memory assets, uint[] memory amounts) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L626

```solidity
function getDepositorLQTYGain(address _depositor) public view override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L683

```solidity
function getCompoundedLUSDDeposit(address _depositor) public view override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L718

```solidity
function batchLiquidateTroves(address _collateral, address[] memory _troveArray) public override {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L715

```solidity
function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1045

```solidity
function getCurrentICR(
        address _borrower,
        address _collateral,
        uint _price
    ) public view override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1054

```solidity
function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1123

```solidity
function getPendingLUSDDebtReward(address _borrower, address _collateral) public view override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1138

```solidity
function hasPendingRewards(address _borrower, address _collateral) public view override returns (bool) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1152

```solidity
function getRedemptionRate() public view override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1425

```solidity
function getRedemptionRateWithDecay() public view override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1429

```solidity
function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1440

```solidity
function getBorrowingRate() public view override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1456

```solidity
function getBorrowingRateWithDecay() public view override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1460

```solidity
function initialize(
        address _vault,
        address[] memory _strategists,
        address[] memory _multisigRoles,
        IAToken _gWant
    ) public initializer {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62

```solidity
function balanceOf() public view override returns (uint256) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L222

```solidity
function balanceOfWant() public view returns (uint256) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L226

```solidity
function balanceOfPool() public view returns (uint256) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L230

```solidity
function convertToShares(uint256 assets) public view override returns (uint256 shares) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51

```solidity
function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66

```solidity
function previewMint(uint256 shares) public view override returns (uint256 assets) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L139

```solidity
function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L183

```solidity
function availableCapital() public view returns (int256) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L225

```solidity
function balance() public view returns (uint256) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L278

```solidity
function getPricePerFullShare() public view returns (uint256) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L295

```solidity
function updateTvlCap(uint256 _newTvlCap) public {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L578

```solidity
function decimals() public view override returns (uint8) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L650






### <a href="#Summary">[GAS&#x2011;14]</a><a name="GAS&#x2011;14"> `require()` Should Be Used Instead Of `assert()`

#### <ins>Proof Of Concept</ins>


```solidity
128: assert(MIN_NET_DEBT > 0);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L128

```solidity
197: assert(vars.compositeDebt > 0);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197

```solidity
301: assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L301

```solidity
331: assert(_collWithdrawal <= vars.coll);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L331

```solidity
312: assert(sender != address(0));
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312

```solidity
313: assert(recipient != address(0));
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L313

```solidity
321: assert(account != address(0));
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L321

```solidity
329: assert(account != address(0));
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L329

```solidity
337: assert(owner != address(0));
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L337

```solidity
338: assert(spender != address(0));
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L338

```solidity
526: assert(_debtToOffset <= _totalLUSDDeposits);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L526

```solidity
551: assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L551

```solidity
591: assert(newP > 0);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L591

```solidity
417: assert(_LUSDInStabPool != 0);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L417

```solidity
1224: assert(totalStakesSnapshot[_collateral] > 0);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1224

```solidity
1279: assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1279

```solidity
1342: assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1342

```solidity
1348: assert(index <= idxLast);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1348

```solidity
1414: assert(newBaseRate > 0);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1414

```solidity
1489: assert(decayedBaseRate <= DECIMAL_PRECISION);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1489





### <a href="#Summary">[GAS&#x2011;15]</a><a name="GAS&#x2011;15"> Non-usage of specific imports

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace.
Instead, the Solidity docs recommend specifying imported symbols explicitly.
https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

A good example:
```solidity
import {OwnableUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/OwnableUpgradeable.sol";
import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
import {SafeCastLib} from "solmate/utils/SafeCastLib.sol";
import {ERC20} from "solmate/tokens/ERC20.sol";
import {IProducer} from "src/interfaces/IProducer.sol";
import {GlobalState, UserState} from "src/Common.sol";
```

#### <ins>Proof Of Concept</ins>


```solidity
5: import './Interfaces/IActivePool.sol';
6: import "./Interfaces/ICollateralConfig.sol";
7: import './Interfaces/IDefaultPool.sol';
8: import "./Interfaces/ICollSurplusPool.sol";
9: import "./Interfaces/ILQTYStaking.sol";
10: import "./Interfaces/IStabilityPool.sol";
11: import "./Interfaces/ITroveManager.sol";
12: import "./Dependencies/SafeMath.sol";
13: import "./Dependencies/Ownable.sol";
14: import "./Dependencies/CheckContract.sol";
15: import "./Dependencies/console.sol";
16: import "./Dependencies/SafeERC20.sol";
17: import "./Dependencies/IERC4626.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L5-L17

```solidity
5: import "./Interfaces/IBorrowerOperations.sol";
6: import "./Interfaces/ICollateralConfig.sol";
7: import "./Interfaces/ITroveManager.sol";
8: import "./Interfaces/ILUSDToken.sol";
9: import "./Interfaces/ICollSurplusPool.sol";
10: import "./Interfaces/ISortedTroves.sol";
11: import "./Interfaces/ILQTYStaking.sol";
12: import "./Dependencies/LiquityBase.sol";
13: import "./Dependencies/Ownable.sol";
14: import "./Dependencies/CheckContract.sol";
15: import "./Dependencies/console.sol";
16: import "./Dependencies/SafeERC20.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L5-L16

```solidity
5: import "./Dependencies/CheckContract.sol";
6: import "./Dependencies/Ownable.sol";
7: import "./Dependencies/SafeERC20.sol";
8: import "./Interfaces/ICollateralConfig.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L5-L8

```solidity
5: import "./Interfaces/ILUSDToken.sol";
6: import "./Interfaces/ITroveManager.sol";
7: import "./Dependencies/SafeMath.sol";
8: import "./Dependencies/CheckContract.sol";
9: import "./Dependencies/console.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L5-L9

```solidity
5: import './Interfaces/IBorrowerOperations.sol';
8: import './Interfaces/IBorrowerOperations.sol';
6: import "./Interfaces/ICollateralConfig.sol";
7: import './Interfaces/IStabilityPool.sol';
5: import './Interfaces/IBorrowerOperations.sol';
8: import './Interfaces/IBorrowerOperations.sol';
9: import './Interfaces/ITroveManager.sol';
10: import './Interfaces/ILUSDToken.sol';
11: import './Interfaces/ISortedTroves.sol';
12: import "./Interfaces/ICommunityIssuance.sol";
13: import "./Dependencies/LiquityBase.sol";
14: import "./Dependencies/SafeMath.sol";
15: import "./Dependencies/LiquitySafeMath128.sol";
16: import "./Dependencies/Ownable.sol";
17: import "./Dependencies/CheckContract.sol";
18: import "./Dependencies/console.sol";
19: import "./Dependencies/SafeERC20.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L8

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L6

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L7

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L8

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L9

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L10

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L11

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L12

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L13

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L14

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L15

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L16

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L17

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L18

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L19



```solidity
5: import "./Interfaces/ICollateralConfig.sol";
6: import "./Interfaces/ITroveManager.sol";
7: import "./Interfaces/IStabilityPool.sol";
8: import "./Interfaces/ICollSurplusPool.sol";
9: import "./Interfaces/ILUSDToken.sol";
10: import "./Interfaces/ISortedTroves.sol";
11: import "./Interfaces/ILQTYStaking.sol";
12: import "./Interfaces/IRedemptionHelper.sol";
13: import "./Dependencies/LiquityBase.sol";
14: import "./Dependencies/Ownable.sol";
15: import "./Dependencies/CheckContract.sol";
16: import "./Dependencies/IERC20.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L5-L16

```solidity
5: import "../Dependencies/IERC20.sol";
6: import "../Interfaces/ICommunityIssuance.sol";
7: import "../Dependencies/BaseMath.sol";
8: import "../Dependencies/LiquityMath.sol";
9: import "../Dependencies/Ownable.sol";
10: import "../Dependencies/CheckContract.sol";
11: import "../Dependencies/SafeMath.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L5-L11

```solidity
5: import "../Dependencies/BaseMath.sol";
6: import "../Dependencies/SafeMath.sol";
7: import "../Dependencies/Ownable.sol";
8: import "../Dependencies/CheckContract.sol";
9: import "../Dependencies/console.sol";
10: import "../Dependencies/IERC20.sol";
11: import "../Interfaces/ICollateralConfig.sol";
12: import "../Interfaces/ILQTYStaking.sol";
13: import "../Interfaces/ITroveManager.sol";
14: import "../Dependencies/LiquityMath.sol";
15: import "../Interfaces/ILUSDToken.sol";
16: import "../Dependencies/SafeERC20.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L5-L16

```solidity
5: import "./abstract/ReaperBaseStrategyv4.sol";
6: import "./interfaces/IAToken.sol";
7: import "./interfaces/IAaveProtocolDataProvider.sol";
8: import "./interfaces/ILendingPool.sol";
9: import "./interfaces/ILendingPoolAddressesProvider.sol";
10: import "./interfaces/IRewardsController.sol";
11: import "./libraries/ReaperMathUtils.sol";
12: import "./mixins/VeloSolidMixin.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L5-L12

```solidity
5: import "./ReaperVaultV2.sol";
6: import "./interfaces/IERC4626Functions.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L5-L6

```solidity
5: import "./interfaces/IERC4626Events.sol";
6: import "./interfaces/IStrategy.sol";
7: import "./libraries/ReaperMathUtils.sol";
8: import "./mixins/ReaperAccessControl.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L5-L8




#### <ins>Recommended Mitigation Steps</ins>

Use specific imports syntax per solidity docs recommendation.



### <a href="#Summary">[GAS&#x2011;16]</a><a name="GAS&#x2011;16"> Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

#### <ins>Proof Of Concept</ins>


```solidity
807: uint[] memory amounts = new uint[](collaterals.length);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L807

```solidity
313: address[] memory borrowers = new address[](1);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L313

```solidity
227: uint[] memory amounts = new uint[](collaterals.length);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L227

```solidity
660: bytes32[] memory cascadingAccessRoles = new bytes32[](5);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L660





### <a href="#Summary">[GAS&#x2011;17]</a><a name="GAS&#x2011;17"> State variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper

#### <ins>Proof Of Concept</ins>


```solidity
15: contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {
    ...
    bool public initialized = false;
    uint256 constant public MIN_ALLOWED_MCR = 1.1 ether; // 110%
    uint256 constant public MIN_ALLOWED_CCR = 1.5 ether; // 150%
    address[] public collaterals; // for returning entire list of allowed collaterals
    ...

```

Changing to the following can save 1 storage slot:
```solidity
15: contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {
    ...
    bool public initialized = false;
    address[] public collaterals; // for returning entire list of allowed collaterals
    uint256 constant public MIN_ALLOWED_MCR = 1.1 ether; // 110%
    uint256 constant public MIN_ALLOWED_CCR = 1.5 ether; // 150%
    ...

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L15




### <a href="#Summary">[GAS&#x2011;18]</a><a name="GAS&#x2011;18"> Help The Optimizer By Saving A Storage Variable's Reference Instead Of Repeatedly Fetching It

To help the optimizer, declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array.
The effect can be quite significant.
As an example, instead of repeatedly calling someMap[someIndex], save its reference like this: SomeStruct storage someStruct = someMap[someIndex] and use it.

#### <ins>Proof Of Concept</ins>


```solidity
822: uint currentG = epochToScaleToG[currentEpochCached][currentScaleCached];
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L822

```solidity
833: uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][collateral];
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L833

```solidity
208: uint F_Collateral_Snapshot = snapshots[_user].F_Collateral_Snapshot[collateral];
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L208

```solidity
209: amounts[i] = stakes[_user].mul(F_Collateral[collateral].sub(F_Collateral_Snapshot)).div(DECIMAL_PRECISION);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L209

```solidity
230: snapshots[_user].F_Collateral_Snapshot[collateral] = F_Collateral[collateral];
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L230

```solidity
231: amounts[i] = F_Collateral[collateral];
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L231

```solidity
266: StrategyParams storage params = strategies[strategy];
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L266

```solidity
379: uint256 strategyBal = strategies[stratAddr].allocated;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L379

```solidity
395: strategies[stratAddr].allocated -= actualWithdrawn;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L395






### <a href="#Summary">[GAS&#x2011;19]</a><a name="GAS&#x2011;19"> Superfluous event fields

`block.number` and `block.timestamp` are added to the event information by default, so adding them manually will waste additional gas.

#### <ins>Proof Of Concept</ins>


```solidity
1505: emit LastFeeOpTimeUpdated(block.timestamp);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1505





### <a href="#Summary">[GAS&#x2011;20]</a><a name="GAS&#x2011;20"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
35: uint8 constant internal _DECIMALS = 18;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L35

```solidity
182: uint128 scale;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L182

```solidity
183: uint128 epoch;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L183

```solidity
200: uint128 public currentScale;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L200

```solidity
203: uint128 public currentEpoch;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L203

```solidity
817: uint128 currentScaleCached = currentScale;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L817

```solidity
818: uint128 currentEpochCached = currentEpoch;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L818

```solidity
648: uint128 epochSnapshot;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L648

```solidity
649: uint128 scaleSnapshot;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L649

```solidity
739: uint128 epochSnapshot = snapshots.epoch;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L739

```solidity
738: uint128 scaleSnapshot = snapshots.scale;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L738

```solidity
745: uint128 scaleDiff = currentScale.sub(scaleSnapshot);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L745

```solidity
807: uint[] memory amounts = new uint[](collaterals.length);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L807

```solidity
82: uint128 arrayIndex;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L82

```solidity
1344: uint128 index = Troves[_borrower][_collateral].arrayIndex;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1344

```solidity
110: uint[] memory collGainAmounts;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L110

```solidity
227: uint[] memory amounts = new uint[](collaterals.length);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L227

```solidity
35: uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35






### <a href="#Summary">[GAS&#x2011;21]</a><a name="GAS&#x2011;21"> `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

#### <ins>Proof Of Concept</ins>


```solidity
108: for(uint256 i = 0; i < numCollaterals; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108

```solidity
56: for(uint256 i = 0; i < _collaterals.length; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56

```solidity
351: for (uint i = 0; i < numCollaterals; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351

```solidity
397: for (uint i = 0; i < numCollaterals; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L397

```solidity
640: for (uint i = 0; i < assets.length; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640

```solidity
810: for (uint i = 0; i < collaterals.length; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810

```solidity
859: for (uint i = 0; i < numCollaterals; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L859

```solidity
608: for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608

```solidity
690: for (vars.i = 0; vars.i < _n; vars.i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690

```solidity
808: for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808

```solidity
882: for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882

```solidity
206: for (uint i = 0; i < assets.length; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206

```solidity
228: for (uint i = 0; i < collaterals.length; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228

```solidity
240: for (uint i = 0; i < numCollaterals; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240





### <a href="#Summary">[GAS&#x2011;22]</a><a name="GAS&#x2011;22"> Using `unchecked` blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isnâ€™t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an `unchecked` block

#### <ins>Proof Of Concept</ins>

```solidity
330: _amount = balAfter - balBefore;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L330

```solidity
384: uint256 remaining = value - vaultBalance;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L384

```solidity
526: token.safeTransfer(vars.stratAddr, vars.credit - vars.freeWantInStrat);
528: token.safeTransferFrom(vars.stratAddr, address(this), vars.freeWantInStrat - vars.credit);
533: vars.lockedProfitBeforeLoss = _calculateLockedProfit() + vars.gain - vars.fees;
535: lockedProfit = vars.lockedProfitBeforeLoss - vars.loss;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L526

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L528

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L533

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L535












### <a href="#Summary">[GAS&#x2011;23]</a><a name="GAS&#x2011;23"> Use v4.8.1 OpenZeppelin contracts

The upcoming v4.8.1 version of OpenZeppelin provides many small gas optimizations.
https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.8.1

#### <ins>Proof Of Concept</ins>

```solidity
    "@openzeppelin/contracts": "^3.3.0"
```

https://github.com/code-423n4/2023-02-ethos/blob/main/../2023-02-ethos/blob/main/Ethos-Core/package.json#L34

```solidity
    "@openzeppelin/contracts": "^4.7.3"
```

https://github.com/code-423n4/2023-02-ethos/blob/main/../2023-02-ethos/blob/main/Ethos-Vault/package.json#L41

```solidity
    "@openzeppelin/contracts-upgradeable": "^4.7.3"
```

https://github.com/code-423n4/2023-02-ethos/blob/main/../2023-02-ethos/blob/main/Ethos-Vault/package.json#L42



#### <ins>Recommended Mitigation Steps</ins>

Update OpenZeppelin Contracts Usage in package.json and require at least version of 4.8.1






### <a href="#Summary">[GAS&#x2011;24]</a><a name="GAS&#x2011;24"> Use solidity version 0.8.19 to gain some gas boost

Upgrade to the latest solidity version 0.8.19 to get additional gas savings.
See latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3

```solidity
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3

```solidity
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3

```solidity
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3





### <a href="#Summary">[GAS&#x2011;25]</a><a name="GAS&#x2011;25"> Using `storage` instead of `memory` saves gas

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another `memory` array/struct

#### <ins>Proof Of Concept</ins>


```solidity
687: Snapshots memory snapshots = depositSnapshots[_depositor];
722: Snapshots memory snapshots = depositSnapshots[_depositor];

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L687

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L722



```solidity
687: Snapshots memory snapshots = depositSnapshots[_depositor];
722: Snapshots memory snapshots = depositSnapshots[_depositor];

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L687

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L722



```solidity
166: address[2] memory step = _newSteps[i];

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L166





### <a href="#Summary">[GAS&#x2011;26]</a><a name="GAS&#x2011;26"> Using `10**X` for constants isn't gas efficient 

In Solidity, a constant expression in a variable will compute the expression everytime the variable is called. It's not the result of the expression that is stored, but the expression itself.

As Solidity supports the scientific notation, constants of form 10**X can be rewritten as 1eX to save the gas cost from the calculation with the exponentiation operator **.

#### <ins>Proof Of Concept</ins>


```solidity
40: uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40

```solidity
125: lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125



#### <ins>Recommended Mitigation Steps</ins>

Replace 10**X with 1eX

