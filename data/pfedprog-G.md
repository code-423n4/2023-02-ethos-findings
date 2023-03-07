# Report

## Don't Initialize Variables with Default Value

Uninitialized variables are assigned with the types default value.

Explicitly initializing a variable with it's default value costs unnecessary gas.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L369 => uint256 totalLoss = 0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L371 => uint256 vaultBalance = 0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L100 => uint256 amountFreed = 0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L112 => uint256 debt = 0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L117 => uint256 repayment = 0;

## Cache Array Length Outside of Loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L106 => uint256 numCollaterals = collaterals.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L107 => require(numCollaterals == erc4626vaults.length, "Vaults array length must match number of collaterals");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L52 => require(collaterals.length != 0, "At least one collateral required");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L53 => require(MCRs.length == collaterals.length, "Array lengths must match");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L54 => require(CCRs.length == collaterals.length, "Array lenghts must match");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56 => for(uint256 i = 0; i < collaterals.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/Address.sol#L152 => if (returndata.length > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/SafeERC20.sol#L41 => // solhint-disable-next-line max-line-length
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/SafeERC20.sol#L70 => if (returndata.length > 0) { // Return data is optional
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/SafeERC20.sol#L71 => // solhint-disable-next-line max-line-length
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/UsingTellor.sol#L359 => for (uint256 i = 0; i < b.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L157 => be <= sqrt(length) positions away from the correct insert position.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L205 => amounts = new uint[](assets.length);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206 => for (uint i = 0; i < assets.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L227 => uint[] memory amounts = new uint[](collaterals.length);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228 => for (uint i = 0; i < collaterals.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L239 => uint numCollaterals = assets.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L105 => uint256 numCollaterals = collaterals.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L107 => require(priceAggregatorAddresses.length == numCollaterals, "Array lengths must match");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L108 => require(tellorQueryIds.length == numCollaterals, "Array lengths must match");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L350 => uint numCollaterals = assets.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L396 => uint numCollaterals = assets.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L639 => amounts = new uint[](assets.length);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640 => for (uint i = 0; i < assets.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L807 => uint[] memory amounts = new uint[](collaterals.length);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810 => for (uint i = 0; i < collaterals.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831 => for (uint i = 0; i < collaterals.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L858 => uint numCollaterals = collaterals.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/CDPManagerTester.sol#L57 => uint troveOwnersArrayLength = TroveOwners[_collateral].length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L300 => return TroveOwners[_collateral].length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L372 => if (TroveOwners[_collateral].length <= 1) {return singleLiquidation;} // don't liquidate if last trove
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L716 => require(troveArray.length != 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808 => for (vars.i = 0; vars.i < troveArray.length; vars.i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882 => for (vars.i = 0; vars.i < troveArray.length; vars.i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1281 => uint TroveOwnersArrayLength = TroveOwners[_collateral].length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1329 => index = uint128(TroveOwners[_collateral].length.sub(1));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1345 => uint length = TroveOwnersArrayLength;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1346 => uint idxLast = length.sub(1);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L116 => uint256 numSteps = steps.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L164 => uint256 numSteps = newSteps.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L127 => uint256 numStrategists = strategists.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L260 => uint256 queueLength = withdrawalQueue.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L370 => uint256 queueLength = withdrawalQueue.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L76 => uint256 numStrategists = strategists.length;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L81 => require(multisigRoles.length == 3, "Invalid number of multisig roles");

## Use != 0 instead of > 0 for Unsigned Integer Comparison

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L278 => if (vars.netAssetMovement > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L128 => assert(MIN_NET_DEBT > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197 => assert(vars.compositeDebt > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L301 => assert(msg.sender == borrower || (msg.sender == stabilityPoolAddress && collTopUp > 0 && LUSDChange == 0));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L337 => if (!isDebtIncrease && LUSDChange > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L552 => require(LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollSurplusPool.sol#L97 => require(claimableColl > 0, "CollSurplusPool: No collateral available to claim");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/Address.sol#L34 => return size > 0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/Address.sol#L152 => if (returndata.length > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/CheckContract.sol#L17 => require(size > 0, "Account code size cannot be zero");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/LiquityMath.sol#L98 => if (debt > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/LiquityMath.sol#L109 => if (debt > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/SafeERC20.sol#L70 => if (returndata.length > 0) { // Return data is optional
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/SafeMath.sol#L122 => require(b > 0, errorMessage);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L120 => while (vars.currentTroveuser != address(0) && vars.remainingLUSD > 0 && maxIterations-- > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L152 => if (LQTYamount > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L181 => if (totalLQTYStaked > 0) {collFeePerLQTYStaked = collFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L191 => if (totalLQTYStaked > 0) {LUSDFeePerLQTYStaked = LUSDFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L266 => require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L270 => require(amount > 0, 'LQTYStaking: Amount must be non-zero');
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L279 => if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerWrappersScript.sol#L105 => if (claimedCollateral > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerWrappersScript.sol#L111 => if (LUSDAmount > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerWrappersScript.sol#L118 => if (claimedLQTY > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerWrappersScript.sol#L137 => if (gainedCollateral > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerWrappersScript.sol#L145 => if (totalLUSD > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerWrappersScript.sol#L151 => if (claimedLQTY > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L153 => while (totals.currentBorrower != address(0) && totals.remainingLUSD > 0 && maxIterations > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L179 => require(totals.totalCollateralDrawn > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L317 => require(amount > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L115 => require(NICR > 0, "SortedTroves: NICR must be positive");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L216 => require(newNICR > 0, "SortedTroves: NICR must be positive");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L591 => assert(newP > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L870 => require(initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L874 => require(amount > 0, 'StabilityPool: Amount must be non-zero');
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L129 => require(MCR > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L130 => require(CCR > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L169 => require(price > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L204 => assert(numberOfTroves > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L328 => if (IERC20(collaterals[0]).balanceOf(address(activePool)) > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L355 => \* Stake > 0
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L374 => // Stake > 0
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L387 => if (IERC20(collaterals[0]).balanceOf(address(troveManager)) > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L391 => if (IERC20(collaterals[0]).balanceOf(address(borrowerOperations)) > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L407 => if (IERC20(collaterals[0]).balanceOf(address(lusdToken)) > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L411 => if (IERC20(collaterals[0]).balanceOf(address(priceFeedTestnet)) > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/EchidnaTester.sol#L415 => if (IERC20(collaterals[0]).balanceOf(address(sortedTroves)) > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L424 => if (singleLiquidation.collSurplus > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L452 => if (LUSDInStabPool > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L551 => require(totals.totalDebtInSequence > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L563 => if (totals.totalCollSurplus > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754 => require(totals.totalDebtInSequence > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L766 => if (totals.totalCollSurplus > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L916 => if (LUSD > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L920 => if (collAmount > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1224 => assert(totalStakesSnapshot[_collateral] > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1414 => assert(newBaseRate > 0); // Base rate is always non-zero after redemption
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L502 => } else if (roi > 0) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L518 => } else if (vars.available > 0) {

## Use immutable for OpenZeppelin AccessControl's Roles Declarations

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L178 => latestRandomSeed = uint(keccak256(abi.encodePacked(latestRandomSeed)));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L41 => // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L43 => // keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L120 => bytes32 hashedName = keccak256(bytes(NAME));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L121 => bytes32 hashedVersion = keccak256(bytes(VERSION));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L283 => bytes32 digest = keccak256(abi.encodePacked('\x19\x01',
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L284 => domainSeparator(), keccak256(abi.encode(
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L305 => return keccak256(abi.encode(typeHash, name, version, chainID(), address(this)));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/DappSys/proxy.sol#L205 => bytes32 hash = keccak256(code);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/DappSys/proxy.sol#L218 => bytes32 hash = keccak256(code);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/LUSDTokenTester.sol#L59 => return keccak256(abi.encodePacked(
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TestContracts/LUSDTokenTester.sol#L62 => keccak256(abi.encode(PERMIT_TYPEHASH, owner, spender, amount, nonce, deadline))
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73 => bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74 => bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L75 => bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L76 => bytes32 public constant ADMIN = keccak256("ADMIN");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49 => bytes32 public constant KEEPER = keccak256("KEEPER");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50 => bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51 => bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52 => bytes32 public constant ADMIN = keccak256("ADMIN");

## Long Revert Strings

Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L107 => require(numCollaterals == erc4626vaults.length, "Vaults array length must match number of collaterals");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L133 => require(driftBps <= 500, "Exceeds max allowed value of 500 BPS");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L324 => "ActivePool: Caller is neither BO nor Default Pool");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L334 => "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L341 => "ActivePool: Caller is neither BorrowerOperations nor TroveManager");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L5 => import "./Interfaces/IBorrowerOperations.sol";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L6 => import "./Interfaces/ICollateralConfig.sol";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L9 => import "./Interfaces/ICollSurplusPool.sol";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L525 => require(collateralConfig.isCollateralAllowed(collateral), "BorrowerOps: Invalid collateral address");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L529 => require(IERC20(collateral).balanceOf(user) >= collAmount, "BorrowerOperations: Insufficient user collateral balance");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L530 => require(IERC20(collateral).allowance(user, address(this)) >= collAmount, "BorrowerOperations: Insufficient collateral allowance");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L534 => require(collTopUp == 0 || collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L538 => require(collTopUp != 0 || collWithdrawal != 0 || LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L543 => require(status == 1, "BorrowerOps: Trove does not exist or is closed");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L552 => require(LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L563 => "BorrowerOps: Operation not permitted during Recovery Mode"
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L568 => require(collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L617 => require(newICR >= MCR, "BorrowerOps: An operation that would result in ICR < MCR is not permitted");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L621 => require(newICR >= CCR, "BorrowerOps: Operation must leave trove with ICR >= CCR");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L625 => require(newICR >= oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L629 => require(newTCR >= CCR, "BorrowerOps: An operation that would result in TCR < CCR is not permitted");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L633 => require (netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L637 => require(debtRepayment <= currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L641 => require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L645 => require(lusdToken.balanceOf(borrower) >= debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L651 => "Max fee percentage must less than or equal to 100%");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L654 => "Max fee percentage must be between 0.5% and 100%");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollSurplusPool.sol#L97 => require(claimableColl > 0, "CollSurplusPool: No collateral available to claim");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollSurplusPool.sol#L128 => "CollSurplusPool: Caller is not Borrower Operations");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollSurplusPool.sol#L134 => "CollSurplusPool: Caller is not TroveManager");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollSurplusPool.sol#L140 => "CollSurplusPool: Caller is not Active Pool");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/DefaultPool.sol#L127 => require(msg.sender == activePoolAddress, "DefaultPool: Caller is not the ActivePool");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/DefaultPool.sol#L131 => require(msg.sender == troveManagerAddress, "DefaultPool: Caller is not the TroveManager");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/Address.sol#L58 => require(success, "Address: unable to send value, recipient may have reverted");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/Address.sol#L105 => return functionCallWithValue(target, data, value, "Address: low-level call with value failed");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/Address.sol#L115 => require(address(this).balance >= value, "Address: insufficient balance for call");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/Address.sol#L130 => return functionStaticCall(target, data, "Address: low-level static call failed");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/Address.sol#L140 => require(isContract(target), "Address: static call to non-contract");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/LiquitySafeMath128.sol#L10 => require(c >= a, "LiquitySafeMath128: addition overflow");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/LiquitySafeMath128.sol#L16 => require(b <= a, "LiquitySafeMath128: subtraction overflow");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/SafeERC20.sol#L43 => "SafeERC20: approve from non-zero to non-zero allowance"
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/SafeERC20.sol#L54 => uint256 newAllowance = token.allowance(address(this), spender).sub(value, "SafeERC20: decreased allowance below zero");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/SafeERC20.sol#L72 => require(abi.decode(returndata, (bool)), "SafeERC20: ERC20 operation did not succeed");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/SafeMath.sol#L87 => require(c / a == b, "SafeMath: multiplication overflow");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L133 => require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L257 => "LQTYStaking: caller is not TroveM or ActivePool"
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L262 => require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L41 => // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L43 => // keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L136 => "LUSD: Caller is not guardian or governance"
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L238 => approve(sender, msg.sender, allowances[sender][msg.sender].sub(amount, "ERC20: transfer amount exceeds allowance"));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L248 => approve(msg.sender, spender, allowances[msg.sender][spender].sub(subtractedValue, "ERC20: decreased allowance below zero"));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L315 => balances[sender] = balances[sender].sub(amount, "ERC20: transfer amount exceeds balance");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L331 => balances[account] = balances[account].sub(amount, "ERC20: burn amount exceeds balance");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L350 => "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L356 => "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L362 => require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L371 => "LUSD: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool"
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L377 => require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L386 => "LUSD: Caller is neither TroveManager nor StabilityPool");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L390 => require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L394 => require(!mintingPaused, "LUSDToken: Minting is currently paused");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L132 => "PriceFeed: Chainlink must be working and current");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L159 => "PriceFeed: Chainlink must be working and current");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerWrappersScript.sol#L70 => require(lqtyStakingAddress == address(lqtyStakingCached), "BorrowerWrappersScript: Wrong LQTYStaking address");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerWrappersScript.sol#L172 => require(troveManager.getTroveStatus(depositor, collateral) == 1, "BorrowerWrappersScript: caller must have an active trove");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L296 => require(msg.sender == address(troveManager), "RedemptionHelper: Caller is not TroveManager");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L111 => require(!contains(collateral, id), "SortedTroves: List already contains the node");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L115 => require(NICR > 0, "SortedTroves: NICR must be positive");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L166 => require(contains(collateral, id), "SortedTroves: List does not contain the id");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L214 => require(contains(collateral, id), "SortedTroves: List does not contain the id");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L216 => require(newNICR > 0, "SortedTroves: NICR must be positive");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L403 => require(msg.sender == address(troveManager), "SortedTroves: Caller is not the TroveManager");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L408 => "SortedTroves: Caller is neither BO nor TroveM");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L849 => require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L853 => require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L865 => require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L98 => require(amount <= balanceOf(), "Ammount must be less than balance");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L193 => "Upgrade cooldown not initiated or still ongoing"

## Use Shift Right/Left instead of Division/Multiplication

A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L397 => troveManagerCached.closeTrove(msg.sender, collateral, 2); // 2 = closedByOwner
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/LiquityMath.sol#L47 => decProd = prod_xy.add(DECIMAL_PRECISION / 2).div(DECIMAL_PRECISION);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/UsingTellor.sol#L102 => middle = (end + start) / 2;

## FOR-LOOPS: ++I COSTS LESS GAS COMPARED TO I++
++i costs less gas compared to i++ for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration)

i++ increments i and returns the initial value of i. Which means:

uint i = 1;  
i++; // == 1 but i == 2  
But ++i returns the actual incremented value:

uint i = 1;  
++i; // == 2 and i == 2 too, so no need for a temporary variable  
In the first case, the compiler has to create a temporary variable (when used) for returning 1 instead of 2

Instances include:


https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L351
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L397
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L640
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L810
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L831
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L859