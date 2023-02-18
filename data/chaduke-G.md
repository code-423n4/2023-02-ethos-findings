G1. We can add another case (base = DECIMAL_PRECISION) as a short-cuircuit:
```diff
function _decPow(uint _base, uint _minutes) public pure returns (uint) {
       
        if (_minutes > 525600000) {_minutes = 525600000;}  // cap to avoid overflow
    
        if (_minutes == 0) {return DECIMAL_PRECISION;}
+     if (base == DECIMAL_PRECISION) {return DECIMAL_PRECISION;}

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