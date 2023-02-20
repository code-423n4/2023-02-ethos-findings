G1. We can add some short-circuits below. For minutes = 1, 2, 3 how short circuits will win. 
```diff
function _decPow(uint _base, uint _minutes) public pure returns (uint) {
       
        if (_minutes > 525600000) {_minutes = 525600000;}  // cap to avoid overflow
    
        if (_minutes == 0) {return DECIMAL_PRECISION;}
+     if (base == DECIMAL_PRECISION) {return DECIMAL_PRECISION;}

+     if (_minutes == 1) return _base;
+     if(_minutes == 2) return decMul(_base, _base);
+     if(_minutes == 3) return decMul(decMul(_base, _base), _base);

        uint y = DECIMAL_PRECISION;
        uint x = _base;
        uint n = _minutes;

        // Exponentiation-by-squaring
        while (n > 1) {
            if (n % 2 == 0) {
                x = decMul(x, x);
                n = n / 2;
            } else { // if (n % 2 != 0)
                y = decMul(x, y);
                x = decMul(x, x);
                n = (n-1)/2
            }
        }

        return decMul(x, y);
  }
}
```

G2. We can check where ``index < lastIndex`` to save gas for function ``_removeTroveOwner()``:

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1339-L1357](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1339-L1357)

```diff
function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {
        Status troveStatus = Troves[_borrower][_collateral].status;
        // It’s set in caller function `_closeTrove`
        assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

        uint128 index = Troves[_borrower][_collateral].arrayIndex;
        uint length = TroveOwnersArrayLength;
        uint idxLast = length.sub(1);

        assert(index <= idxLast);

+     if(index < idxLast){
        address addressToMove = TroveOwners[_collateral][idxLast];

        TroveOwners[_collateral][index] = addressToMove;
        Troves[addressToMove][_collateral].arrayIndex = index;
        emit TroveIndexUpdated(addressToMove, _collateral, index);
+      }
        TroveOwners[_collateral].pop();
    }
```

