## Using private rather than public on constant variables saves gas
### Explained
Substantial gas is saved by marking all constant state variables as private. The compiler does not have to create non-payable getter functions and the value can be read from the bytecode.

### Instances
BorrowerOperations

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L21)

CollateralConfig

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L21)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L25)

TroveManager

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L48](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L48)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L53](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L53)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L61](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L61)

ActivePool

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L30](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L30)

StabilityPool

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L150](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L150)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L197)

CommunityIssuance

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19)

LQTYStaking

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23)

ReaperVaultV2

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L40](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L40)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L41](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L41)

## No need to override variable type default values on declaration
### Explained
Not overwriting the default variables saves gas i.e. no SSTORE required.
Instances record the unrequired declarations of
- uint = 0
- bool = false

### Instances
ReaperStrategyGranarySupplyOnly
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35

ActivePool
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L32
https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/ActivePool.sol#L108

LQRYStaking

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228)

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240)

StabilityPool

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/StabilityPool.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/StabilityPool.sol#L351)

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/StabilityPool.sol#L397](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/StabilityPool.sol#L397)

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/StabilityPool.sol#L640](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/StabilityPool.sol#L640)

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/StabilityPool.sol#L810](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/StabilityPool.sol#L810)

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/StabilityPool.sol#L831](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/StabilityPool.sol#L831)

CollateralConfig

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/CollateralConfig.sol#L56](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/CollateralConfig.sol#L56)

RepearVaultV2

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/ReaperVaultV2.sol#L128](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/ReaperVaultV2.sol#L128)

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/ReaperVaultV2.sol#L264](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/ReaperVaultV2.sol#L264)

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/ReaperVaultV2.sol#L372](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/ReaperVaultV2.sol#L372)