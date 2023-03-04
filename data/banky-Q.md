# Add error messages for `require` statements

Doing a diff with the liquity codebase, some useful error messaging has been removed from `require` statements. This makes it difficult for users to figure out why their transactions didn't work. The `require` statements in question are: 

1. https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/TroveManager.sol#L551
2. https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/TroveManager.sol#L716
3. https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/TroveManager.sol#L754
