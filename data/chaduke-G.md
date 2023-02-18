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