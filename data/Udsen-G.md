## 1. `keccak256()` EXPRESSIONS WHICH ARE FIXED, CAN BE PRECALCULATED AND ASSIGNED TO THE CONSTANT VARIABLES.

Instead of calculating a fixed string with keccak256() every time the contract is made, pre-calculate them before and only give the result to a constant.

    bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
    bytes32 public constant ADMIN = keccak256("ADMIN");

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73-L76
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49-L52

## 2. `require()/revert()` STRINGS LONGER THANT 32bytes COST EXTRA GAS

Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas	

        require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
		
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L150
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L436
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L98
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L191-L194

## 3. USE CUSTOM ERRORS RATHER THAN `revert()/require()` STRINGS TO SAVE GAS

Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they’re hit by avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas.

        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
        require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");
		
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155-L156
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L436
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L568
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L98
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L191-L194

## 4. `require()` OR `revert()` STATEMENTS THAT CHECK INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
        require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");
		
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L150-L156
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L321-L322

## 5. USE ASSEMBLY TO CHECK FOR `address(0)` THIS WILL SAVE 6 GAS PER INSTANCE

        require(_strategy != address(0), "Invalid strategy address");
		
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L151
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L629

## 6. ADD `unchecked {}` FOR ADDITIONS AND SUBTRACTIONS WHERE THE OPERANDS CANNOT OVERFLOW OR UNDERFLOW RESPECTIVELY, BECAUSE OF A PREVIOUS `require()` OR `if`-STATEMENT

        require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");
        totalAllocBPS += _allocBPS;
		
above code line can be changed as below:

		unchecked {totalAllocBPS += _allocBPS};

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L156-L168
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L214
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L99-L100
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L131-L132
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L122-L123

## 7. CAN MAKE THE VARIABLE OUTSIDE THE `for` LOOP TO SAVE GAS

Consider making the stack variables before the `for` loop which is will save gas.

            for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
                vaultBalance = token.balanceOf(address(this));
                if (value <= vaultBalance) {
                    break;
                }

                address stratAddr = withdrawalQueue[i]; 
                uint256 strategyBal = strategies[stratAddr].allocated; 
                if (strategyBal == 0) {
                    continue;
                }

                uint256 remaining = value - vaultBalance;
                uint256 loss = IStrategy(stratAddr).withdraw(Math.min(remaining, strategyBal)); 
                uint256 actualWithdrawn = token.balanceOf(address(this)) - vaultBalance;
				
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L372-L386
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L120
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L166

## 8. CALLDATA VARIABLE SHOULD BE USED TO EMIT THE EVENT INSTEAD OF USING THE STATE VARIABLE.

It is gas efficient to use the calldata variable as the argument to `TvlCapUpdated` event instead of using the state variable. 
This will save an extra `Gcoldsload - 2100 gas`.

    function updateTvlCap(uint256 _newTvlCap) public {
        _atLeastRole(ADMIN);
        tvlCap = _newTvlCap;
        emit TvlCapUpdated(tvlCap);
    }
	
The `emit TvlCapUpdated(tvlCap)` event can be coded as follows to save gas:

        emit TvlCapUpdated(_newTvlCap);
		
`_newTvlCap` is calldata variable which is passed into the function as an input parameter.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L578-L582

## 9. USE A MORE RECENT VERSION OF SOLIDITY

Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining

Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads

Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings

Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value

	pragma solidity ^0.8.0;
	
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3

##10. INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

The function `_deposit` is an `internal` function and is called only once with in the contract. Hence this function can be inlined to save gas.

    function _deposit(uint256 toReinvest) internal { 
        if (toReinvest != 0) { 
            address lendingPoolAddress = ADDRESSES_PROVIDER.getLendingPool();
            IERC20Upgradeable(want).safeIncreaseAllowance(lendingPoolAddress, toReinvest);
            ILendingPool(lendingPoolAddress).deposit(want, toReinvest, address(this), LENDER_REFERRAL_CODE_NONE);
        }
    }
	
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176-L182
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L187-L191