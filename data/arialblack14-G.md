# GAS OPTIMIZATION REPORT

### Summary of optimizations


| Number       | Issue details                                                                                                                                         | Instances |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------: |
| [G-1](#G1)   | `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when  it is not possible for them to overflow, for example when used in `for` and `while` loops |    15    |
| [G-2](#G2)   | Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate.                                         |     5     |
| [G-3](#G3)   | Using storage instead of memory for structs/arrays saves gas.                                                                                         |    12    |
| [G-4](#G4)   | Usage of`uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead.                                                                              |    31    |
| [G-5](#G5)   | `abi.encode()` is less efficient than `abi.encodePacked()`                                                                                            |     2     |
| [G-6](#G6)   | Use`bytes32` instead of string.                                                                                                                       |     9     |
| [G-7](#G7)   | Internal functions only called once can be inlined to save gas.                                                                                       |    25    |
| [G-8](#G8)   | `>=` costs less gas than `>`.                                                                                                                         |    39    |
| [G-9](#G9)   | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables                                                                                |    22    |
| [G-10](#G10) | Import only what you need.                                                                                                                            |    102    |
| [G-11](#G11) | Using fixed bytes is cheaper than using`string`.                                                                                                      |     7     |
| [G-12](#G13) | Use assembly to check for address(0)                                                                                                                  |     4     |

*Total: 12 issues.*
*The instances of the issue with ID `G-12 ` differ from the ones on the c4udit output.*

---

### Gas Optimizations

### <a id=G1>[G-1]</a> `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when  it is not possible for them to overflow, for example when used in `for` and `while` loops

##### Description

In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck [cannot be made inline](https://github.com/ethereum/solidity/issues/10695).
Example for loop:

```Solidity
for (uint i = 0; i < length; i++) {
// do something that doesn't change the value of i
}
```

In this example, the for loop post condition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body `is 2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks. You should manually do this by:

```Solidity
for (uint i = 0; i < length; i = unchecked_inc(i)) {
	// do something that doesn't change the value of i
}

function unchecked_inc(uint i) returns (uint) {
	unchecked {
		return i + 1;
	}
}
```

Or just:

```Solidity
for (uint i = 0; i < length;) {
	// do something that doesn't change the value of i
	unchecked { i++; 	}
}
```

Note that it’s important that the call to `unchecked_inc` is inlined. This is only possible for solidity versions starting from `0.8.2`.

Gas savings: roughly speaking this can save 30-40 gas per loop iteration. For lengthy loops, this can be significant!
(This is only relevant if you are using the default solidity checked arithmetic.)

##### Recommendation

Use the `unchecked` keyword

##### *Instances (15):*

File: [2023-02-ethos/Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L108 )

```solidity
108: for(uint256 i = 0; i < numCollaterals; i++) {
```

File: [2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L56 )

```solidity
56: for(uint256 i = 0; i < _collaterals.length; i++) {
```

File: [2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206 )

```solidity
206: for (uint i = 0; i < assets.length; i++) {
228: for (uint i = 0; i < collaterals.length; i++) {
240: for (uint i = 0; i < numCollaterals; i++) {
```

File: [2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L351 )

```solidity
351: for (uint i = 0; i < numCollaterals; i++) {
397: for (uint i = 0; i < numCollaterals; i++) {
640: for (uint i = 0; i < assets.length; i++) {
810: for (uint i = 0; i < collaterals.length; i++) {
831: for (uint i = 0; i < collaterals.length; i++) {
859: for (uint i = 0; i < numCollaterals; i++) {
```

File: [2023-02-ethos/Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L608 )

```solidity
608: for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
690: for (vars.i = 0; vars.i < _n; vars.i++) {
808: for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
882: for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
```

### <a id=G2>[G-2]</a> Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate.

##### Description

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

```Solidity
mapping(address => uint256) public nonces;
-mapping (address => bool) public minters;
-mapping (address => bool) public markets;
+mapping (address => uint256) public markets_minters;
+            // address => uint256 => 1 (minters true)
+            // address => uint256 => 2 (minters false)
+            // address => uint256 => 3 (markets true)
+            // address => uint256 => 4 (markets false)```


##### Recommendation
Where appropriate, you can combine multiple address mappings into a single mapping of an address to a struct.

##### *Instances (5):*
File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L58 )
```solidity
58: mapping (address => mapping (address => uint256)) private _allowances;
```

File: [2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L214 )

```solidity
214: mapping (uint128 => mapping(uint128 => mapping (address => uint))) public epochToScaleToSum;
223: mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;
```

File: [2023-02-ethos/Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L86 )

```solidity
86: mapping (address => mapping (address => Trove)) public Troves;
109: mapping (address => mapping (address => RewardSnapshot)) public rewardSnapshots;
```

### <a id=G3>[G-3]</a> Using storage instead of memory for structs/arrays saves gas.

##### Description

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.
Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct([src](https://ethereum.stackexchange.com/questions/128380/why-using-storage-keyword-instead-of-memory-cost-less-gas))

##### Recommendation

Use storage for `struct/array`

##### *Instances (12):*

File: [2023-02-ethos/Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L105 )

```solidity
105: address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();
```

File: [2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L147 )

```solidity
147: (address[] memory collGainAssets, uint[] memory collGainAmounts) = _getPendingCollateralGain(msg.sender);
226: address[] memory collaterals = collateralConfig.getAllowedCollaterals();
227: uint[] memory amounts = new uint[](collaterals.length);
```

File: [2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L332 )

```solidity
332: (address[] memory assets, uint[] memory amounts) = getDepositorCollateralGain(msg.sender);
376: (address[] memory assets, uint[] memory amounts) = getDepositorCollateralGain(msg.sender);
806: address[] memory collaterals = collateralConfig.getAllowedCollaterals();
807: uint[] memory amounts = new uint[](collaterals.length);
857: address[] memory collaterals = collateralConfig.getAllowedCollaterals();
```

File: [2023-02-ethos/Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L313 )

```solidity
313: address[] memory borrowers = new address[](1);
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L660 )

```solidity
660: bytes32[] memory cascadingAccessRoles = new bytes32[](5);
```

File: [2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L204 )

```solidity
204: bytes32[] memory cascadingAccessRoles = new bytes32[](5);
```

### <a id=G4>[G-4]</a> Usage of `uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead.

##### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

##### Recommendation

Use `uint256` instead.

##### *Instances (31):*

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L35 )

```solidity
35: uint8 constant internal _DECIMALS = 18;
268: uint8 v,
407: function decimals() external view override returns (uint8) {
```

File: [2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L147 )

```solidity
147: using LiquitySafeMath128 for uint128;
182: uint128 scale;
183: uint128 epoch;
200: uint128 public currentScale;
203: uint128 public currentEpoch;
214: mapping (uint128 => mapping(uint128 => mapping (address => uint))) public epochToScaleToSum;
223: mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;
247: event S_Updated(address _collateral, uint _S, uint128 _epoch, uint128 _scale);
248: event G_Updated(uint _G, uint128 _epoch, uint128 _scale);
249: event EpochUpdated(uint128 _currentEpoch);
250: event ScaleUpdated(uint128 _currentScale);
558: uint128 currentScaleCached = currentScale;
559: uint128 currentEpochCached = currentEpoch;
648: uint128 epochSnapshot;
649: uint128 scaleSnapshot;
699: uint128 epochSnapshot = snapshots.epoch;
700: uint128 scaleSnapshot = snapshots.scale;
738: uint128 scaleSnapshot = snapshots.scale;
739: uint128 epochSnapshot = snapshots.epoch;
745: uint128 scaleDiff = currentScale.sub(scaleSnapshot);
817: uint128 currentScaleCached = currentScale;
818: uint128 currentEpochCached = currentEpoch;
```

File: [2023-02-ethos/Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L82 )

```solidity
82: uint128 arrayIndex;
1321: function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
1329: index = uint128(TroveOwners[_collateral].length.sub(1));
1344: uint128 index = Troves[_borrower][_collateral].arrayIndex;
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35 )

```solidity
35: uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L650 )

```solidity
650: function decimals() public view override returns (uint8) {
```

### <a id=G5>[G-5]</a> `abi.encode()` is less efficient than `abi.encodePacked()`

##### Description

`abi.encode` will apply ABI encoding rules. Therefore all elementary types are padded to 32 bytes and dynamic arrays include their length. Therefore it is possible to also decode this data again (with `abi.decode`) when the type are known.

`abi.encodePacked` will only use the only use the minimal required memory to encode the data. E.g. an address will only use 20 bytes and for dynamic arrays only the elements will be stored without length. For more info see the Solidity docs for packed mode.

For the input of the `keccak` method it is important that you can ensure that the resulting bytes of the encoding are unique. So if you always encode the same types and arrays always have the same length then there is no problem. But if you switch the parameters that you encode or encode multiple dynamic arrays you might have conflicts.
For example:
`abi.encodePacked(address(0x0000000000000000000000000000000000000001), uint(0))` encodes to the same as `abi.encodePacked(uint(0x0000000000000000000000000000000000000001000000000000000000000000), address(0))`
and `abi.encodePacked(uint[](1,2), uint[](3))` encodes to the same as `abi.encodePacked(uint[](1), uint[](2,3))`
Therefore these examples will also generate the same hashes even so they are different inputs.
On the other hand you require less memory and therefore in most cases abi.encodePacked uses less gas than abi.encode.

##### Recommendation

Use `abi.encodePacked()` where possible to save gas

##### *Instances (2):*

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L284 )

```solidity
284: domainSeparator(), keccak256(abi.encode(
305: return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));
```

### <a id=G6>[G-6]</a> Use `bytes32` instead of string.

##### Description

Use `bytes32` instead of `string` to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming than `bytes32`.

##### Recommendation

Use `bytes32` instead of `string`

##### *Instances (9):*

File: [2023-02-ethos/Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L30 )

```solidity
30: string constant public NAME = "ActivePool";
```

File: [2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L21 )

```solidity
21: string constant public NAME = "BorrowerOperations";
```

File: [2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19 )

```solidity
19: string constant public NAME = "CommunityIssuance";
```

File: [2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23 )

```solidity
23: string constant public NAME = "LQTYStaking";
```

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L32 )

```solidity
32: string constant internal _NAME = "LUSD Stablecoin";
33: string constant internal _SYMBOL = "LUSD";
34: string constant internal _VERSION = "1";
```

File: [2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L150 )

```solidity
150: string constant public NAME = "StabilityPool";
```

File: [2023-02-ethos/Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L19 )

```solidity
19: // string constant public NAME = "TroveManager";
```

### <a id=G7>[G-7]</a> Internal functions only called once can be inlined to save gas.

##### Description

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.
Check this out: [link](https://betterprogramming.pub/solidity-gas-optimizations-and-tricks-2bcee0f9f1f2)
Note: To sum up, when there is an internal function that is only called once, there is no point for that function to exist.

##### Recommendation

Remove the function and make it inline if it is a simple function.

##### *Instances (25):*

File: [2023-02-ethos/Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L320 )

```solidity
320: function _requireCallerIsBorrowerOperationsOrDefaultPool() internal view {
```

File: [2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L203 )

```solidity
265: function _requireUserHasStake(uint currentStake) internal pure {
269: function _requireNonZeroAmount(uint _amount) internal pure {
```

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L311 )

```solidity
380: function _requireCallerIsTroveMorSP() internal view {
393: function _requireMintingNotPaused() internal view {
```

File: [2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L416 )

```solidity
597: function _moveOffsetCollAndDebt(address _collateral, uint _collToAdd, uint _debtToOffset) internal {
613: function _decreaseLUSD(uint _amount) internal {
637: function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {
657: function _getSingularCollateralGain(uint _initialDeposit, address _collateral, Snapshots storage _snapshots) internal view returns (uint) {
693: function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {
776: function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {
783: function _sendCollateralGainToDepositor(address _collateral, uint _amount) internal {
794: function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {
840: function _payOutLQTYGains(ICommunityIssuance _communityIssuance, address _depositor) internal {
848: function _requireCallerIsActivePool() internal view {
869: function _requireUserHasDeposit(uint _initialDeposit) internal pure {
873: function _requireNonZeroAmount(uint _amount) internal pure {
```

File: [2023-02-ethos/Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1538 )

```solidity
1538: function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L74 )

```solidity
74: function _adjustPosition(uint256 _debt) internal override {
104: function _liquidateAllPositions() internal override returns (uint256 amountFreed) {
114: function _harvestCore(uint256 _debt) internal override returns (int256 roi, uint256 repayment) {
206: function _claimRewards() internal {
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L287 )

```solidity
2418: function _calculateLockedProfit() internal view returns (uint256) {
432: function _reportLoss(address strategy, uint256 loss) internal {
462: function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {
```

### <a id=G8>[G-8]</a> `>=` costs less gas than `>`.

##### Description

The compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=`, which [saves 3 gas](https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde)

##### Recommendation

Use `>=` instead if appropriate.

##### *Instances (39):*

File: [2023-02-ethos/Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L264 )

```solidity
264: if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {
269: } else if(vars.percentOfFinalBal < vars.yieldingPercentage && vars.yieldingPercentage.sub(vars.percentOfFinalBal) > yieldingPercentageDrift) {
278: if (vars.netAssetMovement > 0) {
```

File: [2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L128 )

```solidity
128: assert(MIN_NET_DEBT > 0);
197: assert(vars.compositeDebt > 0);
301: assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
337: if (!_isDebtIncrease && _LUSDChange > 0) {
552: require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
```

File: [2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L152 )

```solidity
152: if (_LQTYamount > 0) {
181: if (totalLQTYStaked > 0) {collFeePerLQTYStaked = _collFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}
191: if (totalLQTYStaked > 0) {LUSDFeePerLQTYStaked = _LUSDFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}
266: require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');
270: require(_amount > 0, 'LQTYStaking: Amount must be non-zero');
```

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L279 )

```solidity
279: if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {
```

File: [2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L591 )

```solidity
591: assert(newP > 0);
```

File: [2023-02-ethos/Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L397 )

```solidity
397: } else if ((_ICR > _100pct) && (_ICR < _MCR)) {
424: if (singleLiquidation.collSurplus > 0) {
431: } else { // if (_ICR >= _MCR && ( _ICR >= _TCR || singleLiquidation.entireTroveDebt > _LUSDInStabPool))
452: if (_LUSDInStabPool > 0) {
495: } else if (_collDecimals > LiquityMath.CR_CALCULATION_DECIMALS) {
551: require(totals.totalDebtInSequence > 0);
563: if (totals.totalCollSurplus > 0) {
754: require(totals.totalDebtInSequence > 0);
766: if (totals.totalCollSurplus > 0) {
916: if (_LUSD > 0) {
920: if (_collAmount > 0) {
1224: assert(totalStakesSnapshot[_collateral] > 0);
1539: require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L80 )

```solidity
80: if (wantBalance > _debt) {
99: if (_amountNeeded > liquidatedAmount) {
131: if (totalAssets > allocated) {
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L234 )

```solidity
234: if (stratCurrentAllocation > stratMaxAllocation) {
368: if (value > token.balanceOf(address(this))) {
400: if (value > vaultBalance) {
502: } else if (_roi > 0) {
518: } else if (vars.available > 0) {
525: if (vars.credit > vars.freeWantInStrat) {
534: if (vars.lockedProfitBeforeLoss > vars.loss) {
```

File: [2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L122 )

```solidity
122: } else if (amountFreed > debt) {
```

### <a id=G9>[G-9]</a> `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

##### Description

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

##### Recommendation

Use `<x> = <x> + <y>` instead.

##### *Instances (22):*

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L133 )

```solidity
133: toFree += profit;
141: roi -= int256(loss);
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168 )

```solidity
168: totalAllocBPS += _allocBPS;
194: totalAllocBPS -= strategies[_strategy].allocBPS;
196: totalAllocBPS += _allocBPS;
214: totalAllocBPS -= strategies[_strategy].allocBPS;
390: value -= loss;
391: totalLoss += loss;
395: strategies[stratAddr].allocated -= actualWithdrawn;
396: totalAllocated -= actualWithdrawn;
444: stratParams.allocBPS -= bpsChange;
445: totalAllocBPS -= bpsChange;
450: stratParams.losses += loss;
451: stratParams.allocated -= loss;
452: totalAllocated -= loss;
505: strategy.gains += vars.gain;
514: strategy.allocated -= vars.debtPayment;
515: totalAllocated -= vars.debtPayment;
516: vars.debt -= vars.debtPayment; // tracked for return value
520: strategy.allocated += vars.credit;
521: totalAllocated += vars.credit;
```

File: [2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L128 )

```solidity
128: repayment -= uint256(-roi);
```

### <a id=G10>[G-10]</a> Import only what you need.

##### Description

Importing only what is needed is ideal for gas optimization.

##### Recommendation

Import only what is necessary from file.

##### *Instances (102):*

File: [2023-02-ethos/Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L6 )

```solidity
6: import "./Interfaces/ICollateralConfig.sol";
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

File: [2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L5 )

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

File: [2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L5 )

```solidity
5: import "./Dependencies/CheckContract.sol";
6: import "./Dependencies/Ownable.sol";
7: import "./Dependencies/SafeERC20.sol";
8: import "./Interfaces/ICollateralConfig.sol";
```

File: [2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L5 )

```solidity
5: import "../Dependencies/IERC20.sol";
6: import "../Interfaces/ICommunityIssuance.sol";
7: import "../Dependencies/BaseMath.sol";
8: import "../Dependencies/LiquityMath.sol";
9: import "../Dependencies/Ownable.sol";
10: import "../Dependencies/CheckContract.sol";
11: import "../Dependencies/SafeMath.sol";
```

File: [2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L5 )

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

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L5 )

```solidity
5: import "./Interfaces/ILUSDToken.sol";
6: import "./Interfaces/ITroveManager.sol";
7: import "./Dependencies/SafeMath.sol";
8: import "./Dependencies/CheckContract.sol";
9: import "./Dependencies/console.sol";
```

File: [2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L6 )

```solidity
6: import "./Interfaces/ICollateralConfig.sol";
12: import "./Interfaces/ICommunityIssuance.sol";
13: import "./Dependencies/LiquityBase.sol";
14: import "./Dependencies/SafeMath.sol";
15: import "./Dependencies/LiquitySafeMath128.sol";
16: import "./Dependencies/Ownable.sol";
17: import "./Dependencies/CheckContract.sol";
18: import "./Dependencies/console.sol";
19: import "./Dependencies/SafeERC20.sol";
```

File: [2023-02-ethos/Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L5 )

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
14: // import "./Dependencies/Ownable.sol";
15: import "./Dependencies/CheckContract.sol";
16: import "./Dependencies/IERC20.sol";
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L5 )

```solidity
5: import "./abstract/ReaperBaseStrategyv4.sol";
6: import "./interfaces/IAToken.sol";
7: import "./interfaces/IAaveProtocolDataProvider.sol";
8: import "./interfaces/ILendingPool.sol";
9: import "./interfaces/ILendingPoolAddressesProvider.sol";
10: import "./interfaces/IRewardsController.sol";
11: import "./libraries/ReaperMathUtils.sol";
12: import "./mixins/VeloSolidMixin.sol";
13: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
14: import "@openzeppelin/contracts-upgradeable/utils/math/MathUpgradeable.sol";
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L5 )

```solidity
5: import "./ReaperVaultV2.sol";
6: import "./interfaces/IERC4626Functions.sol";
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L5 )

```solidity
5: import "./interfaces/IERC4626Events.sol";
6: import "./interfaces/IStrategy.sol";
7: import "./libraries/ReaperMathUtils.sol";
8: import "./mixins/ReaperAccessControl.sol";
9: import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";
10: import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
11: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
12: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
13: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
14: import "@openzeppelin/contracts/utils/math/Math.sol";
```

File: [2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L5 )

```solidity
5: import "../interfaces/IStrategy.sol";
6: import "../interfaces/IVault.sol";
7: import "../libraries/ReaperMathUtils.sol";
8: import "../mixins/ReaperAccessControl.sol";
9: import "@openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";
10: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
11: import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
12: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
```

### <a id=G11>[G-11]</a> Using fixed bytes is cheaper than using `string`.

##### Description

As a rule of thumb, use bytes for arbitrary-length raw byte data and `string` for arbitrary-length string (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of `bytes1` to `bytes32` because they are much cheaper..

##### Recommendation

Use fixed bytes instead of `string`.

##### *Instances (7):*

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L399 )

```solidity
399: function name() external view override returns (string memory) {
403: function symbol() external view override returns (string memory) {
411: function version() external view override returns (string memory) {
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L18 )

```solidity
18: string memory _name,
19: string memory _symbol,
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L113 )

```solidity
113: string memory _name,
114: string memory _symbol,
```

### <a id=G13>[G-12]</a> Use assembly to check for address(0)

##### Description

You can save about 6 gas per instance if using assembly to check for address(0)

##### Recommendation

Use assembly.

##### *Instances (4):*

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167 )

```solidity
167: require(step[0] != address(0));
168: require(step[1] != address(0));
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L151 )

```solidity
151: require(_strategy != address(0), "Invalid strategy address");
629: require(newTreasury != address(0), "Invalid address");
```
