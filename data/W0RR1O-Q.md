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


