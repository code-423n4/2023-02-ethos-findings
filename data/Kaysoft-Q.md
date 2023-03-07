## [L-01] REMOVE UNSED IMPORT `./Dependencies/console.sol` in `BorrowerOperations.sol`
console.sol is imported in `BorrowerOperations.sol` and it is not used.
File:
- [Ethos-Core/contracts/BorrowerOperations.sol#L15](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L15)

#### Recommended Mitigation Steps
Consider removing unused import of console.sol in `BorrowerOperations.sol`

## [L-02] UNUSED VARIABLES.
Remove all unused variables.
Files: 
- [Ethos-Core/contracts/StabilityPool.sol#L384](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L384)
- [Ethos-Core/contracts/StabilityPool.sol#L339](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L339)

## [L-03] USE LATEST COMPILER STABLE VERSION
The EThos-Core project uses the Solidity version 0.6.11 while the Ethos-Vault version uses the version 0.80. It is best practice to use the latest Solidity stable version as there will have been some security bug fixes and updates.
see: https://swcregistry.io/docs/SWC-102

Files: All files

#### Recommended Mitigation Step
Consider using latest Solidity stable compiler version 0.8.17 for all the contracts.

## [L-04] AVOID FLOATING PRAGMA
Consider locking compiler versions by using `pragma solidity 0.8.17;` instead of `pragma solidity ^0.8.0;` in order not to deploy the contract with a different compiler version that is used to test the contract thereby introducing compiler bugs.
see: https://swcregistry.io/docs/SWC-103
Files: 
All files in the Ethos-Vault project

#### Recommended Mitigation Steps
Consider locking the pragma version by replacing `pragma solidity ^0.8.0;` with `pragma solidity 0.8.17;`

## [L-05] ENABLING SOLIDITY COMPILER OPTIMIZATION CAN BE BUGGY
The compiler optimization is enabled in the project since it is set to true. Enabling compiler optimization can lead to security issues.

```Solidity
/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: {
    compilers: [
      {
        version: "0.8.11",
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
        },
      },
    ],
  },

```

#### Recommended Mitigation Steps
Consider the amount of gas savings with this settings and weigh the tradeoff of not enabling the compiler optimization.

## [L-7] CRITICAL ADDRESS CHANGES SHOULD FOLLOW 2STEP PROCESS
The ActivePool.sol smart contract inherits the Ownable contract. The Ownable smart contract do not have 2 step transfer of ownership which can lead to irecoverable mistake when new owner is set with a wrong address. Consider using Openzeppelin's Ownable2step contract instead of the Ownable contract.
see: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4fb6833e325658946c2185862b8e57e32f3683bc/contracts/access/Ownable2Step.sol#L32

```
 /**
     * @dev Starts the ownership transfer of the contract to a new account. Replaces the pending transfer if there is one.
     * Can only be called by the current owner.
     */
    function transferOwnership(address newOwner) public virtual override onlyOwner {
        _pendingOwner = newOwner;
        emit OwnershipTransferStarted(owner(), newOwner);
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`) and deletes any pending owner.
     * Internal function without access restriction.
     */
    function _transferOwnership(address newOwner) internal virtual override {
        delete _pendingOwner;
        super._transferOwnership(newOwner);
    }

    /**
     * @dev The new owner accepts the ownership transfer.
     */
    function acceptOwnership() public virtual {
        address sender = _msgSender();
        require(pendingOwner() == sender, "Ownable2Step: caller is not the new owner");
        _transferOwnership(sender);
    }
```


## [NC-1] USE NAMED IMPORTS
It is recommended to use specific named import like `import {MyContract} from ./Files/Mycontract.sol` instead of the global imports `import ./Files/MyContract.sol`
Both the EThos-Core and Ethos-Vault use the global imports which is not recommended because it makes it difficult to quickly figure out where modules are defined.

Files: All files

## [NC-2] LACK OF NATSPEC COMMENTS
Most of the functions, contructors in Ethos-core and Ethos-Vault do not have Natspec commments.
It is recommended by the Solidity Documentation that all Smart contracts are anotated using the NatSpec comments for all public functions.

see: [Natspec Solidity Docs](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html#:~:text=Solidity%20contracts%20can%20use%20a,Language%20Specification%20Format%20(NatSpec).&text=NatSpec%20was%20inspired%20by%20Doxygen.)
Files: Most functions of all files.

## [NC-3] USE LATEST STABLE VERSION OF OPENZEPPELIN LIBRARY
The Ethos-Core project used openzeppelin version 3.3.0 and the latest version of openzeppelin contract is 4.8.2

File: Ethos-core/package.json

```Solidity
- package.json
"@openzeppelin/contracts": "^3.3.0"
```

## [NC-4] ADD `rescueERC20` FUNCTION TO THE `ReaperVaultV2` SMART CONTRACT IN ORDER TO RECOVER ANY ERC20 TOKEN MISTAKENLY SENT TO THE CONTRACTS ADDRESS.

```
/**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}
``` 


## [NC-5] USE SCIENTIFIC NOTATION FOR POWER OF 10 NUMBER LITERALS
USE 1e18 instead of 10 ** 18;

File: 
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L40](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L40)

```
uint256 public constant DEGRADATION_COEFFICIENT = 10 ** 18;
```

## [NC-6] USE UNDERSCORE FOR NUMBER LITERALS TO AID READABILITY
Use 10_000 instead of 10000

File: 
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L41](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L41)

```
41: uint256 public constant PERCENT_DIVISOR = 10000;
```
## [NC-7] ADD MAXIMUM CHARACTER PER LINE LIMIT
The Solidity documentation style guide recommends 120 characters as maximum number of characters per line. Some lines exceed the recommended maximum number of characters per line. 
see: https://docs.soliditylang.org/en/latest/style-guide.html#maximum-line-length

File:
- [Ethos-Core/contracts/ActivePool.sol#L47](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L47)