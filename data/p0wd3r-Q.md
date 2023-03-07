# The liquidateTroves function in TroveManager does not verify if the Trove is active.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L513


Compared to the `liquidate` function, the `liquidateTroves` function is also an external function and does not check if the Trove is active, which could lead to unexpected issues.

```
function liquidate(address _borrower, address _collateral) external override {
        _requireTroveIsActive(_borrower, _collateral);
        ...
}
```

# Add checkContract for RedemptionHelper

Use `checkContract` to verify the correctness of the `setAddresses` function in RedemptionHelper, just like other contracts such as ActivePool.


