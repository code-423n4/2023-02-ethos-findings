# Collateral config didn't check duplicate (or if collateral alread exist in list) when `collaterals.push(collateral);`

In `CollateralConfig.sol` the `initialize` function doesn't really check if collateral which currently being pushed to array is not duplicates.

Eventhough the project owner mention currently only two collateral asset token which is planned as the collateral, WBTC and WETH, it's best to include the check to minimize the duplicate of collateral

# Critical function like `updateChainlinkAggregator()` is recommend to have timelock to prevent any abuse / mistakes

`updateChainlinkAggregator()` can be a potential issue when it's mistakenly or unauthorized changes by the owner. Currently if the `owner` is compromized, and attacker can change this price aggregator, this would be a major issue. So, giving a timelock on this function can help atleast prevent it.

```js
File: PriceFeed.sol
140:     // Admin function to update the Chainlink aggregator address for a particular collateral.
141:     //
142:     // !!!PLEASE USE EXTREME CARE AND CAUTION!!!
143:     function updateChainlinkAggregator(
144:         address _collateral,
145:         address _priceAggregatorAddress
146:     ) external onlyOwner {
147:         _requireValidCollateralAddress(_collateral);
148:         checkContract(_priceAggregatorAddress);
149:         priceAggregator[_collateral] = AggregatorV3Interface(_priceAggregatorAddress);
150:
151:         // Explicitly set initial system status
152:         status[_collateral] = Status.chainlinkWorking;
153:
154:         // Get an initial price from Chainlink to serve as first reference for lastGoodPrice
155:         ChainlinkResponse memory chainlinkResponse = _getCurrentChainlinkResponse(_collateral);
156:         ChainlinkResponse memory prevChainlinkResponse = _getPrevChainlinkResponse(_collateral, chainlinkResponse.roundId, chainlinkResponse.decimals);
157:
158:         require(!_chainlinkIsBroken(chainlinkResponse, prevChainlinkResponse) && !_chainlinkIsFrozen(chainlinkResponse),
159:             "PriceFeed: Chainlink must be working and current");
160:
161:         _storeChainlinkPrice(_collateral, chainlinkResponse);
162:     }
```

# Unbounded distribution period in `CommunityIssuance`

The `distributionPeriod` is currently not being bounded with minimum and maximum value of it, which can have a wide range of possibility. It can have 0 distributionPeriod or it can have maximum uint256 value (which will never be happen, in our timespan).

So, it's best to set this bound of value.

```js
File: CommunityIssuance.sol
120:     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
121:         distributionPeriod = _newDistributionPeriod;
122:     }
```

# Unnecessary `_scaleTellorPriceByDigits` because of constants

As the `TARGET_DIGITS` and `TELLOR_DIGITS` are both a constant `18`, then the `_scaleTellorPriceByDigits` function is actually unnecessary, as it would just return the `_price` since the conditional `if-else` statement will not being entered.

The reason why the code exist is because Ethos forks the Liquity, and on Liquity the function exist because of different `TARGET_DIGITS` and `TELLOR_DIGITS`. But since the contract is not upgradeable, so removing the function for Ethos project is understandable.

Further more the `_scaleTellorPriceByDigits` function is being used in on of function which fetch a collateral price `fetchPrice()`, thus removing this function will save frequent gas consumption.

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

# One of mostly used function, `checkContract()` can be replaced

If using Solidity v0.8 and above, we can use `address.code.length` > 0 to detect if an address is a contract or just EOA.

```js
pragma solidity >=0.8.0;

contract IsContract {

    constructor(){}

    function checkContract(address _account) internal view {
        require(_account != address(0), "Account cannot be zero address");

        uint256 size;
        // solhint-disable-next-line no-inline-assembly
        assembly { size := extcodesize(_account) }
        require(size > 0, "Account code size cannot be zero");
    }

    function checkContractNew(address _account) view returns (bool) {
        require(_account.code.length > 0)
    }
}
```

# Critical Address Changes Should Use Two-step Procedure

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure (transfer and accept) on the critical functions.

The critical function or procedures which should be in two step process inside `LUSDToken.sol`, which I noticed are:

```js
File: LUSDToken.sol
146:     function updateGovernance(address _newGovernanceAddress) external {
147:         _requireCallerIsGovernance();
148:         checkContract(_newGovernanceAddress); // must be a smart contract (multi-sig, timelock, etc.)
149:         governanceAddress = _newGovernanceAddress;
150:         emit GovernanceAddressChanged(_newGovernanceAddress);
151:     }
152:
153:     function updateGuardian(address _newGuardianAddress) external {
154:         _requireCallerIsGovernance();
155:         checkContract(_newGuardianAddress); // must be a smart contract (multi-sig, timelock, etc.)
156:         guardianAddress = _newGuardianAddress;
157:         emit GuardianAddressChanged(_newGuardianAddress);
158:     }
```

# Signature Malleability of EVM’s `ecrecover()`

In `LUSDToken.sol` exist `permit()` function which contains `ecrecover()`. This `ecrecover()` function returns an address of zero when the signature does not match.

The `permit` function which calls the Solidity `ecrecover()` function directly to verify the given signatures. However, the `ecrecover()` EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this LUSDToken contract, I recommend using the battle-tested OpenZeppelin’s ECDSA library.

# Use safeTransfer/safeTransferFrom consistently instead of transfer/transferFrom

A transfer function can fail without reverting, and the return values are not properly checked, the contract may assume a transfer succeeded and end up in a bad state with inaccurate assumptions about the value held in different locations. To prepare for such situations, a contract should either check the return value of the transfer function or use a solution such as Open Zeppelin's SafeTransfer function. The SafeTransfer or SafeTransferFrom are used in the Unipool contracts, but not in other locations. While some of the uses of transfer check the return value, not all do.

```js
File: CommunityIssuance.sol
103:         OathToken.transferFrom(msg.sender, address(this), amount);
...
127:         OathToken.transfer(_account, _OathAmount);

File: LQTYStaking.sol
135:             lusdToken.transfer(msg.sender, LUSDGain);
...
171:         lusdToken.transfer(msg.sender, LUSDGain);

```

# No Reentrancy Guard in BorrowerOperations & TroveManager

There are some external function, user-facing function which potentially introduce reentrancy

* BorrowerOperations.sol (openTrove, closeTrove, addColl, and others which basicall call the `_adjustTrove` which potentially impacts user debt, collateral top-ups or withdrawals)
* TroveManager.sol (redeemCollateral, and other that invokes `_applyPendingRewards` which potentially impacts collateral and debt rewards from redistributions.)

The Ethos protocol makes no use of reentrancy guards in those files which can externally be called by any user. Our recommendation is to use of the Open Zeppelin ReentrancyGuard.sol

# Wrong comment information

The Ethos core is a fork of Liquity project, with multiple collateral token as their collateral (unlike Liquity which use native ETH's coin).

There are still comments using this `ETH` which should be replaced with `collaterals`:

- // Calculate the ETH fee,
- // Update Active Pool LUSD, and send ETH to account
- // Update Active Pool LUSD, and send ETH to account
- // send ETH from Active Pool to CollSurplus Pool

# Not using latest solidity

All Ethos-core contracts which is a fork of Liquity is using `pragma solidity 0.6.11;` which is outdated version of solidity.

It's best to keep up with latest version and use the recent code (for example, checking if address is contract, `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)).

# Not using named import

All import using plain `import 'file.sol`' instead use named import.

# unused console.log

cleanup `console.log` as it doesn't really have effect in production.

# Any changes to state variable is best to Emit events

Some of function which best to trigger an event and currently not emitting it when called are:

- updateTellorCaller
- updateDistributionPeriod

# Commented code, TroveManager

Remove unnecessary commented code:

- TroveManager, remove the `Ownable` contract commented code to cleanup the source code

# Flatten internal function which being use only once

These functions are being used only once thus it's better to flatten it.

- \_requireCallerIsStabilityPool
- \_requireValidLUSDRepayment
- \_requireNewICRisAboveOldICR
