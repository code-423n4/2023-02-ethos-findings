Ethos-Core
------------
In the AccessControl, there are cases where the comments do not reflect exactly what the code is doing. Not major issues, but not totally accurate.
-------------
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol
contracts/TroveManager.sol
line 1077 - in function applyPendingRewards
the require is less restrictive than a require further down the call stack
```
        _requireCallerIsBorrowerOperationsOrRedemptionHelper();
        // _updateTroveRewardSnapshots calls _requireCallerIsBorrowerOperations() internally, so allowing RedemptionHelper is useless
        // _updateTroveRewardSnapshots is called by _applyPendingRewards(...)
```
so code should read
```
_requireCallerIsBorrowerOperations();
```
-----------------
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol
contracts/ActivePool.sol
line 334 - should include redemptionHelper
```
            "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool nor redemptionHelper");
```

-----------------
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/test/AccessControlTest.js
test/AccessControlTest.js
Access control tests text does not match access controls in solidity code (in some cases)
line 83 - add redemptionHelper
```
it("applyPendingRewards(): reverts when called by an account that is not BorrowerOperations nor RedemptionHelper", async () => {
```
line 107 - add redemptionHelper
```
it("removeStake(): reverts when called by an account that is not BorrowerOperations nor RedemptionHelper", async () => {
```
line 131 - add redemptionHelper
```
it("closeTrove(): reverts when called by an account that is not BorrowerOperations nor RedemptionHelper", async () => {
```
line 278 - add redemptionHelper
```
it("sendCollateral(): reverts when called by an account that is not BO nor TroveM nor SP nor redemptionHelper", async () => {
```
line 285 - add redemptionHelper
```
assert.include(err.message, "Caller is neither BorrowerOperations nor TroveManager nor StabilityPool nor redemptionHelper")
```
line 302 - add redemptionHelper
```
it("decreaseLUSDDebt(): reverts when called by an account that is not BO nor TroveM nor SP nor redemptionHelper", async () => {
```
line 309 - add redemptionHelper
```
assert.include(err.message, "Caller is neither BorrowerOperations nor TroveManager nor StabilityPool nor redemptionHelper")
```

        