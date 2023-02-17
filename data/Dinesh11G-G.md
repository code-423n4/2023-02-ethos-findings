### 1st BUG
Don't Initialize Variables with Default Value

#### Impact
Issue Information: 
Uninitialized variables are assigned with the types default value.

Explicitly initializing a variable with it's default value costs unnecessary gas.

Example
🤦 Bad:

uint256 x = 0;
bool y = false;
🚀 Good:

uint256 x;
bool y;

#### Findings:
```
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::72 => for (uint idx = 0; idx < _startIdx; ++idx) {
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::78 => for (uint idx = 0; idx < _count; ++idx) {
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::101 => for (uint idx = 0; idx < _startIdx; ++idx) {
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::107 => for (uint idx = 0; idx < _count; ++idx) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::113 => for (uint256 i = 0; i < numCollaterals; i++) {
```
#### Tools used
Manual VS Code review

### 2nd BUG
Cache Array Length Outside of Loop

#### Impact
Issue Information: 
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

Example
🤦 Bad:

for (uint256 i = 0; i < array.length; i++) {
    // invariant: array's length is not changed
}
🚀 Good:

uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // invariant: array's length is not changed
}

#### Findings:
```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::106 => uint256 numCollaterals = collaterals.length;
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::107 => require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::52 => require(_collaterals.length != 0, "At least one collateral required");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::53 => require(_MCRs.length == _collaterals.length, "Array lengths must match");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::54 => require(_CCRs.length == _collaterals.length, "Array lenghts must match");
2023-02-ethos/Ethos-Core/contracts/Dependencies/Address.sol::152 => if (returndata.length > 0) {
2023-02-ethos/Ethos-Core/contracts/Dependencies/SafeERC20.sol::70 => if (returndata.length > 0) { // Return data is optional
2023-02-ethos/Ethos-Core/contracts/Dependencies/UsingTellor.sol::359 => for (uint256 _i = 0; _i < _b.length; _i++) {
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::156 => Submitting numTrials = k * sqrt(length), with k = 15 makes it very, very likely that the ouput address will
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::157 => be <= sqrt(length) positions away from the correct insert position.
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::205 => amounts = new uint[](assets.length);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::227 => uint[] memory amounts = new uint[](collaterals.length);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::239 => uint numCollaterals = assets.length;
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::105 => uint256 numCollaterals = collaterals.length;
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::107 => require(_priceAggregatorAddresses.length == numCollaterals, "Array lengths must match");
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::108 => require(_tellorQueryIds.length == numCollaterals, "Array lengths must match");
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::350 => uint numCollaterals = assets.length;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::396 => uint numCollaterals = assets.length;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::639 => amounts = new uint[](assets.length);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::807 => uint[] memory amounts = new uint[](collaterals.length);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::858 => uint numCollaterals = collaterals.length;
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::300 => return TroveOwners[_collateral].length;
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::372 => if (TroveOwners[_collateral].length <= 1) {return singleLiquidation;} // don't liquidate if last trove
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::716 => require(_troveArray.length != 0);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1281 => uint TroveOwnersArrayLength = TroveOwners[_collateral].length;
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1329 => index = uint128(TroveOwners[_collateral].length.sub(1));
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1345 => uint length = TroveOwnersArrayLength;
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1346 => uint idxLast = length.sub(1);
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::116 => uint256 numSteps = steps.length;
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::164 => uint256 numSteps = _newSteps.length;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::127 => uint256 numStrategists = _strategists.length;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::260 => uint256 queueLength = _withdrawalQueue.length;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::370 => uint256 queueLength = withdrawalQueue.length;
2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::76 => uint256 numStrategists = _strategists.length;
2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::81 => require(_multisigRoles.length == 3, "Invalid number of multisig roles");
```
#### Tools used
Manual VS Code review

### 3rd BUG
Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Impact
Issue Information: 
When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.

Example
🤦 Bad:

// `a` being of type unsigned integer
require(a > 0, "!a > 0");
🚀 Good:

// `a` being of type unsigned integer
require(a != 0, "!a > 0");

#### Findings:
```
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::97 => require(claimableColl > 0, "CollSurplusPool: No collateral available to claim");
2023-02-ethos/Ethos-Core/contracts/Dependencies/Address.sol::34 => return size > 0;
2023-02-ethos/Ethos-Core/contracts/Dependencies/Address.sol::152 => if (returndata.length > 0) {
2023-02-ethos/Ethos-Core/contracts/Dependencies/CheckContract.sol::17 => require(size > 0, "Account code size cannot be zero");
2023-02-ethos/Ethos-Core/contracts/Dependencies/LiquityMath.sol::98 => if (_debt > 0) {
2023-02-ethos/Ethos-Core/contracts/Dependencies/LiquityMath.sol::109 => if (_debt > 0) {
2023-02-ethos/Ethos-Core/contracts/Dependencies/SafeERC20.sol::70 => if (returndata.length > 0) { // Return data is optional
2023-02-ethos/Ethos-Core/contracts/Dependencies/SafeMath.sol::122 => require(b > 0, errorMessage);
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::120 => while (vars.currentTroveuser != address(0) && vars.remainingLUSD > 0 && _maxIterations-- > 0) {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::153 => while (totals.currentBorrower != address(0) && totals.remainingLUSD > 0 && _maxIterations > 0) {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::179 => require(totals.totalCollateralDrawn > 0);
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::317 => require(_amount > 0);
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::115 => require(_NICR > 0, "SortedTroves: NICR must be positive");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::216 => require(_newNICR > 0, "SortedTroves: NICR must be positive");
```
#### Tools used
Manual VS Code review

### 4th BUG
Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: 
⚡️ Only valid for solidity versions <0.6.12 ⚡️

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.

Example
🤦 Bad:

bytes32 public constant GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
🚀 Good:

bytes32 public immutable GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");

#### Findings:
```
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::178 => latestRandomSeed = uint(keccak256(abi.encodePacked(latestRandomSeed)));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::120 => bytes32 hashedName = keccak256(bytes(_NAME));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::121 => bytes32 hashedVersion = keccak256(bytes(_VERSION));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::284 => domainSeparator(), keccak256(abi.encode(
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::305 => return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::73 => bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::74 => bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::75 => bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::76 => bytes32 public constant ADMIN = keccak256("ADMIN");
2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::49 => bytes32 public constant KEEPER = keccak256("KEEPER");
2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::50 => bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::51 => bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::52 => bytes32 public constant ADMIN = keccak256("ADMIN");
```
#### Tools used
Manual VS Code review

### 5th BUG
Long Revert Strings

#### Impact
Issue Information: 
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

Example
🤦 Bad:

require(condition, "UniswapV3: The reentrancy guard. A transaction cannot re-enter the pool mid-swap");
🚀 Good (with shorter string):

// TODO: Provide link to a reference of error codes
require(condition, "LOK");
🚀 Good (with custom errors):

/// @notice A transaction cannot re-enter the pool mid-swap.
error NoReentrancy();

// ...

if (!condition) {
    revert NoReentrancy();
}

#### Findings:
```
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::97 => require(claimableColl > 0, "CollSurplusPool: No collateral available to claim");
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::127 => require(msg.sender == activePoolAddress, "DefaultPool: Caller is not the ActivePool");
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::131 => require(msg.sender == troveManagerAddress, "DefaultPool: Caller is not the TroveManager");
2023-02-ethos/Ethos-Core/contracts/Dependencies/Address.sol::58 => require(success, "Address: unable to send value, recipient may have reverted");
2023-02-ethos/Ethos-Core/contracts/Dependencies/Address.sol::105 => return functionCallWithValue(target, data, value, "Address: low-level call with value failed");
2023-02-ethos/Ethos-Core/contracts/Dependencies/Address.sol::115 => require(address(this).balance >= value, "Address: insufficient balance for call");
2023-02-ethos/Ethos-Core/contracts/Dependencies/Address.sol::130 => return functionStaticCall(target, data, "Address: low-level static call failed");
2023-02-ethos/Ethos-Core/contracts/Dependencies/Address.sol::140 => require(isContract(target), "Address: static call to non-contract");
2023-02-ethos/Ethos-Core/contracts/Dependencies/LiquitySafeMath128.sol::10 => require(c >= a, "LiquitySafeMath128: addition overflow");
2023-02-ethos/Ethos-Core/contracts/Dependencies/LiquitySafeMath128.sol::16 => require(b <= a, "LiquitySafeMath128: subtraction overflow");
2023-02-ethos/Ethos-Core/contracts/Dependencies/SafeERC20.sol::43 => "SafeERC20: approve from non-zero to non-zero allowance"
2023-02-ethos/Ethos-Core/contracts/Dependencies/SafeERC20.sol::54 => uint256 newAllowance = token.allowance(address(this), spender).sub(value, "SafeERC20: decreased allowance below zero");
2023-02-ethos/Ethos-Core/contracts/Dependencies/SafeERC20.sol::72 => require(abi.decode(returndata, (bool)), "SafeERC20: ERC20 operation did not succeed");
2023-02-ethos/Ethos-Core/contracts/Dependencies/SafeMath.sol::87 => require(c / a == b, "SafeMath: multiplication overflow");
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::943 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(uint,address,address,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1063 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(string,string,string,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1093 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(string,string,address,string)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1103 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(string,string,address,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1213 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(string,address,string,string)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1223 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(string,address,string,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1253 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(string,address,address,string)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1263 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(string,address,address,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1583 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(bool,address,address,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1663 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,uint,address,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1693 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,string,string,string)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1703 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,string,string,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1733 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,string,address,string)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1743 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,string,address,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1823 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,bool,address,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1843 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,address,uint,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1853 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,address,string,string)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1863 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,address,string,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1883 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,address,bool,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1888 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,address,address,uint)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1893 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,address,address,string)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1898 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,address,address,bool)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/Dependencies/console.sol::1903 => (bool ignored, ) = CONSOLE_ADDRESS.staticcall(abi.encodeWithSignature("log(address,address,address,address)", p0, p1, p2, p3));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::238 => _approve(sender, msg.sender, _allowances[sender][msg.sender].sub(amount, "ERC20: transfer amount exceeds allowance"));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::248 => _approve(msg.sender, spender, _allowances[msg.sender][spender].sub(subtractedValue, "ERC20: decreased allowance below zero"));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::315 => _balances[sender] = _balances[sender].sub(amount, "ERC20: transfer amount exceeds balance");
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::331 => _balances[account] = _balances[account].sub(amount, "ERC20: burn amount exceeds balance");
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::296 => require(msg.sender == address(troveManager), "RedemptionHelper: Caller is not TroveManager");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::111 => require(!contains(_collateral, _id), "SortedTroves: List already contains the node");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::115 => require(_NICR > 0, "SortedTroves: NICR must be positive");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::166 => require(contains(_collateral, _id), "SortedTroves: List does not contain the id");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::214 => require(contains(_collateral, _id), "SortedTroves: List does not contain the id");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::216 => require(_newNICR > 0, "SortedTroves: NICR must be positive");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::403 => require(msg.sender == address(troveManager), "SortedTroves: Caller is not the TroveManager");
```
#### Tools used
Manual VS Code review

### 6th BUG
Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: 
A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

Example
🤦 Bad:

uint256 b = a / 2;
uint256 c = a / 4;
uint256 d = a * 8;
🚀 Good:

uint256 b = a >> 1;
uint256 c = a >> 2;
uint256 d = a << 3;

#### Findings:
```
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::397 => troveManagerCached.closeTrove(msg.sender, _collateral, 2); // 2 = closedByOwner
2023-02-ethos/Ethos-Core/contracts/Dependencies/LiquityMath.sol::47 => decProd = prod_xy.add(DECIMAL_PRECISION / 2).div(DECIMAL_PRECISION);
2023-02-ethos/Ethos-Core/contracts/Dependencies/LiquityMath.sol::104 => return 2**256 - 1;
2023-02-ethos/Ethos-Core/contracts/Dependencies/LiquityMath.sol::117 => return 2**256 - 1;
2023-02-ethos/Ethos-Core/contracts/Dependencies/UsingTellor.sol::102 => _middle = (_end + _start) / 2;
2023-02-ethos/Ethos-Core/contracts/Dependencies/UsingTellor.sol::360 => _number = _number * 256 + uint8(_b[_i]);
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::44 => uint constant public TIMEOUT = 14400;  // 4 hours: 60 * 60 * 4
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::593 => uint256 /* startedAt */,
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::621 => uint256 /* startedAt */,
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::266 => troveManager.closeTrove(_borrower, _collateral, 4); // 4 = closedByRedemption
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::125 => lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```
#### Tools used
Manual VS Code review