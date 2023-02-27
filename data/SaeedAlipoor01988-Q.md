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

#########  Omissions in Events ######### 

#### Impact
Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters. The events should include the new value and old value where possible.

#### Findings:
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L172
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L176
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L180

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L150
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L157

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L129
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L135
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L141


#### Tools used
Manually