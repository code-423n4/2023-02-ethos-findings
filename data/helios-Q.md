# 1. Reentrancy in `contracts/BorrowerOperations.sol`
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

## Impact

There are several potential re-entrant functions in contracts/BorrowerOperations.sol:

==> Function `addColl()` on line 242 is potentially re-entrant as it is external but has no re-entrancy guard declared. This function invokes `_adjustTrove()` which potentially impacts user debt, collateral top-ups or withdrawals.

Same applies to
-- `withdrawColl()` on line 254 -- `withdrawLUSD()` on line 259 -- `repayLUSD()` on line 264 -- `adjustTrove()` on line 268 

==> Function `closeTrove()` on line 375 is potentially re-entrant as it is external but has no re-entrancy guard declared. This function invokes `troveManagerCached.removeStake(msg.sender)` and `troveManagerCached.closeTrove(msg.sender)` impacting outcomes like debt, rewards and trove ownership.

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

# 2. Unchecked return value for `transferFrom` calls

To avoid potential issues with token transfers and contract token accounting, it is recommended to include a `require()` statement that verifies the `return` value of the transfer or to use a safe transfer method like OpenZeppelin's `safeTransfer/safeTransferFrom`. This is especially important unless there is certainty that the specific token being used will automatically revert in case of a transfer failure. Failing to implement these precautions may result in transfers failing silently and impacting contract functionality.Failure to do so will cause silent failures of transfers and affect token accounting in contract.

==> Navigate to the following contracts.
==> `transfer/transferFrom` functions are used instead of safe `transfer/transferFrom` on the following contracts.

Links : https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127
https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103

Recommendation: 

Consider using `safeTransfer/safeTransferFrom` or `require()` consistently.
