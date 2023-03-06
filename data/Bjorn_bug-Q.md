## [L-01] In case of an emergency exit a compromised KEEPER can liquidate all positions
According to the Natspec at `ReaperBaseStrategyv4.sol`, it does not mention that the `KEEPER` role is a multi-signature role. Therefore, in case of an emergency exit being activated, a compromised `KEEPER` could call the `harvest()` and `liquidateAllPositions()` back to the vault, even if it is not the desired action by the protocol team.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L105-L138
## Recommended Mitigation Steps
Limit access to non-multi-sign wallets Roles for calling sensitive functions in the protocol to mitigate the impact of a compromised actor

## [L-02] `SafeMath` library is used but not imported in `LiquityBase.sol`
The contract `LiquityBase` uses `SafeMath` library for units without importing it, and it uses the `0.6.11` compiler version. which makes the contract and others contracts that will interact with it(e.g,`BorrowerOperations.sol`) vulnerable to underflow and overflow attacks.
https://github.com/OpenZeppelin/openzeppelin-conSafeMath library not imported in LiquityBase contracttracts/blob/master/contracts/utils/math/SafeMath.sol
## Recommended Mitigation Steps
Import and use the latest version of the SafeMath library from OpenZeppelin and update the compiler version to ^0.8.16.

## [L-03] Use OpenZeppelin `ECDA` for `erecover`
The `LUSDToken.sol` contract's permit function uses the `ecrecover` to verify signatures. I would recommend using the `ECDSA` from OpenZeppelin as it does more validations when verifying the signature.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287
## Recommended Mitigation Steps
```
import {ECDSA} from "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
//...
//Before
address recoveredAddress = ecrecover(digest, v, r, s);
// After
address recoveredAddress = ECDSA.recover(digest, v, r, s);
...//
```

## [N-01] Use a more recent version of solidity
The protocol uses a very old version of the solidity compiler 0.6.11 in most of the contracts at Ethos-Core. Use a solidity version of at least 0.8.4 to get bytes.concat() instead of abi.encodePacked(,) Use a solidity version of at least 0.8.12 to get string.concat() instead of abi.encodePacked(,) Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/DefaultPool.sol#L3

## [N-02] Add a timelock to critical functions
It is good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing the risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L125
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L132
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144
 
## [N-03] Use modifiers instead of functions
It’s generally recommended and good practice to use modifiers instead of functions due to their readability, reusability, and gas efficiency.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L869
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1520-L1540
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L873

## [N-04] use SafeMathUpgradeable instead MathUpgradeable
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L14
https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/math/SafeMathUpgradeable.sol

## [N-05] Avoid hardcoding addresses in contracts to enable future updates
In case the addresses change due to reasons such as updating their versions in the future, addresses coded as constants cannot be updated, so it is recommended to add the update option with the onlyOwner modifier.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L24-L29
