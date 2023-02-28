# 1. Use revert with custom errors instead require

There are many places in code where require statement is used. If you use revert with custom errors instead require you will save a lot of gas.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L51-L54
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L66
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L69
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L91-L94
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L97
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L129
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L525
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L529-L530
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L534
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L538
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L543

