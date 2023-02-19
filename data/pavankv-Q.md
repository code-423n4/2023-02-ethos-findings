## 1.  It's not good practice to use assert() in external function:-
Assert should only be used to test for internal errors, and to check invariants.

 code snippet:-
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197

reference :-
https://docs.soliditylang.org/en/v0.8.9/control-structures.html#panic-via-assert-and-error-via-require

## 2 . Add return comment for function which had return keyword :-

code snippet:-
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L299
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L303

reference :-
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## 3 .Weird returns :- 
It used  return -int
code snippet :-
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L228
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L235
