## QA Report

## N-01

Remove unnecessary type casting in the constructor parameters of ReaperVaultV2.sol

## Summary

In ReaperVaultV2.sol, the constructor takes several input parameters, including _name and _symbol, which are of type string. In the current implementation, these parameters are being unnecessarily cast to string in the constructor declaration, which has no effect on the code's functionality and readability.

Therefore, it is recommended to remove the unnecessary type casting in the constructor parameters of ReaperVaultV2.sol. 

[Link to the code on Github](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L119)

After making this change, the constructor should look like this:

```solidity
constructor(
    address _token,
    string memory _name,
    string memory _symbol,
    uint256 _tvlCap,
    address _treasury,
    address[] memory _strategists,
    address[] memory _multisigRoles
) ERC20(_name, _symbol) {
    // rest of the constructor code
}

```

## N-02

Use the `emit` keyword for events to improve code clarity.

## Summary

In older versions of Solidity, it was not mandatory to use the `emit` keyword when triggering events. However, this has changed in newer versions of Solidity, and it is now required. We recommend using the `emit` keyword in the following functions.

[Link to the code on Github](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L190-L202)

Current `increaseLUSDDebt` function:
```
    function increaseLUSDDebt(address _collateral, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveM();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount);
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
    }
```
change to :
```solidity
    function increaseLUSDDebt(address _collateral, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveM();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount);
        emit ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
    }
```

Current `decreaseLUSDDebt` function:

```solidity
    function decreaseLUSDDebt(address _collateral, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveMorSP();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].sub(_amount);
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
    }
```

change to:

```solidity
    function decreaseLUSDDebt(address _collateral, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveMorSP();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].sub(_amount);
        emit ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
    }
```
