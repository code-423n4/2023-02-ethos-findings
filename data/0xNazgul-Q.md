## [NAZ-L1] `removeTvlCap()` doesn't remove TVL cap
**Severity**: Low
**Context**: [`ReaperVaultV2.sol#L587`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L587)

**Description**:
`removeTvlCap()` doesn't completely remove TVL cap. Although it does so effectively, it can be reset to lower than `type(uint256).max`. 

**Recommendation**:
Consider adding in a check in `updateTvlCap()` like so:
```Solidity
function updateTvlCap(uint256 _newTvlCap) public {
    _atLeastRole(ADMIN);
    if (tvlCap != type(uint256).max) {
        tvlCap = _newTvlCap;
        emit TvlCapUpdated(tvlCap);
    }
}
```