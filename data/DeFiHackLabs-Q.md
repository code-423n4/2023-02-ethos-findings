
Solidity outdate version in Ethos.

**Proof of concept**
pragma solidity 0.6.11;

in all Ethos contract, ex:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3

**Recommendation:**
Update solidity 0.8.18 or higher which has the fix for some vulnerabilites.