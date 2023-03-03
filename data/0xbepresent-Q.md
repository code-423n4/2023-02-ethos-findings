1 - Add a timelock to ```updateCollateralRatios()``` critical function
==

Changes in ```MCR``` and ```CCR``` values in the ```updateCollateralRatios()``` function can damage the borrower positions. It is a good practice to give time for users to react and adjust to critical changes.

Consider adding a timelock to:

- [updateCollateralRatios()](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L85)

2 - Resolve unnecessary dependency comment
==

The ```Ownable``` dependency is not used in the ```TroveManager.sol``` contract so it should be removed.

Instance of this issue:
- [Ethos-Core/contracts/TroveManager.sol#L14](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L14)

3 - The ```require()``` message error does not reflect the ```require()``` logic
==

The function can be called from ```redemptionHelper``` but the ```require()``` message error doesn't mention that.

Instance of this issue:
- [Ethos-Core/contracts/ActivePool.sol#L334](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L334)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L257](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L257)