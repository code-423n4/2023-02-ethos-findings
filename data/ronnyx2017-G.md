# 1. simplified calculation for repayment

code lines: https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L118-L129

if roi < 0, roi = -int256(debt - amountFreed) , so `repayment = debt; repayment -= uint256(-roi)` can be simplified to `repayment = debt + roi = debt - debt + amountFreed = amountFreed`.

So if roi < 0, repayment = amountFreed.