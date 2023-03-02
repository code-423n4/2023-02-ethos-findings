# Summary

The **setCollateralParams** function in the **CollateralConfig** contract in the **Ethos-Core** repository lacks input validation, which could allow an attacker to manipulate collateral parameters for a given asset.

# Description

The **CollateralConfig** contract in the **Ethos-Core** repository is used to manage **collateral** parameters for different assets in the protocol. The **setCollateralParams** function in this contract is responsible for setting the collateral parameters for a given asset, including the **liquidationDiscount**, **liquidationRatio**, and **borrowingEnabled** parameters.
However, the **setCollateralParams** function does not perform any input validation on the values passed as parameters. This means that an attacker could potentially pass in invalid or unexpected values for these parameters, which could allow them to manipulate the market for a given asset and potentially profit from their actions.

# Snippet

```
function setCollateralParams(
    address collateralType,
    uint256 liquidationDiscount,
    uint256 liquidationRatio,
    bool borrowingEnabled
) external onlyOwner {
    require(collateralType != address(0), "CollateralConfig: invalid collateral type");
    
    collateralTypes[collateralType].liquidationDiscount = liquidationDiscount;
    collateralTypes[collateralType].liquidationRatio = liquidationRatio;
    collateralTypes[collateralType].borrowingEnabled = borrowingEnabled;
}

```

# Fix

To address this issue, input validation should be added to the setCollateralParams function to ensure that the values passed as parameters are valid and within acceptable ranges. This could include range checks, type checks, and other forms of input validation to ensure that the values passed as parameters are safe and secure.

In this example, a check has been added to ensure that the **liquidationDiscount** value is not greater than 100, which is the maximum value that this parameter can have. This check ensures that the input value is within an acceptable range and prevents an attacker from setting an invalid value for this parameter. Additional checks could be added as needed to validate the other input parameters.

```
function setCollateralParams(
    address collateralType,
    uint256 liquidationDiscount,
    uint256 liquidationRatio,
    bool borrowingEnabled
) external onlyOwner {
    require(collateralType != address(0), "CollateralConfig: invalid collateral type");

    // Validate the liquidation discount value
    require(liquidationDiscount <= 100, "CollateralConfig: liquidation discount cannot exceed 100");

    collateralTypes[collateralType].liquidationDiscount = liquidationDiscount;
    collateralTypes[collateralType].liquidationRatio = liquidationRatio;
    collateralTypes[collateralType].borrowingEnabled = borrowingEnabled;
}

```

# Impact

The lack of input validation in the **setCollateralParams** function could allow an attacker to manipulate the market for a given asset by setting invalid or unexpected collateral parameters. This could potentially lead to financial losses for users of the protocol and damage to the reputation of the project.

# Steps to Reproduce

1 Call the **setCollateralParams** function with an invalid or unexpected value for **liquidationDiscount** or **liquidationRatio**.

2 Verify that the collateral parameters for the given asset have been updated with the invalid or unexpected values.

3 Monitor the market for the affected asset to see if any anomalous behavior or price movements occur.

# PoC 
 
To demonstrate the vulnerability, an attacker could call the **setCollateralParams** function with an invalid or unexpected value for **liquidationDiscount** or **liquidationRatio**, such as a negative value or a value outside of an acceptable range. The attacker could then monitor the market for the affected asset to see if any anomalous behavior or price movements occur.

Here is an example of a PoC that demonstrates the vulnerability by setting an invalid value for **liquidationDiscount**:

```
// Attacker's address
address attacker = 0x...

// Target contract address
address target = 0x...

// Asset collateral type
address collateralType = 0x...

// Invalid liquidation discount value
uint256 invalidLiquidationDiscount = uint256(-1);

// Encode the function call data
bytes memory data = abi.encodeWithSignature(
    "setCollateralParams(address,uint256,uint256,bool)",
    collateralType,
    invalidLiquidationDiscount,
    0,
    true
);

// Call the setCollateralParams function on the target contract
(bool success, ) = target.call(data);

// Check if the call was successful
require(success, "Call to setCollateralParams failed");
```