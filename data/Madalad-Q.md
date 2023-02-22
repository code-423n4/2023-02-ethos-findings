# QA Report

## Solidity's `ecrecover` is vulnerable to signature malleability

The built-in EVM precompile `ecrecover` is susceptible to signature malleability, which could lead to replay attacks.
References: https://swcregistry.io/docs/SWC-117, https://swcregistry.io/docs/SWC-121, and https://medium.com/cryptronics/signature-replay-vulnerabilities-in-smart-contracts-3b6f7596df57

While this is not immediately exploitable, this may become a vulnerability if used elsewhere.

Consider using [OpenZeppelin's ECDSA library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/cryptography/ECDSA.sol) (which prevents this malleability) instead of the built-in function.

Instances: 2
- [Ethos-Core/contracts/LUSDToken.sol#L275](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L275)
- [Ethos-Core/contracts/LUSDToken.sol#L287](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287)

<br>

## Use `indexed` for event parameters

Index event fields make the field more quickly accessible to off-chain tools that parse events. 

If the variable is value type (uint, address, bool), then using `indexed` also saves gas, so there is no tradeoff.

Instances: 113
- [Ethos-Core/contracts/CollateralConfig.sol#L37](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L37)
- [Ethos-Core/contracts/CollateralConfig.sol#L38](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L38)
- [Ethos-Core/contracts/BorrowerOperations.sol#L92](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L92)
- [Ethos-Core/contracts/BorrowerOperations.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L93)
- [Ethos-Core/contracts/BorrowerOperations.sol#L94](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L94)
- [Ethos-Core/contracts/BorrowerOperations.sol#L95](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L95)
- [Ethos-Core/contracts/BorrowerOperations.sol#L96](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L96)
- [Ethos-Core/contracts/BorrowerOperations.sol#L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L97)
- [Ethos-Core/contracts/BorrowerOperations.sol#L98](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L98)
- [Ethos-Core/contracts/BorrowerOperations.sol#L99](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L99)
- [Ethos-Core/contracts/BorrowerOperations.sol#L100](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L100)
- [Ethos-Core/contracts/BorrowerOperations.sol#L101](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L101)
- [Ethos-Core/contracts/BorrowerOperations.sol#L102](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L102)
- [Ethos-Core/contracts/BorrowerOperations.sol#L104](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L104)
- [Ethos-Core/contracts/BorrowerOperations.sol#L105](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L105)
- [Ethos-Core/contracts/BorrowerOperations.sol#L106](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L106)
- [Ethos-Core/contracts/TroveManager.sol#L186](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L186)
- [Ethos-Core/contracts/TroveManager.sol#L187](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L187)
- [Ethos-Core/contracts/TroveManager.sol#L188](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L188)
- [Ethos-Core/contracts/TroveManager.sol#L189](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L189)
- [Ethos-Core/contracts/TroveManager.sol#L190](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L190)
- [Ethos-Core/contracts/TroveManager.sol#L191](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L191)
- [Ethos-Core/contracts/TroveManager.sol#L192](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L192)
- [Ethos-Core/contracts/TroveManager.sol#L193](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L193)
- [Ethos-Core/contracts/TroveManager.sol#L194](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L194)
- [Ethos-Core/contracts/TroveManager.sol#L195](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L195)
- [Ethos-Core/contracts/TroveManager.sol#L196](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L196)
- [Ethos-Core/contracts/TroveManager.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L197)
- [Ethos-Core/contracts/TroveManager.sol#L198](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L198)
- [Ethos-Core/contracts/TroveManager.sol#L200](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L200)
- [Ethos-Core/contracts/TroveManager.sol#L201](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L201)
- [Ethos-Core/contracts/TroveManager.sol#L202](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L202)
- [Ethos-Core/contracts/TroveManager.sol#L203](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L203)
- [Ethos-Core/contracts/TroveManager.sol#L204](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L204)
- [Ethos-Core/contracts/TroveManager.sol#L205](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L205)
- [Ethos-Core/contracts/TroveManager.sol#L206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L206)
- [Ethos-Core/contracts/TroveManager.sol#L207](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L207)
- [Ethos-Core/contracts/TroveManager.sol#L208](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L208)
- [Ethos-Core/contracts/TroveManager.sol#L209](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L209)
- [Ethos-Core/contracts/TroveManager.sol#L211](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L211)
- [Ethos-Core/contracts/TroveManager.sol#L212](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L212)
- [Ethos-Core/contracts/TroveManager.sol#L213](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L213)
- [Ethos-Core/contracts/TroveManager.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L214)
- [Ethos-Core/contracts/TroveManager.sol#L215](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L215)
- [Ethos-Core/contracts/ActivePool.sol#L58](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L58)
- [Ethos-Core/contracts/ActivePool.sol#L59](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L59)
- [Ethos-Core/contracts/ActivePool.sol#L60](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L60)
- [Ethos-Core/contracts/ActivePool.sol#L61](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L61)
- [Ethos-Core/contracts/ActivePool.sol#L62](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L62)
- [Ethos-Core/contracts/ActivePool.sol#L63](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L63)
- [Ethos-Core/contracts/ActivePool.sol#L64](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L64)
- [Ethos-Core/contracts/ActivePool.sol#L65](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L65)
- [Ethos-Core/contracts/ActivePool.sol#L66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L66)
- [Ethos-Core/contracts/ActivePool.sol#L67](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L67)
- [Ethos-Core/contracts/StabilityPool.sol#L233](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L233)
- [Ethos-Core/contracts/StabilityPool.sol#L234](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L234)
- [Ethos-Core/contracts/StabilityPool.sol#L236](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L236)
- [Ethos-Core/contracts/StabilityPool.sol#L237](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L237)
- [Ethos-Core/contracts/StabilityPool.sol#L238](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L238)
- [Ethos-Core/contracts/StabilityPool.sol#L239](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L239)
- [Ethos-Core/contracts/StabilityPool.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L240)
- [Ethos-Core/contracts/StabilityPool.sol#L241](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L241)
- [Ethos-Core/contracts/StabilityPool.sol#L242](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L242)
- [Ethos-Core/contracts/StabilityPool.sol#L243](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L243)
- [Ethos-Core/contracts/StabilityPool.sol#L244](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L244)
- [Ethos-Core/contracts/StabilityPool.sol#L246](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L246)
- [Ethos-Core/contracts/StabilityPool.sol#L247](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L247)
- [Ethos-Core/contracts/StabilityPool.sol#L248](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L248)
- [Ethos-Core/contracts/StabilityPool.sol#L249](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L249)
- [Ethos-Core/contracts/StabilityPool.sol#L250](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L250)
- [Ethos-Core/contracts/StabilityPool.sol#L252](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L252)
- [Ethos-Core/contracts/StabilityPool.sol#L253](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L253)
- [Ethos-Core/contracts/StabilityPool.sol#L255](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L255)
- [Ethos-Core/contracts/StabilityPool.sol#L256](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L256)
- [Ethos-Core/contracts/StabilityPool.sol#L257](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L257)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L50)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L51)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L52)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L53](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L53)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L49](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L49)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L50)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L51)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L52)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L53](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L53)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L54](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L54)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L56](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L56)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L57](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L57)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L58](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L58)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L59](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L59)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L60](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L60)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L61](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L61)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L62](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L62)
- [Ethos-Core/contracts/LUSDToken.sol#L78](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L78)
- [Ethos-Core/contracts/LUSDToken.sol#L79](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L79)
- [Ethos-Core/contracts/LUSDToken.sol#L80](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L80)
- [Ethos-Core/contracts/LUSDToken.sol#L81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L81)
- [Ethos-Core/contracts/LUSDToken.sol#L82](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L82)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L80](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L80)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L81)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L82](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L82)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L85](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L85)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L86](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L86)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L87](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L87)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L88](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L88)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L89](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L89)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L92](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L92)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L93)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L94](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L94)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L95](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L95)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L96](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L96)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L97)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L98](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L98)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L99](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L99)

<br>

## Use fixed compiler version

A floating pragma risks a different compiler version being used in production vs testing, which poses security risks.

Instances: 4
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3)

<br>

## Lines too long

[Solidity's style guide](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length) states lines should be kept within 79 characters.
 
Also, if lines exceed 164 characters then a horizontal scroll bar will be required when viewing the file on Github.

Extensions such as prettier are a simple solution.

Instances: 11
- [Ethos-Core/contracts/BorrowerOperations.sol#L268](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268)
- [Ethos-Core/contracts/BorrowerOperations.sol#L282](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L282)
- [Ethos-Core/contracts/BorrowerOperations.sol#L343](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L343)
- [Ethos-Core/contracts/BorrowerOperations.sol#L511](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L511)
- [Ethos-Core/contracts/TroveManager.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L351)
- [Ethos-Core/contracts/TroveManager.sol#L393](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L393)
- [Ethos-Core/contracts/TroveManager.sol#L407](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L407)
- [Ethos-Core/contracts/TroveManager.sol#L428](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L428)
- [Ethos-Core/contracts/TroveManager.sol#L934](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L934)
- [Ethos-Core/contracts/TroveManager.sol#L1044](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1044)
- [Ethos-Core/contracts/StabilityPool.sol#L637](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L637)

<br>

## Use `safeTransfer` instead of `transfer` for token transfers

`SafeERC20` is a wrapper around ERC20 operations that throw on failure (when the token contract returns false). This prevents calls executing if a token transfer is unsuccessful. Simply add `using SafeERC20 for ERC20`.

Instances: 2
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127)

<br>

## Low level calls before Solidity version `0.8.14` may result in optimiser bug

See https://medium.com/certora/overly-optimistic-optimizer-certora-bug-disclosure-2101e3f7994d for more information.

Consider using a more recent Solidity version.

Instances: 12
- [Ethos-Core/contracts/CollateralConfig.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3)
- [Ethos-Core/contracts/BorrowerOperations.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3)
- [Ethos-Core/contracts/TroveManager.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3)
- [Ethos-Core/contracts/ActivePool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3)
- [Ethos-Core/contracts/StabilityPool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3)
- [Ethos-Core/contracts/LUSDToken.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3)

<br>

## Use two-step ownership transfers

The current ownership transfer process involves the current owner calling `transferOwnership()`. This function checks the new owner is not the zero address and proceeds to write the new owner's address into the owner's state variable.

If the nominated EOA account is not a valid account, it is entirely possible the owner may accidentally transfer ownership to an uncontrolled account, breaking all functions with the onlyOwner() modifier.

Consider implementing a two step process where the owner nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed.

This can be achieved using OpenZeppelin's `Ownable2Step`.

Instances: 6
- [Ethos-Core/contracts/CollateralConfig.sol#L15](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L15)
- [Ethos-Core/contracts/BorrowerOperations.sol#L18](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L18)
- [Ethos-Core/contracts/ActivePool.sol#L26](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L26)
- [Ethos-Core/contracts/StabilityPool.sol#L146](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L146)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L14](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L14)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L18](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L18)

<br>

## Remove unused imports

Improves readability and saves gas.

Instances: 8
- [Ethos-Core/contracts/BorrowerOperations.sol#L15](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L15)
- [Ethos-Core/contracts/ActivePool.sol#L15](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L15)
- [Ethos-Core/contracts/StabilityPool.sol#L18](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L18)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L8](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L8)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L9](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L9)
- [Ethos-Core/contracts/LUSDToken.sol#L6](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L6)
- [Ethos-Core/contracts/LUSDToken.sol#L9](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L9)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L10](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L10)

<br>

## Missing `address(0)` checks in constructor/initialize

Failing to check for invalid parameters on deployment may result in expensive redeployment.

Instances: 4
- [Ethos-Core/contracts/LUSDToken.sol#L84](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L84)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L111](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L16](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L16)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62)

<br>
