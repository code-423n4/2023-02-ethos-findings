# 1: USE A MORE RECENT VERSION OF SOLIDITY

Optimization details

## Context:

Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining

Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads

Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings

Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value

## Proof of Concept

There are 12 instances of this issue:

File: contracts/ReaperStrategyGranarySupplyOnly.sol

3: pragma solidity ^0.8.0;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3

File: contracts/abstract/ReaperBaseStrategyv4.sol

3: pragma solidity ^0.8.0;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3 

File: contracts/ReaperVaultERC4626.sol 

3: pragma solidity ^0.8.0;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3 

File: contracts/ReaperVaultV2.sol

3: pragma solidity ^0.8.0;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L3 


File: contracts/LUSDToken.sol

3: pragma solidity ^0.8.0;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L3 

File: contracts/LQTY/LQTYStaking.sol 

3: pragma solidity 0.6.11;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3 

File: contracts/LQTY/CommunityIssuance.sol

3: pragma solidity 0.6.11;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3 

File: contracts/StabilityPool.sol

3: pragma solidity 0.6.11;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L3 

File: contracts/ActivePool.sol

3: pragma solidity 0.6.11; 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L3 

File: contracts/TroveManager.sol

3: pragma solidity 0.6.11;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L3 

File: contracts/BorrowerOperations.sol

3: pragma solidity 0.6.11;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L3 

File: contracts/CollateralConfig.sol

3: pragma solidity 0.6.11;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L3 

### Tools Used

Manual Analysis

