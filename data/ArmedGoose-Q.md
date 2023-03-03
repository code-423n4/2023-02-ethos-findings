## 1. `batchLiquidateTroves` does not check if single liquidations may fail in `TroveManager.sol`
### Issue:
The batch liquidation routine is constructed in a way, that subsequent borrowersArray members are used as an reference to call _liquidateNormalMode, as long as there are borrowers in the array. There is no check if all the borrowers are correct or if there are no duplicates in the array. 
It should work in a way, that even if some members of the array would cause a revert, it should not occur, just to process all the other liquidations. However, at the moment a revert could break the loop, burning caller's gas for operation that fails. 

### Recommendation:
Probably it would be better to call subsequent single liquidations in a loop with a low level call to not get a revert, and just collect information if it was successful or not

## 2. `prevewMint` and `previewDeposit` does not revert when in emergency mode in `ReaperVaultERC4626.sol`
### Issue: 
Currently, the application is protected from this issue because this routine is called only within other functions which have the revert implemented, but I understand if its public it might be planned to be used also as standalone function. Thus, when called in emergency mode, these functions will return some value, but in fact this will not be true, as one cannot execute these operations in emergency mode. 

### Recommendation: 
For accuracy, add an emergency mode check and if its active, return 0. 


