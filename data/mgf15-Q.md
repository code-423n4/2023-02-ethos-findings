
Low Risk and Non-Critical Issues .

[L-01] Using vulnerable dependency of OpenZeppelin
The package.json configuration file says that the project is using 3.3.0 of OZ which has a not last update version.

```
"devDependencies": {
    "@openzeppelin/contracts": "^3.3.0",
```
ref
https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts

Recommendation:
Use patched versions
Latest non vulnerable version 4.8.0

Non-Critical Issues 

[N-01] Stop using transferFrom() ! 

It is recommended to use safeTransferFrom() instead of transferFrom()
```
    function fund(uint amount) external onlyOwner {
        require(amount != 0, "cannot fund 0");
        OathToken.transferFrom(msg.sender, address(this), amount);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103

[N-02] Stop using transfer() ! 

it seems that the contract is using transfer to send tokens/ether . 

```
        OathToken.transfer(_account, _OathAmount);
```
Code 
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127

ref
https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/




[N-03] Use require instead of assert !

The Solidity assert() function is meant to assert invariants. Properly functioning code should never reach a failing assert statement.
```
assert(MIN_NET_DEBT > 0);

assert(vars.compositeDebt > 0);

assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

assert(_collWithdrawal <= vars.coll); 
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```
assert(totalStakesSnapshot[_collateral] > 0);

assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

assert(newBaseRate > 0); // Base rate is always non-zero after redemption

assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```
assert(_debtToOffset <= _totalLUSDDeposits);

assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);

assert(newP > 0);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```
assert(sender != address(0));

assert(recipient != address(0));

assert(account != address(0));

assert(owner != address(0));

assert(spender != address(0));
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

[N-04] Use a more recent version of Solidity

Context:
All contracts

Description:
For security, it is best practice to use the latest Solidity version.
For the security fix list in the versions;
https://github.com/ethereum/solidity/blob/develop/Changelog.md

Recommendation:
Old version of Solidity is used (^0.6.11) and (^0.8.0), newer version can be used (0.8.17).

[N-05] Local variable assignment . 

Catch frequently used storage variables in memory/stack, converting multiple SLOAD into 1 SLOAD.
from 
```
for(uint256 i = 0; i < _collaterals.length; i++) {
```
to 
```
uint256 l = _collaterals.length;
for(uint256 i = 0; i < l; i++) {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol

and on all other files .