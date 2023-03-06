# Table of contents

- [N-01] Insufficient test coverage
- [N-02] Use `require` instead of `assert`
- [N-03] Values such as a call to keccak256() should used to `immutable` rather than `constant`
- [N-04] `0 address` check for `asset`
- [N-05] Use a more recent version of solidity
- [N-06] Avoid the use of sensitive terms in favour of neutral ones
- [N-07] Non-library/interface files should use fixed compiler versions, not floating ones
- [N-08] Best practice is to prevent signature malleability
- [N-09] Import exact functions
- [L-01] Include check to ensure lastReport time > activation time
- [L-02] Loss of precision due to rounding
- [L-03] Reentrancy Guard not initialized

## [N-01] Insufficient test coverage
Testing all functions is best practice in terms of security criteria. Noted the current overall line coverage percentage provided by tests is 93. Due to its capacity, test coverage is expected to be 100%.

## [N-02] Use `require` instead of `assert`

`assert` should not be used except for tests; `require` should be used.

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

`assert()` and `reqire()` -

The big difference between the two is that the assert()function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay.This is the most common Solidity function used by developers for debugging and error handling.

For example: 

```solidity
LUSDToken.sol:
311     function _transfer(address sender, address recipient, uint256 amount) internal {
312:    assert(sender != address(0));
313:    assert(recipient != address(0));
```

## [N-03] Values such as a call to keccak256() should used to `immutable` rather than `constant`

`constant` and `immutable` variables should each be used in their appropriate contexts: Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it is still best to use the right tool for the task.

```
ReaperVaultV2.sol (L73 - 76):
    bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
    bytes32 public constant ADMIN = keccak256("ADMIN");
```
```
ReaperBaseStrategyv4.sol (L49 - 52):
    bytes32 public constant KEEPER = keccak256("KEEPER");
    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
    bytes32 public constant ADMIN = keccak256("ADMIN");
```

## [N-04] `0 address` check for `asset`

```
ERC4626.sol (L32-38):
    constructor(
        ERC20 _asset,
        string memory _name,
        string memory _symbol
    ) public ERC20(_name, _symbol) {
        asset = _asset;
    }
```
As best practice, include the zero address check or alternatively - add some good NatSpec comments explaining what is valid and what is invalid and what are the implications of accidentally using an invalid address.

Recommended Mitigation Steps:
`if (_asset == address(0)) revert ADDRESS_ZERO();`

## [N-05] Use a more recent version of solidity

- Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining. 
- Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads. 
- Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require()strings and get bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>. 
- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value. 
- Use a solidity version of at least 0.8.12 to get string.concat() instead of abi.encodePacked(<str>,<str>)

Instance of old compiler version used: 
```
LUSDToken.sol (L3):
pragma solidity 0.6.11;
```
## [N-06] Avoid the use of sensitive terms in favour of neutral ones

Use allowlist rather than whitelist

```
CollateralConfig.sol
11:    * Houses whitelist of allowed collaterals (ERC20s) for the entire system. Also provides access to collateral-specific
37:    event CollateralWhitelisted(address _collateral, uint256 _decimals, uint256 _MCR, uint256 _CCR);
72:    emit CollateralWhitelisted(collateral, decimals, _MCRs[i], _CCRs[i]);
```

## Non-library/interface files should use fixed compiler versions, not floating ones

eg. 0.8.10 instead of ^0.8.10

For example: 
```
ReaperVaultERC4626.sol, ReaperVaultV2.sol:
L3:    pragma solidity ^0.8.0;
```

## [N-08] Best practice is to prevent signature malleability

Use OpenZeppelin’s ECDSA contract rather than calling ecrecover() directly.

```
LUSDToken.sol
287:    address recoveredAddress = ecrecover(digest, v, r, s);
```

## [N-09] Import exact functions
Noted that the exact functions are not imported in various contracts. It is best practice to import exact functions rather than just the whole contracts.

For example: 
```
LUSDToken.sol (L5 - 9):
import "./Interfaces/ILUSDToken.sol";
import "./Interfaces/ITroveManager.sol";
import "./Dependencies/SafeMath.sol";
import "./Dependencies/CheckContract.sol";
import "./Dependencies/console.sol";
```

```
HintHelpers.sol (L5 - 10): 
import "./Interfaces/ICollateralConfig.sol";
import "./Interfaces/ITroveManager.sol";
import "./Interfaces/ISortedTroves.sol";
import "./Dependencies/LiquityBase.sol";
import "./Dependencies/Ownable.sol";
import "./Dependencies/CheckContract.sol";
```

## [L-01] Include check to ensure lastReport time > activation time

In function addStrategy, there is no check to ensure that lastReport time > activation time. Having a case where lastReport time happens earlier than acitivation time is not aligned with the intention of the function and may impact related code. 

```
 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol (L144 - 171): 

function addStrategy(
        address _strategy,
        uint256 _feeBPS,
        uint256 _allocBPS
    ) external {
        _atLeastRole(DEFAULT_ADMIN_ROLE);
        require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
        require(_strategy != address(0), "Invalid strategy address");
        require(strategies[_strategy].activation == 0, "Strategy already added");
        require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");
        require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");
        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
        require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");

        strategies[_strategy] = StrategyParams({
            activation: block.timestamp,
            feeBPS: _feeBPS,
            allocBPS: _allocBPS,
            allocated: 0,
            gains: 0,
            losses: 0,
            lastReport: block.timestamp
        });

        totalAllocBPS += _allocBPS;
        withdrawalQueue.push(_strategy);
        emit StrategyAdded(_strategy, _feeBPS, _allocBPS);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L144-L171

## [L-02] Loss of precision due to rounding

```
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol:
237:     uint256 vaultMaxAllocation = (totalAllocBPS * balance()) / PERCENT_DIVISOR;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L237

## [L-03] Reentrancy Guard not initialized

In `ReaperVaultV2.sol`, openzepplin's reentrancy guard is imported but not initialized. 
