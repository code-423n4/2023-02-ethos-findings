1 - The BorrowerOperations.sol::moveCollateralGainToTrove() function is not called by StabilityPool, it could be removed in order to save gas in the deploy
==

The ```moveCollateralGainToTrove()``` function [is only available for the StabilityPool](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L249) but the StabilityPool doesn't call that function, so the funcion can be removed in order to save gas in the deploy.

- [BorrowerOperations.sol::moveCollateralGainToTrove()](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L248)