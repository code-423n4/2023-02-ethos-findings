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