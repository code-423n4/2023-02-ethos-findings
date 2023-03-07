## 1. Use a more recent version of solidity if possible
 Version 0.6.11 is too old consider upgrading it that is possible to a newer solidity version as it will give `custom Error` benefits from `v0.8.4` and reduce gas usage for the users by a significant margin and has `safe math` check for under and overflow so there will be no need to use SafeMath library and reducing code size along with saving in the gas from `v0.8.0`

All the contracts in `Ethos-Vault` use floating pragma which in not recommended so using a version that is above v0.8.4 and using custom errors is recommended, as well as it is recommended using more recent version of solidity will be a better practice in general.


## 2. No need to inherits self interface and override it: 
The following contracts inherit themselves and override themselves that results in increase of bytecode
removing the interface from below contracts and removing override will optimize them to a significant extent
1. `Ethos-Core/contracts/CollateralConfig.sol`
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L8
2. `Ethos-Core/contracts/BorrowerOperations.sol`
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L5
`removing "override" of the below functions so that they are not taking a significant chunk in bytecode`
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L46
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L102
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L106
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L110
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L116
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L122

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L110
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L172
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L172
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L248
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L254
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L259
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L264
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L375
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L412
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L748

more contracts that follow this pattern `3. https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L6`, `4. https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L5`, `5. https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5`, `6. https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L6`, `7. https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L12`, `8. https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L5` , `9. ` refactoring them as above is recommended 
## 3.  Using AND (&&) operator for require blocks
Using AND (&&) operator for require blocks that are used multiple times in same function can drastically reduce gas and bytecode size.
The reason this works is because, in case of AND operator if one condition is false then there is no need to check for the other codition it saves the computation to check again in the next require and then revert.
Here, are few example of that
eg: 1`https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L529`
we can do the following code in a single line because in 
```solidity
require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");
require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");
```
### e.g. 1, implementation
```if(IERC20(IERC20(_collateral).balanceOf(_user) >= _collAmount && IERC20(_collateral).allowance(_user, address(this)) {revert ("BorrowerOperations: Insufficient collateral balance") }
```
eg: 2`https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L91` and 92 
require(_MCR <= config.MCR && _CCR <= config.CCR, "Can only walk down the MCR");
and followed by the same pattern in line 94 and 97 can use 
require(_MCRs[i] >= MIN_ALLOWED_MCR && _CCRs[i] >= MIN_ALLOWED_CCR, "MCR and CCR below allowed min"); to save gas
### e.g. 2, implementation
```solidity
uint256 collateralsLength= _collaterals.length
require(!initialized, "Can only initialize once");
require(collateralsLength != 0, "At least one collateral required");
require(_MCRs.length == collateralsLength && _CCRs.length == collateralsLength, "Array lengths must match");
```

More examples which can benefit from above methodology :

## 4. Redundant code components
The code that is inherited from `import "./Dependencies/CheckContract.sol";`
```solidity
function checkContract(address _account) internal view {
        require(_account != address(0), "Account cannot be zero address");

        uint256 size;
        // solhint-disable-next-line no-inline-assembly
        assembly { size := extcodesize(_account) }
        require(size > 0, "Account code size cannot be zero");
    }
```
is already present in some of the contracts because they `import "./Dependencies/SafeERC20.sol";` and SafeERC20 imports `import "./Address.sol";` which has a function that does the same task as above `checkContract()`, the function that can be used instead is 
```solidity
function isContract(address account) internal view returns (bool) {
        // This method relies on extcodesize, which returns 0 for contracts in
        // construction, since the code is only stored at the end of the
        // constructor execution.

        uint256 size;
        // solhint-disable-next-line no-inline-assembly
        assembly { size := extcodesize(account) }
        return size > 0;
    }
```
The reason this can be done is because both the functions check the size of the address passed and if the size is > 0 then both return the same boolean.
### Implementation:
You can similarly remove `import of import "./Dependencies/CheckContract.sol";` from below contracts:
`/Ethos-Core/contracts/CollateralConfig.sol`, `Ethos-Core/contracts/BorrowerOperations.sol`, `Ethos-Core/contracts/ActivePool.sol`, `Ethos-Core/contracts/StabilityPool.sol`, `Ethos-Core/contracts/LQTY/LQTYStaking.sol`, `Ethos-Vault/contracts/ReaperVaultERC4626.sol`
And 
You can safely change the checkContract() to isContract() and the code will behave the same way.

the imports on below lines are not needed since it has only one function and that is also present in LiquityMath
`https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L7`, `https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L5`, `https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L9`, `https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L9`, 


## 5. Variables do not have access modifiers defined
Consider making them private or public or internal
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L28
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L30
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L32

## 6. Contract is already in the state
There is no need to initialize the variable in the state  if the contract is initialized with the address, you can save 1 slot of space simply by typecasting `address(lqtyStaking)` and using this instead of `lqtyStakingAddress` whenever deemed necessary

## 7. Confusing function name/ error msg
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L533
The function name and error `BorrowerOperations: Cannot withdraw and add coll` was is confusing because the error msg has `cannot withdraw and add coll` and here it got confusing because I thought both the cases are supposed to be true for this function and it should use `&&` instead of `||`, so consider naming or adding a comment or at least changing the error msg to `BorrowOperations: Cannot withdraw or add coll`. The reason this was suggested is because in future audit if the team decides to get any, it will reduce some confusion and save some time.

## 8. Using more than 32 charactors of string takes more than one slot
When you use error message on reverts and the message is more than 32 charactors of english alphabets including space then the storage and gas are required to perform is a lot more and considering those functions are called multiple times but multiple users, that will cost a lot of gas and the Ethos team has most of the contracts and they have reverts in this manner. It can save tonn of gas if optimized. 
### Note: there are certain characters in ASCII that cost more than 1 byte for example `é` this charactor probably costs 2 bytes
### for example, The below line costs 2 slots of storage in the contract since it uses 50 bytes and 32 bytes will take one slot and move to the next
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L651
Here, the recommendation is to try to use the errors with less than 32 bytes and the best approach will be to use custom errors but that will require the contracts to be upgraded over `v0.8.4` since after `v0.8.4`  solidity introduced custom errors and they will save tonns in gas.

## 9. Interfaces are imported multiple times redundunt code 
The following imports 
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5
and
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L8
are imported twice, it doesn't actually do a change in code apart from not following solidity best practices.

## 10. Unexpected return on if a user calls
The function `revokeStrategy(address _strategy)` when called in the if block `https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L210` and the `allocPBS` is zero of the address _strategy which is called on the mapping strategies i.e.`https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L35`. It returns unexpectedly for the user
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L211