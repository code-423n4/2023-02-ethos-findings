```diff
diff --git a/Ethos-Core/contracts/ActivePool.sol b/Ethos-Core/contracts/ActivePool.sol
index 753fcd0..771da52 100644
--- a/Ethos-Core/contracts/ActivePool.sol
+++ b/Ethos-Core/contracts/ActivePool.sol
@@ -176,12 +176,13 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
         emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);
         emit CollateralSent(_collateral, _account, _amount);
 
+        //@audit (GAS): save using the parameter _account instead of the state variable defaultPoolAddress and collSurplusPoolAddress
         if (_account == defaultPoolAddress) {
-            IERC20(_collateral).safeIncreaseAllowance(defaultPoolAddress, _amount);
-            IDefaultPool(defaultPoolAddress).pullCollateralFromActivePool(_collateral, _amount);
+            IERC20(_collateral).safeIncreaseAllowance(_account, _amount); 
+            IDefaultPool(_account).pullCollateralFromActivePool(_collateral, _amount);
         } else if (_account == collSurplusPoolAddress) {
-            IERC20(_collateral).safeIncreaseAllowance(collSurplusPoolAddress, _amount);
-            ICollSurplusPool(collSurplusPoolAddress).pullCollateralFromActivePool(_collateral, _amount);
+            IERC20(_collateral).safeIncreaseAllowance(_account, _amount);
+            ICollSurplusPool(_account).pullCollateralFromActivePool(_collateral, _amount);
         } else {
             IERC20(_collateral).safeTransfer(_account, _amount);
         }
@@ -274,6 +275,7 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
         }
 
         // + means deposit, - means withdraw
+        //@audit (GAS) can save some gas here with yieldGenerator[_collateral] being local var
         vars.netAssetMovement = int256(vars.toDeposit) - int256(vars.toWithdraw) - int256(vars.profit);
         if (vars.netAssetMovement > 0) {
             IERC20(_collateral).safeIncreaseAllowance(yieldGenerator[_collateral], uint256(vars.netAssetMovement));
```

```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index 2be0b81..a60c162 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -266,7 +266,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
         delete withdrawalQueue;
         for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
             address strategy = _withdrawalQueue[i];
-            StrategyParams storage params = strategies[strategy];
+            StrategyParams storage params = strategies[strategy]; // @audit (GAS) use memory instead of storage, only reading
             require(params.activation != 0, "Invalid strategy address");
             withdrawalQueue.push(strategy);
         }
@@ -610,6 +610,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
      * goes back into Normal Operation.
      */
     function setEmergencyShutdown(bool _active) external {
+        //@audit (GAS) add a require to prevent waste : require(_active != emergencyShutdown, "Already in this state");
         if (_active) {
             _atLeastRole(GUARDIAN);
         } else {
```

```diff
diff --git a/Ethos-Core/contracts/SortedTroves.sol b/Ethos-Core/contracts/SortedTroves.sol
index 35a4634..3ad8962 100644
--- a/Ethos-Core/contracts/SortedTroves.sol
+++ b/Ethos-Core/contracts/SortedTroves.sol
@@ -124,31 +124,33 @@ contract SortedTroves is Ownable, CheckContract, ISortedTroves {
             (prevId, nextId) = _findInsertPosition(_troveManager, _collateral, _NICR, prevId, nextId);
         }
 
-         data[_collateral].nodes[_id].exists = true;
+        //@audit (GAS) save using a local variable
+        Data storage dataCollateralCached = data[_collateral];
+        dataCollateralCached.nodes[_id].exists = true;
 
         if (prevId == address(0) && nextId == address(0)) {
             // Insert as head and tail
-            data[_collateral].head = _id;
-            data[_collateral].tail = _id;
+            dataCollateralCached.head = _id;
+            dataCollateralCached.tail = _id;
         } else if (prevId == address(0)) {
             // Insert before `prevId` as the head
-            data[_collateral].nodes[_id].nextId = data[_collateral].head;
-            data[_collateral].nodes[data[_collateral].head].prevId = _id;
-            data[_collateral].head = _id;
+            dataCollateralCached.nodes[_id].nextId = dataCollateralCached.head;
+            dataCollateralCached.nodes[dataCollateralCached.head].prevId = _id;
+            dataCollateralCached.head = _id;
         } else if (nextId == address(0)) {
             // Insert after `nextId` as the tail
-            data[_collateral].nodes[_id].prevId = data[_collateral].tail;
-            data[_collateral].nodes[data[_collateral].tail].nextId = _id;
-            data[_collateral].tail = _id;
+            dataCollateralCached.nodes[_id].prevId = dataCollateralCached.tail;
+            dataCollateralCached.nodes[dataCollateralCached.tail].nextId = _id;
+            dataCollateralCached.tail = _id;
         } else {
             // Insert at insert position between `prevId` and `nextId`
-            data[_collateral].nodes[_id].nextId = nextId;
-            data[_collateral].nodes[_id].prevId = prevId;
-            data[_collateral].nodes[prevId].nextId = _id;
-            data[_collateral].nodes[nextId].prevId = _id;
+            dataCollateralCached.nodes[_id].nextId = nextId;
+            dataCollateralCached.nodes[_id].prevId = prevId;
+            dataCollateralCached.nodes[prevId].nextId = _id;
+            dataCollateralCached.nodes[nextId].prevId = _id;
         }
 
-        data[_collateral].size = data[_collateral].size.add(1);
+        dataCollateralCached.size = dataCollateralCached.size.add(1);
         emit NodeAdded(_collateral, _id, _NICR);
     }
 
@@ -166,36 +168,39 @@ contract SortedTroves is Ownable, CheckContract, ISortedTroves {
         // List must contain the node
         require(contains(_collateral, _id), "SortedTroves: List does not contain the id");
 
-        if (data[_collateral].size > 1) {
+        //@audit (GAS) save using a local variable
+        Data storage dataCollateralCached = data[_collateral];
+
+        if (dataCollateralCached.size > 1) {
             // List contains more than a single node
-            if (_id == data[_collateral].head) {
+            if (_id == dataCollateralCached.head) {
                 // The removed node is the head
                 // Set head to next node
-                data[_collateral].head = data[_collateral].nodes[_id].nextId;
+                dataCollateralCached.head = dataCollateralCached.nodes[_id].nextId;
                 // Set prev pointer of new head to null
-                data[_collateral].nodes[data[_collateral].head].prevId = address(0);
-            } else if (_id == data[_collateral].tail) {
+                dataCollateralCached.nodes[dataCollateralCached.head].prevId = address(0);
+            } else if (_id == dataCollateralCached.tail) {
                 // The removed node is the tail
                 // Set tail to previous node
-                data[_collateral].tail = data[_collateral].nodes[_id].prevId;
+                dataCollateralCached.tail = dataCollateralCached.nodes[_id].prevId;
                 // Set next pointer of new tail to null
-                data[_collateral].nodes[data[_collateral].tail].nextId = address(0);
+                dataCollateralCached.nodes[dataCollateralCached.tail].nextId = address(0);
             } else {
                 // The removed node is neither the head nor the tail
                 // Set next pointer of previous node to the next node
-                data[_collateral].nodes[data[_collateral].nodes[_id].prevId].nextId = data[_collateral].nodes[_id].nextId;
+                dataCollateralCached.nodes[dataCollateralCached.nodes[_id].prevId].nextId = dataCollateralCached.nodes[_id].nextId;
                 // Set prev pointer of next node to the previous node
-                data[_collateral].nodes[data[_collateral].nodes[_id].nextId].prevId = data[_collateral].nodes[_id].prevId;
+                dataCollateralCached.nodes[dataCollateralCached.nodes[_id].nextId].prevId = dataCollateralCached.nodes[_id].prevId;
             }
         } else {
             // List contains a single node
             // Set the head and tail to null
-            data[_collateral].head = address(0);
-            data[_collateral].tail = address(0);
+            dataCollateralCached.head = address(0);
+            dataCollateralCached.tail = address(0);
         }
 
-        delete data[_collateral].nodes[_id];
-        data[_collateral].size = data[_collateral].size.sub(1);
+        delete dataCollateralCached.nodes[_id];
+        dataCollateralCached.size = dataCollateralCached.size.sub(1);
         NodeRemoved(_collateral, _id);
     }
```

```diff
diff --git a/Ethos-Core/contracts/StabilityPool.sol b/Ethos-Core/contracts/StabilityPool.sol
index a33d8c3..5f8fac6 100644
--- a/Ethos-Core/contracts/StabilityPool.sol
+++ b/Ethos-Core/contracts/StabilityPool.sol
@@ -807,13 +807,14 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
 
     function _updateDepositAndSnapshots(address _depositor, uint _newValue) internal {
         deposits[_depositor].initialValue = _newValue;
+        Snapshots storage depositSnapshotCached = depositSnapshots[_depositor]; //@audit (GAS) added a cache snapshot
 
         address[] memory collaterals = collateralConfig.getAllowedCollaterals();
         uint[] memory amounts = new uint[](collaterals.length);
 
         if (_newValue == 0) {
             for (uint i = 0; i < collaterals.length; i++) {
-                delete depositSnapshots[_depositor].S[collaterals[i]];
+                delete depositSnapshotCached.S[collaterals[i]];
             }
             delete depositSnapshots[_depositor];
             emit DepositSnapshotUpdated(_depositor, 0, collaterals, amounts, 0);
@@ -827,16 +828,16 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
         uint currentG = epochToScaleToG[currentEpochCached][currentScaleCached];
 
         // Record new snapshots of the latest running product P, and sum G, for the depositor
-        depositSnapshots[_depositor].P = currentP;
-        depositSnapshots[_depositor].G = currentG;
-        depositSnapshots[_depositor].scale = currentScaleCached;
-        depositSnapshots[_depositor].epoch = currentEpochCached;
+        depositSnapshotCached.P = currentP;
+        depositSnapshotCached.G = currentG;
+        depositSnapshotCached.scale = currentScaleCached;
+        depositSnapshotCached.epoch = currentEpochCached;
 
         // Record new snapshots of the latest running sum S for all collaterals for the depositor
         for (uint i = 0; i < collaterals.length; i++) {
             address collateral = collaterals[i];
             uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][collateral];
-            depositSnapshots[_depositor].S[collateral] = currentS;
+            depositSnapshotCached.S[collateral] = currentS;
             amounts[i] = currentS;
         }
         emit DepositSnapshotUpdated(_depositor, currentP, collaterals, amounts, currentG);
@@ -844,6 +845,7 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
 
     function _payOutLQTYGains(ICommunityIssuance _communityIssuance, address _depositor) internal {
         uint depositorLQTYGain = getDepositorLQTYGain(_depositor);
+        if (depositorLQTYGain == 0) {return;} //@audit (GAS) don't interact with external contract if not needed too?
         _communityIssuance.sendOath(_depositor, depositorLQTYGain);
         emit LQTYPaidToDepositor(_depositor, depositorLQTYGain);
     }
```