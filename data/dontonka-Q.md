Some reported issues here are just added as a comment, but most have been applied the suggested corrections already, you can see all the final diffs. Take note that the code compile and all test pass with those too.

```diff
diff --git a/Ethos-Core/contracts/ActivePool.sol b/Ethos-Core/contracts/ActivePool.sol
index 771da52..4e58ef4 100644
--- a/Ethos-Core/contracts/ActivePool.sol
+++ b/Ethos-Core/contracts/ActivePool.sol
@@ -83,6 +83,7 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
         onlyOwner
     {
         require(!addressesSet, "Can call setAddresses only once");
+        require(_treasuryAddress != address(0), "Treasury cannot be 0 address"); //@audit (QA) moved beginning of function so its cleaner.
 
         checkContract(_collateralConfigAddress);
         checkContract(_borrowerOperationsAddress);
@@ -90,7 +91,6 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
         checkContract(_stabilityPoolAddress);
         checkContract(_defaultPoolAddress);
         checkContract(_collSurplusPoolAddress);
-        require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
         checkContract(_lqtyStakingAddress);
 
         collateralConfigAddress = _collateralConfigAddress;
@@ -124,19 +124,20 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
 
     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
         _requireValidCollateralAddress(_collateral);
-        require(_bps <= 10_000, "Invalid BPS value");
+        require(_bps <= 10_000, "Invalid BPS value"); //@audit (QA) no lower limit verification. A value of 0 seems to be problematic it would turn off the yield.
         yieldingPercentage[_collateral] = _bps;
         emit YieldingPercentageUpdated(_collateral, _bps);
     }
 
     function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
-        require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
+        require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS"); //@audit (QA) no lower limit verification. A value of 0 seems to be problematic.
         yieldingPercentageDrift = _driftBps;
         emit YieldingPercentageDriftUpdated(_driftBps);
     }
 
     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
         _requireValidCollateralAddress(_collateral);
+        //@audit (QA) no validation on _threshold. A value too big could case no profit to be sent to SP/Staking. This can affect rebalancing not effective. The team should think this throught and add proper require.
         yieldClaimThreshold[_collateral] = _threshold;
         emit YieldClaimThresholdUpdated(_collateral, _threshold);
     }
@@ -171,6 +172,8 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
     function sendCollateral(address _collateral, address _account, uint _amount) external override {
         _requireValidCollateralAddress(_collateral);
         _requireCallerIsBOorTroveMorSP();
+        require (_account != address(this)); //@audit (QA) To be cautious, added a require so it cannot send to itself
+
         _rebalance(_collateral, _amount);
         collAmount[_collateral] = collAmount[_collateral].sub(_amount);
         emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);
@@ -192,14 +195,14 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
         _requireValidCollateralAddress(_collateral);
         _requireCallerIsBOorTroveM();
         LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount);
-        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
+        emit ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]); //@audit (QA) added emit
     }
 
     function decreaseLUSDDebt(address _collateral, uint _amount) external override {
         _requireValidCollateralAddress(_collateral);
         _requireCallerIsBOorTroveMorSP();
         LUSDDebt[_collateral] = LUSDDebt[_collateral].sub(_amount);
-        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
+        emit ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]); //@audit (QA) added emit
     }
 
     function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override {
@@ -212,6 +215,7 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
         _rebalance(_collateral, 0);
     }
 
+    //@audit (QA) this seems to be used only for testing purpose. Still, ff not stricly required would probably remove it from mainnet deployement.
     function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {
         _requireValidCollateralAddress(_collateral);
         _rebalance(_collateral, _simulatedAmountLeavingPool);
```

```diff
diff --git a/Ethos-Core/contracts/BorrowerOperations.sol b/Ethos-Core/contracts/BorrowerOperations.sol
index 6ee5c75..ecc605b 100644
--- a/Ethos-Core/contracts/BorrowerOperations.sol
+++ b/Ethos-Core/contracts/BorrowerOperations.sol
@@ -194,7 +194,7 @@ contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOpe
 
         // ICR is based on the composite debt, i.e. the requested LUSD amount + LUSD borrowing fee + LUSD gas comp.
         vars.compositeDebt = _getCompositeDebt(vars.netDebt);
-        assert(vars.compositeDebt > 0);
+        assert(vars.compositeDebt > 0); //@audit (QA) seems redundant, _requireAtLeastMinNetDebt already confirm this indirectly. What would be more useful is probably vars.compositeDebt > vars.netDebt. I would probably just remove this assert tbh to save gas, its dead code in the end.
         
         vars.ICR = LiquityMath._computeCR(_collAmount, vars.compositeDebt, vars.price, vars.collDecimals);
         vars.NICR = LiquityMath._computeNominalCR(_collAmount, vars.compositeDebt, vars.collDecimals);
```

```diff
diff --git a/Ethos-Core/contracts/CollSurplusPool.sol b/Ethos-Core/contracts/CollSurplusPool.sol
index 13a56ff..76588a5 100644
--- a/Ethos-Core/contracts/CollSurplusPool.sol
+++ b/Ethos-Core/contracts/CollSurplusPool.sol
@@ -68,6 +68,7 @@ contract CollSurplusPool is Ownable, CheckContract, ICollSurplusPool {
 
     /* Returns the collAmount state variable.
        Not necessarily equal to the raw collateral balance - collateral can be forcibly sent to contracts. */
+    //@audit (QA) this comment is not really valid anymore as not receiving ETH throught payable etc.
     function getCollateral(address _collateral) external view override returns (uint) {
         _requireValidCollateralAddress(_collateral);
         return collAmount[_collateral];
```

```diff
diff --git a/Ethos-Core/contracts/LQTY/CommunityIssuance.sol b/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
index c79adfe..39f7ac8 100644
--- a/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
+++ b/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
@@ -2,17 +2,18 @@
 
 pragma solidity 0.6.11;
 
 import "../Dependencies/IERC20.sol";
 import "../Interfaces/ICommunityIssuance.sol";
 import "../Dependencies/BaseMath.sol";
 import "../Dependencies/LiquityMath.sol";
 import "../Dependencies/Ownable.sol";
 import "../Dependencies/CheckContract.sol";
 import "../Dependencies/SafeMath.sol";
+import "../Dependencies/SafeERC20.sol";
 
 
 contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMath {
     using SafeMath for uint;
+    using SafeERC20 for IERC20; //@audit (QA) added the safe as other contracts, and updated the corresponding lines
 
     // --- Data ---
 
@@ -83,6 +84,7 @@ contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMa
     // @dev issues a set amount of Oath to the stability pool
     function issueOath() external override returns (uint issuance) {
         _requireCallerIsStabilityPool();
+
         if (lastIssuanceTimestamp < lastDistributionTime) {
             uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;
             uint256 timePassed = endTimestamp.sub(lastIssuanceTimestamp);
@@ -100,7 +102,7 @@ contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMa
     */
     function fund(uint amount) external onlyOwner {
         require(amount != 0, "cannot fund 0");
-        OathToken.transferFrom(msg.sender, address(this), amount);
+        OathToken.safeTransferFrom(msg.sender, address(this), amount);
 
         // roll any unissued OATH into new distribution
         if (lastIssuanceTimestamp < lastDistributionTime) {
@@ -109,7 +111,7 @@ contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMa
             amount = amount.add(notIssued);
         }
 
-        rewardPerSecond = amount.div(distributionPeriod);
+        rewardPerSecond = amount.div(distributionPeriod); //@audit (QA) what is the precision management here about amount VS distributionPeriod both being injected by the owner? There could be a precision loss here, I guess its fine, still reporting as FYI.
         lastDistributionTime = block.timestamp.add(distributionPeriod);
         lastIssuanceTimestamp = block.timestamp;
 
@@ -118,13 +120,13 @@ contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMa
 
     // Owner-only function to update the distribution period
     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
+        //@audit (QA) there should be a minimum validation here, seems like a 0 value would prevent from funding (always revert by 0 division)
         distributionPeriod = _newDistributionPeriod;
     }
 
     function sendOath(address _account, uint _OathAmount) external override {
         _requireCallerIsStabilityPool();
-
-        OathToken.transfer(_account, _OathAmount);
+        OathToken.safeTransfer(_account, _OathAmount);
     }
 
     // --- 'require' functions ---
```

```diff
diff --git a/Ethos-Core/contracts/LQTY/LQTYStaking.sol b/Ethos-Core/contracts/LQTY/LQTYStaking.sol
index 6206361..5f20a9b 100644
--- a/Ethos-Core/contracts/LQTY/LQTYStaking.sol
+++ b/Ethos-Core/contracts/LQTY/LQTYStaking.sol
@@ -91,7 +91,7 @@ contract LQTYStaking is ILQTYStaking, Ownable, CheckContract, BaseMath {
         collateralConfig = ICollateralConfig(_collateralConfigAddress);

         emit LQTYTokenAddressSet(_lqtyTokenAddress);
-        emit LQTYTokenAddressSet(_lusdTokenAddress);
+        emit LUSDTokenAddressSet(_lusdTokenAddress); //@audit (QA) sending the proper emit
         emit TroveManagerAddressSet(_troveManagerAddress);
         emit BorrowerOperationsAddressSet(_borrowerOperationsAddress);
         emit ActivePoolAddressSet(_activePoolAddress);
```

```diff
diff --git a/Ethos-Core/contracts/LUSDToken.sol b/Ethos-Core/contracts/LUSDToken.sol
index 9c5424d..b449091 100644
--- a/Ethos-Core/contracts/LUSDToken.sol
+++ b/Ethos-Core/contracts/LUSDToken.sol
@@ -318,7 +318,7 @@ contract LUSDToken is CheckContract, ILUSDToken {
     }
 
     function _mint(address account, uint256 amount) internal {
-        assert(account != address(0));
+        assert(account != address(0)); //@audit (QA) would add a require that amount != 0
 
         _totalSupply = _totalSupply.add(amount);
         _balances[account] = _balances[account].add(amount);
```

```diff
diff --git a/Ethos-Core/contracts/PriceFeed.sol b/Ethos-Core/contracts/PriceFeed.sol
index c6810a8..18d6c4e 100644
--- a/Ethos-Core/contracts/PriceFeed.sol
+++ b/Ethos-Core/contracts/PriceFeed.sol
@@ -169,6 +169,9 @@ contract PriceFeed is Ownable, CheckContract, BaseMath, IPriceFeed {
     ) external onlyOwner {
         checkContract(_tellorCallerAddress);
         tellorCaller = ITellorCaller(_tellorCallerAddress);
+
+        //@audit (QA) if the contract _tellorCallerAddress is a valid contract, BUT doesn't implement ITellorCaller interface, the call here will still succeed. In such case, it would let the Admin to think the tellor is good, while it is not.
+        //Recommendation: should try it out like in fetchPrice at least with _getCurrentTellorResponse etc., similar as you do above for chainlink, the interface is tested out fetching the price.
     }
 
     // --- Functions ---
@@ -507,8 +510,8 @@ contract PriceFeed is Ownable, CheckContract, BaseMath, IPriceFeed {
         * future changes.
         *
         */
-        uint price;
-        if (_answerDigits >= TARGET_DIGITS) {
+        uint price = _price; //@audit (QA) if decimals are the same, no need todo anything. code is also clearer that way ans similar pattern as _scaleTellorPriceByDigits.
+        if (_answerDigits > TARGET_DIGITS) {
             // Scale the returned price value down to Liquity's target precision
             price = _price.div(10 ** (_answerDigits - TARGET_DIGITS));
         }
```

```diff
diff --git a/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol b/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
index db020f2..3dc594e 100644
--- a/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
+++ b/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
@@ -10,6 +10,7 @@ import "@openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgrad
 import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
 import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
 import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
+import "@openzeppelin/contracts-upgradeable/utils/math/MathUpgradeable.sol";
 
 abstract contract ReaperBaseStrategyv4 is
     ReaperAccessControl,
@@ -109,12 +110,12 @@ abstract contract ReaperBaseStrategyv4 is
     function harvest() external override returns (int256 roi) {
         _atLeastRole(KEEPER);
         int256 availableCapital = IVault(vault).availableCapital();
-        uint256 debt = 0;
+        uint256 debt;
+        uint256 repayment;
         if (availableCapital < 0) {
             debt = uint256(-availableCapital);
         }
 
-        uint256 repayment = 0;
         if (emergencyExit) {
             uint256 amountFreed = _liquidateAllPositions();
             if (amountFreed < debt) {
@@ -123,10 +124,8 @@ abstract contract ReaperBaseStrategyv4 is
                 roi = int256(amountFreed - debt);
             }
 
-            repayment = debt;
-            if (roi < 0) {
-                repayment -= uint256(-roi);
-            }
+            //@audit (QA) easier to understand and more consice. Also same logic in _harvestCore from Granary which confirm the behavior.
+            repayment = MathUpgradeable.min(debt, amountFreed);
         } else {
             (roi, repayment) = _harvestCore(debt);
         }
```

```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..2be0b81 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -120,15 +120,17 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
         token = IERC20Metadata(_token);
         constructionTime = block.timestamp;
         lastReport = block.timestamp;
-        tvlCap = _tvlCap;
-        treasury = _treasury;
+        tvlCap = _tvlCap; //@audit (QA) probably a require to be sure we receive a proper value, at least > 0?
+        treasury = _treasury; //@audit (QA) probably a require to at least verify this is not address(0)?
         lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
 
+        //@audit (QA) would it make sense to have no strategist? if not maybe add a require statement
         uint256 numStrategists = _strategists.length;
         for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
             _grantRole(STRATEGIST, _strategists[i]);
         }
 
+        //@audit (QA) would add a require that _multisigRoles.length == 3, with the corresponding error message. actually also in ReaperBaseStrategyV4.
         _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
         _grantRole(DEFAULT_ADMIN_ROLE, _multisigRoles[0]);
         _grantRole(ADMIN, _multisigRoles[1]);
@@ -223,6 +225,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
      * the vault.
      */
     function availableCapital() public view returns (int256) {
+        //@audit (QA) add require for the sender: require(strategies[msg.sender].activation != 0, "Invalid strategy address");
         address stratAddr = msg.sender;
         if (totalAllocBPS == 0 || emergencyShutdown) {
             return -int256(strategies[stratAddr].allocated);
@@ -246,7 +249,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
             available = Math.min(available, token.balanceOf(address(this)));
 
             return int256(available);
-        } else {
+        } else { //@audit (QA) else is redundant
             return 0;
         }
     }
@@ -328,10 +331,15 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
         token.safeTransferFrom(msg.sender, address(this), _amount);
         uint256 balAfter = token.balanceOf(address(this));
         _amount = balAfter - balBefore;
+        //@audit (QA) do the following as it is in _chargeFees
+        //    uint256 supply = totalSupply();
+        //    uint256 shares = supply == 0 ? _amount : (_amount * supply) / _freeFunds();
+        //    _mint(_receiver, shares);
+
         if (totalSupply() == 0) {
             shares = _amount;
         } else {
-            shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"
+            shares = (_amount * totalSupply()) / freeFunds;
         }
         _mint(_receiver, shares);
         emit Deposit(msg.sender, _receiver, _amount, shares);
@@ -398,7 +406,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
 
             vaultBalance = token.balanceOf(address(this));
             if (value > vaultBalance) {
-                value = vaultBalance;
+                value = vaultBalance; //@audit (QA) should we emit an event whenever this happen?
             }
 
             require(
@@ -577,6 +585,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
      */
     function updateTvlCap(uint256 _newTvlCap) public {
         _atLeastRole(ADMIN);
+        //@audit (QA) would add require to be sure its not lower then the current balance?  require(_newTvlCap > balance(), "Lower than the current Vault size");
         tvlCap = _newTvlCap;
         emit TvlCapUpdated(tvlCap);
     }
@@ -639,6 +648,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
         require(_token != address(token), "!token");
 
         uint256 amount = IERC20Metadata(_token).balanceOf(address(this));
+        //@audit (QA) Add a require or return in case there is nothing to recover : require(amount != 0, "Token empty")
         IERC20Metadata(_token).safeTransfer(msg.sender, amount);
         emit InCaseTokensGetStuckCalled(_token, amount);
     }
@@ -656,7 +666,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
      *      Subclasses should override this to specify their unique roles arranged in the correct
      *      order, for example, [SUPER-ADMIN, ADMIN, GUARDIAN, STRATEGIST].
      */
-    function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {
+    function _cascadingAccessRoles() internal pure override returns (bytes32[] memory) { // @audit (QA) using pure instead of view
         bytes32[] memory cascadingAccessRoles = new bytes32[](5);
         cascadingAccessRoles[0] = DEFAULT_ADMIN_ROLE;
         cascadingAccessRoles[1] = ADMIN;
```

```diff
diff --git a/Ethos-Core/contracts/StabilityPool.sol b/Ethos-Core/contracts/StabilityPool.sol
index 5f8fac6..34dba94 100644
--- a/Ethos-Core/contracts/StabilityPool.sol
+++ b/Ethos-Core/contracts/StabilityPool.sol
@@ -157,7 +157,7 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {

     ILUSDToken public lusdToken;

-    address public lqtyTokenAddress;
+    address public lqtyTokenAddress; //@audit (QA) unused, to be removed

     // Needed to check if there are pending liquidations
     ISortedTroves public sortedTroves;
@@ -172,7 +172,7 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
    // --- Data structures ---
 
     struct Deposit {
-        uint initialValue;
+        uint initialValue; //@audit (QA) a little weird a struct for a single member. but i understand the reason. not a huge deal.
     }
 
     struct Snapshots {
@@ -376,6 +376,7 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
         (address[] memory assets, uint[] memory amounts) = getDepositorCollateralGain(msg.sender);
         uint compoundedLUSDDeposit = getCompoundedLUSDDeposit(msg.sender);
         uint LUSDtoWithdraw = LiquityMath._min(_amount, compoundedLUSDDeposit);
+        //@audit (QA) would probably add a require or return so its != 0
 
         /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
          * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
@@ -465,6 +466,7 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
     */
     function offset(address _collateral, uint _debtToOffset, uint _collToAdd) external override {
         _requireCallerIsTroveManager();
+        _requireValidCollateralAddress(_collateral); //@audit (QA) Added missing require for _collateral
         uint totalLUSD = totalLUSDDeposits; // cached to save an SLOAD
         if (totalLUSD == 0 || _debtToOffset == 0) { return; }
 
@@ -485,6 +487,8 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
     */
     function updateRewardSum(address _collateral, uint _collToAdd) external override {
         _requireCallerIsActivePool();
+        _requireValidCollateralAddress(_collateral); //@audit (QA) Added missing require for _collateral
+
         uint totalLUSD = totalLUSDDeposits; // cached to save an SLOAD
         if (totalLUSD == 0) { return; }
 
@@ -510,6 +514,8 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
         internal
         returns (uint collGainPerUnitStaked, uint LUSDLossPerUnitStaked)
     {
+        require(_debtToOffset <= _totalLUSDDeposits); //@audit (QA) require and at the beginning is cleaner and even save gas.
+
         /*
         * Compute the LUSD and collateral rewards. Uses a "feedback" error correction, to keep
         * the cumulative error in the P and S state variables low:
@@ -521,9 +527,6 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
         * 4) Store these errors for use in the next correction when this function is called.
         * 5) Note: static analysis tools complain about this "division before multiplication", however, it is intended.
         */
-        uint collNumerator = _collToAdd.mul(DECIMAL_PRECISION).add(lastCollateralError_Offset[_collateral]);
-
-        assert(_debtToOffset <= _totalLUSDDeposits);
         if (_debtToOffset == _totalLUSDDeposits) {
             LUSDLossPerUnitStaked = DECIMAL_PRECISION;  // When the Pool depletes to 0, so does each deposit 
             lastLUSDLossError_Offset = 0;
@@ -537,6 +540,8 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
             lastLUSDLossError_Offset = (LUSDLossPerUnitStaked.mul(_totalLUSDDeposits)).sub(LUSDLossNumerator);
         }
 
+        //@audit (QA) moved this line down so its grouped where it is used.
+        uint collNumerator = _collToAdd.mul(DECIMAL_PRECISION).add(lastCollateralError_Offset[_collateral]);
         collGainPerUnitStaked = collNumerator.div(_totalLUSDDeposits);
         lastCollateralError_Offset[_collateral] = collNumerator.sub(collGainPerUnitStaked.mul(_totalLUSDDeposits));
 
@@ -661,7 +666,7 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
         * If the gain spans no scale change, the second portion will be 0.
         */
         LocalVariables_getSingularCollateralGain memory vars;
-        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);
+        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); //@audit (QA) vars.collDecimals is not used, expected?
         vars.epochSnapshot = _snapshots.epoch;
         vars.scaleSnapshot = _snapshots.scale;
         vars.P_Snapshot = _snapshots.P;
@@ -873,4 +878,8 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
     function _requireNonZeroAmount(uint _amount) internal pure {
         require(_amount > 0, 'StabilityPool: Amount must be non-zero');
     }
+
+    function _requireValidCollateralAddress(address _collateral) internal view {
+        require(collateralConfig.isCollateralAllowed(_collateral), "StabilityPool: Invalid collateral address");
+    }
 }
```

```diff
diff --git a/Ethos-Core/contracts/TroveManager.sol b/Ethos-Core/contracts/TroveManager.sol
index 6f587d9..affd3ac 100644
--- a/Ethos-Core/contracts/TroveManager.sol
+++ b/Ethos-Core/contracts/TroveManager.sol
@@ -363,7 +363,7 @@ contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager
         uint _LUSDInStabPool,
         uint _TCR,
         uint _price,
-        uint256 _MCR
+        uint256 _MCR //@audit (QA) maybe use uint to be consistent with other params. uint default to uint256 anyway.
     )
         internal
         returns (LiquidationValues memory singleLiquidation)
@@ -480,7 +480,7 @@ contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager
         uint _entireTroveDebt,
         uint _entireTroveColl,
         uint _price,
-        uint256 _MCR,
+        uint256 _MCR, //@audit (QA) maybe use uint to be consistent with other params. uint default to uint256 anyway.
         uint _collDecimals
     )
         internal
@@ -674,7 +674,7 @@ contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager
         IDefaultPool _defaultPool,
         address _collateral,
         uint _price,
-        uint256 _MCR,
+        uint256 _MCR, //@audit (QA) maybe use uint to be consistent with other params. uint default to uint256 anyway.
         uint _LUSDInStabPool,
         uint _n
     )
@@ -1446,7 +1446,7 @@ contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager
     }
 
     function _calcRedemptionFee(uint _redemptionRate, uint _collateralDrawn) internal pure returns (uint) {
-        uint redemptionFee = _redemptionRate.mul(_collateralDrawn).div(DECIMAL_PRECISION);
+        uint redemptionFee = _redemptionRate.mul(_collateralDrawn).div(DECIMAL_PRECISION); //@audit (QA) should this be colateral decimals?
         require(redemptionFee < _collateralDrawn);
         return redemptionFee;
     }
```

```diff
diff --git a/Ethos-Vault/test/starter-test.js b/Ethos-Vault/test/starter-test.js
index 4069176..28915f7 100644
--- a/Ethos-Vault/test/starter-test.js
+++ b/Ethos-Vault/test/starter-test.js
@@ -1073,7 +1073,8 @@ describe('Vaults', function () {
       let stratBalance = await strategy.balanceOf();
       expect(stratBalance).to.be.gte(toWantUnit('9'));
 
-      await vault.setEmergencyShutdown(true);
+      //@audit (QA) needs to call the strategy emergency, not the vault.
+      await strategy.setEmergencyExit();
       await strategy.harvest();
       vaultBalance = await want.balanceOf(vault.address);
       expect(vaultBalance).to.be.gte(toWantUnit('10'));
```



