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


# 2: ABI.ENCODE() IS LESS EFFICIENT THAN ABI.ENCODEPACKED()

Optimization details

### Context:

Use abi.encodePacked() where possible to save gas

## Proof of Concept

2 Results 1 File:

***src/LUSDToken.sol***

- 284:  domainSeparator(), keccak256(abi.encode(

- 305: return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L284 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L305 

## Tools Used

Manual Analysis

# 3: SPLITTING REQUIRE() STATEMENTS THAT USE && SAVES GAS

Optimization details

## Context:

Instead of using the operator && on a single require check. Using double require check can save more gas.

For reference, see https://github.com/code-423n4/2022-01-xdefi-findings/issues/128

## Proof of Concept

There are 5 instances of this issue:

> ***File: contracts/LUSDToken.sol***

347:        require(
348:            _recipient != address(0) && 
349:            _recipient != address(this),
350:            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
351:        );

352:  require(
353:            !stabilityPools[_recipient] &&
354:            !troveManagers[_recipient] &&
355:           !borrowerOperations[_recipient],
356:            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
357:        );


https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L348 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L353 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L354 


> ***File: contracts/TroveManager.sol***

1539: require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1); 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1539 


> ***File: contracts/BorrowerOperations.sol***

653:            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
654:                "Max fee percentage must be between 0.5% and 100%");

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L653 


## Tools Used

Manual Analysis


# 4: REQUIRE() OR REVERT() STATEMENTS THAT CHECK INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

There is 1 instance of this issue:

> ***File: contracts/ActivePool.sol***

/// @audit expensive op on line 126
127:   require(_bps <= 10_000, "Invalid BPS value");

