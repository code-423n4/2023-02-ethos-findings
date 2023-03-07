# CheckContract optimization

CheckContract is used in many contracts to verify an address provided is indeed a contract, not just a random address or an EOA. This CheckContract contract is only contains a single `checkContract()` function which is a view function and could easily be implemented as an internal library call. This would result in slightly larger contract bytecode but should be far more gas efficient than an external contract call as is the current case.

If a smart contract is consuming a library which have only internal functions than EVM simply embeds library into the contract. Instead of using delegate call to call a function, it simply uses JUMP statement(normal method call). There is no need to separately deploy library in this scenario.

```js
File: CheckContract.sol
11:     function checkContract(address _account) internal view {
12:         require(_account != address(0), "Account cannot be zero address");
13:
14:         uint256 size;
15:         // solhint-disable-next-line no-inline-assembly
16:         assembly { size := extcodesize(_account) }
17:         require(size > 0, "Account code size cannot be zero");
18:     }
```

Further more, if Ethos decided to use Solidity v0.8 and above, we can just use `address.code.length` > 0 to detect if an address is a contract or just EOA, this will surely will save alot of gas which the current `checkContract()` function is being called everywhere.

# Remove `_scaleTellorPriceByDigits` function and just return the `_price` (its parameter)

In `PriceFeed.sol`, the `TARGET_DIGITS` and `TELLOR_DIGITS` are both a constant `18`.
By looking at this same value, then `_scaleTellorPriceByDigits` function is actually unnecessary, as it would just return the `_price` (it's parameter) since the conditional `if-else` statement will not being entered (nothing matched)

The `_scaleTellorPriceByDigits` function is being used in on of function which fetch a collateral price `fetchPrice()`, thus removing this function will save frequent gas consumption.

```js
File: PriceFeed.sol
38:     // Use to convert a price answer to an 18-digit precision uint
39:     uint constant public TARGET_DIGITS = 18;
40:     // legacy Tellor "request IDs" use 6 decimals, newer Tellor "query IDs" use 18 decimals
41:     uint constant public TELLOR_DIGITS = 18;
...
521:     function _scaleTellorPriceByDigits(uint _price) internal pure returns (uint) {
522:         uint256 price = _price;
523:         if (TARGET_DIGITS > TELLOR_DIGITS) {
524:             price = price.mul(10**(TARGET_DIGITS - TELLOR_DIGITS));
525:         } else if (TARGET_DIGITS < TELLOR_DIGITS) {
526:             price = price.div(10**(TELLOR_DIGITS - TARGET_DIGITS));
527:         }
528:         return price;
529:     }
```

# Use bytes32 instead of string to save gas whenever possible

bytes32 uses less gas because it fits in a single word of the EVM, and string is a dynamically sized-type which has current limitations in Solidity (such as can't be returned from a function to a contract).

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity. Basically, any fixed size variable in solidity is cheaper than variable size. That will save gas on the contract.

```js
    string constant public NAME = "ActivePool";
    string constant public NAME = "BorrowerOperations";
    string constant public NAME = "CollSurplusPool";
    string constant public NAME = "DefaultPool";
    string constant public NAME = "HintHelpers";
    string constant public NAME = "PriceFeed";
    string constant public NAME = "SortedTroves";
    string constant public NAME = "CommunityIssuance";
    string constant public NAME = "LQTYStaking";
    string constant public NAME = "BorrowerWrappersScript";
    string constant public NAME = "StabilityPoolScript";
    string constant public NAME = "TokenScript";
    string constant public NAME = "TroveManagerScript";
```

# Usage of assert() instead of require()

Most of the contracts use solc 0.6.11.
Between solc 0.4.10 and 0.8.0, `require()` used REVERT (0xfd) opcode which refunded remaining gas on failure while `assert()` used INVALID (0xfe) opcode which consumed all the supplied gas.
`require()` should be used for checking error conditions on inputs and return values while `assert()` should be used for invariant checking.
From the current usage of assert, my guess is that most of them can be replaced with require, unless a Panic really is intended.

# Flatten internal function which being use only once

These functions are being used only once thus it's better to flatten it to save some gas

- \_requireCallerIsStabilityPool
- \_requireValidLUSDRepayment
- \_requireNewICRisAboveOldICR
