Use of `ecrecover` is susceptible to signature malleability
=====================================
Description:
-------------
The `permit` function in the contract `LUSDToken.sol` which directly uses `ecrecover()` to verify the given signatures. The built-in EVM precompile `ecrecover` is susceptible to signature malleability(because of the non-unique s and v values) which could lead to replay attacks. Although, a replay attack is not possible due to the nonce value is increased each time as well as checks are in place to ensure that the s value is in a certain range. However, it is considered best practice  (and so is checking `recoveredAddress != address(0)` where `address(0)` means an invalid signature)

A determined attacker could still create a valid signature with a different s or v value which would allow them to transfer more tokens than intended.

 Proof of concept:
--------------------
* https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287

Reference:
-----------
* https://swcregistry.io/docs/SWC-117

Recommended Mitigation Steps:
------------------------------------
Use the `recover` function from `https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol` for signature verification




Vulnerability in `issueOath()` function which can lead to a potential timestamp manipulation
===========================================================
Description:
-------------
The function `issueOath()` in the contract `CommunityIssuance.sol` has a potential vulnerability due wo its reliance on `block.timestamp` for  time-based calculations. The Ethereum Yellow Paper states that it is not guaranteed that 2 nodes on the network will have the same `block.timestamp` value. The function uses `bock.timestamp` to calculate the time passed since the last issuance of Oath. However, the timestamp can be manipulated by a miner to a certain extent causing them to controll the amount of oath issued.

 The severity of this issue is considered low as it requires a specific sequence of events to occur for it to be exploited. An attacker would need to be able to manipulate the `block.timestamp` and either be a miner or have a miner who is willing to manipulate the timestamp to their advantage. Additionally, the attacker would need to have overridden the `issueOath` function in another contract.

Proof of Concept:
--------------------
* https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L84-L95

Recommended Mitigation Steps:
------------------------------------
Use a more reliable and accurate source such as an off-chain oracle or consider using `block.number` instead `block.timestamp`


Avoid floating pragmas for non-library contracts
===============================
Description:
-------------
While allowing floating point pragmas makes sense for library contracts which allow them to be included with multiple different versions of applications. However, including floating point pragmas in non-library contracts might be a security risk as some solidity versions contains vulnerabilities. If you read through the release notes of any version of solidity ranging from 0.8.X to the current version we can see that all sorts of fixed issues

Proof of Concept:
-------------------
* https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3
* https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3
* https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3
* https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3

Recommended Mitigation Steps:
-----------------------------------
It is recommended to not use a floating pragma value for non-library contracts and only use a single and new solidity version. 


