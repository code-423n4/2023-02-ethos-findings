Using && operators inside require statements contribute to a higher execution cost as it tests both sides of the AND operation before reverting.

This means that every time we find && statements we should think about which one of the predicaments is the most probable fail cause for the specific function. As seen on `TroveManager.sol`:

```
function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {
        require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
    }
```
Could be replaced by:

```
function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) external view {
       require (TroveOwnersArrayLength > 1);
       require (getSize(_collateral) > 1);
   } 
```

This way, in case TroveOwnersArrayLength is <= 1, we revert the execution before accessing the `data` map. I've developed a oversimplified example contract to illustrate this issue:

```
// SPDX-License-Identifier: MIT
// Creator: gabriel

pragma solidity ^0.8.4;


contract Example {

    // Information for the list
    struct Data {
        uint256 size;                        // Current size of the list
    }

    // Each collateral has its own list of sorted troves
    mapping (address => Data) public data;

    function addToList(address _collateral) external {
        data[_collateral].size += 1;
    }

    function getSize(address _collateral) public view returns (uint256) {
        return data[_collateral].size;
    }

    function _requireMoreThanOneTroveInSystem1(uint TroveOwnersArrayLength, address _collateral) external view {
        require (TroveOwnersArrayLength > 1);
        require(getSize(_collateral) > 1);
    }
   function _requireMoreThanOneTroveInSystem2(uint TroveOwnersArrayLength, address _collateral) external view {
       require (TroveOwnersArrayLength > 1);
       require (getSize(_collateral) > 1);
   } 
}
```

In the contract above, calling the _requireMoreThanOneTroveInSystem1 function with `TroveOwnersArrayLength == 1` costs 621 gas, while using _requireMoreThanOneTroveInSystem2 costs 641 gas.

Additionally, the same behavior was also found in:
* `_requireValidMaxFeePercentage in BorrowerOperations.sol` 
* `_requireValidRecipient in LUSDToken.sol`
