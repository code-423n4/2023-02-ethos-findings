#########  Event information is incomplete ######### 

#### Impact
In the TroveManager._updateTroveRewardSnapshots function, we should emit an event, including the user address, when we update a user snapshot.

emit TroveSnapshotsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);

should become 

emit TroveSnapshotsUpdated(_borrower, _collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);

#### Findings:
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1119

#### Tools used
Manually