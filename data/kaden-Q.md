#### Unused internal method

##### Severity: Low

##### Context: 

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L432

##### Description

`BorrowerOperations._getUSDValue` is a internal method which is not called, and thus is dead code.


#### Events not being emitted

##### Severity: Low

##### Context:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L194
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L201

##### Description

`ActivePoolLUSDDebtUpdated` events in `ActivePool` are missing the `emit` keyword and thus not actually being emitted as intended.


#### Unused variables

##### Severity: Low

##### Context:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L339
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L384

##### Description

The above listed `LUSDLoss` variables are unused.


#### Use `safeTransfer/safeTransferFrom` instead of `transfer/transferFrom`

##### Severity: Low

##### Context:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127

##### Description

OATH transfers in `CommunityIssuance` contract are not using recommended `safeERC20` logic.


#### Outdated compiler version

##### Severity: Low

##### Description

The contracts in `ethos-core` use solidity v0.6.11, which is quite outdated, with the current version at the time of writing being v0.8.19. It is generally recommended that newer versions of the solidity compiler are used to avoid any bugs and issues which may exist on lower versions.