<h2>(01)Misleading function name<h2/>
<h4>Vulnerability Detail<h4/>
 the function name _requireCallerIsBOorTroveMorSP() is misleading. This function requires the caller to be borrowerOperationsAddress or troveManagerAddress or stabilityPoolAddress, yet there was a check for the redemptionHelper 
this is the highlighted code:https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#:~:text=function%20_requireCallerIsBOorTroveMorSP(),%7D

This could result to an irregular behaviour if redemptionHelper isn't supposed to access the functions in which this function _requireCallerIsBOorTroveMorSP is called, since this function will now be able to sendCollateral and decrease decreaseLUSDDebt
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#:~:text=function%20sendCollateral(,_requireCallerIsBOorTroveMorSP()%3B
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#:~:text=function%20decreaseLUSDDebt(,_requireCallerIsBOorTroveMorSP()%3B

<h4>Recommendation</h4>
If the require statement for redemtionHelper wasn't a mistake, then I suggest the name of the function _requireCallerIsBOorTroveMorSP() be changed to a more descriptive function name. i.e _requireCallerIsBORorTROVorRHorSP() and the require message be changed to "ActivePool: Caller is not BorrowerOperations, TroveManager, RedemptionHelper, or StabilityPool".
If it was a mistake, then the redemtionHelper should be removed


<h2>(02)Misleading function name<h2/>
<h4>Vulnerability Detail<h4/>
The same issue described above is still present here: https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#:~:text=function%20_requireCallerIsTroveManagerOrActivePool(),TroveM%20or%20ActivePool%22

will suggest the function name _requireCallerIsTroveManagerOrActivePool() be changed to _requireCallerIsTroveMOrActivePOrRemdempH()
and the require message be changed to "LQTYStaking: caller is not TroveM or ActivePool or RedemptionHelper" if the naming was a mistake, if that wasn't the case, let the RedemptionHelper declaration be removed