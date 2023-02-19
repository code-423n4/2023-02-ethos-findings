## 1. Use bytes instead of string to save gas .:-
If string called then it cost more gas than bytes data types . So try change to bytes32 and compare if any access control purpose .

 code snippet:-
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21

recommendation :-
change to 
bytes32 immutable stringName  = keccak256("string can here");


## 2 . Using Solidity version 0.8.17 will provide an overall gas optimization

Using at least 0.8.10 will save gas due to skipped extcodesize check if there is a return value. Currently the contracts are compiled using version 0.8.7 (Foundry default). It is easily changeable to 0.8.17 using the command sed -i 's/0\.8\.7/^0.8.0/' test/*.sol && sed -i '4isolc = "0.8.17"' foundry.toml. This will have the following total savings obtained by forge snapshot --diff | tail -1
 

codes snippet:-
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3
