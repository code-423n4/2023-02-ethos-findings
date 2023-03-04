## Impact

There are several potential re-entrant functions in contracts/BorrowerOperations.sol:

==> Function addColl() on line 242 is potentially re-entrant as it is external but has no re-entrancy guard declared. This function invokes _adjustTrove() which potentially impacts user debt, collateral top-ups or withdrawals.

Same applies to
-- withdrawColl() on line 254 -- withdrawLUSD() on line 259 -- repayLUSD() on line 264 -- adjustTrove() on line 268 

==> Function closeTrove() on line 375 is potentially re-entrant as it is external but has no re-entrancy guard declared. This function invokes troveManagerCached.removeStake(msg.sender) and troveManagerCached.closeTrove(msg.sender) impacting outcomes like debt, rewards and trove ownership.

## Proof of Concept
https://solidity-by-example.org/hacks/re-entrancy/
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L242
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L254
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L259
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L264
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L375

## Tools Used
Manual.

## Recommended Mitigation Steps

Potential solution is a re-entrancy guard similar to
https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard
