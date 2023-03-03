## Use a more recent version of solidity
The protocol adopts version 0.6.11 on writing contracts. For better security, it is best practice to use the latest Solidity version, 0.8.17.

Security fix list in the versions can be found in the link below:

https://github.com/ethereum/solidity/blob/develop/Changelog.md

## Modularity on import usages
For cleaner Solidity code in conjunction with the rule of modularity and modular programming, use named imports with curly braces instead of adopting the global import approach.

For instance, the import instances below could be refactored conforming to most other instances in the code bases as follows:

[File: CollateralConfig.sol#L5-L8](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L5-L8)

```diff
- import "./Dependencies/CheckContract.sol";
+ import {CheckContract} from "./Dependencies/CheckContract.sol";
- import "./Dependencies/Ownable.sol";
+ import {Ownable} from "./Dependencies/Ownable.sol";
- import "./Dependencies/SafeERC20.sol";
+ import {SafeERC20} from "./Dependencies/SafeERC20.sol";
- import "./Interfaces/ICollateralConfig.sol";
+ import {ICollateralConfig} from "./Interfaces/ICollateralConfig.sol";
```
## Inadequate NatSpec
Solidity contracts can use a special form of comments, i.e., the Ethereum Natural Language Specification Format (NatSpec) to provide rich documentation for functions, return variables and more. Please visit the following link for further details:

https://docs.soliditylang.org/en/v0.8.16/natspec-format.html

Consider equipping all contracts with complete set of NatSpec to better facilitate users/developers interacting with the protocol's smart contracts.

## Return statement on named returns

Functions with named returns should not have a return statement to avoid unnecessary confusion.

For instance, the following `_getTCR()` may be refactored as follows:

```diff
    function _getTCR(address _collateral, uint _price, uint256 _collateralDecimals) internal view returns (uint TCR) {
        uint entireSystemColl = getEntireSystemColl(_collateral);
        uint entireSystemDebt = getEntireSystemDebt(_collateral);

        TCR = LiquityMath._computeCR(entireSystemColl, entireSystemDebt, _price, _collateralDecimals);

-        return TCR;
    }
``` 
