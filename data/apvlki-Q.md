1. `updateCollateralRatios` needs additional protection in `CollateralConfig.sol`

Affected code:
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L85
```

Use of timelock or a two step control mechanism can be added when updating collateral ratios. this will ensure the finality of the change. 
