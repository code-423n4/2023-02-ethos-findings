#1 CollateralConfig.sol – No need for the initialized variable 
The initialized state variable is not really necessary since the contract can simply check whether collaterals.length is greater than zero to determine if initialization has occurred. This saves gas by avoiding an unnecessary state variable.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L51
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L75

Solution:
require(collaterals.length == 0, "Already initialized");

#2 CollateralConfig.sol – initialize function should be converted to a constructor to save gas 
A constructor is a special function that is executed only once during the deployment of a contract. It is used to initialize the contract's state variables and can receive arguments. The constructor is part of the bytecode of the contract and cannot be called after the contract has been deployed.
On the other hand, an initializer function is a regular function that is used to initialize the state variables of a contract. This function can only be called once, typically after the contract has been deployed. The purpose of an initializer function is to allow for the possibility of upgrading a contract by modifying its state variables after deployment.
Using a constructor to initialize state variables can save gas compared to using an initializer function. This is because the constructor is executed as part of the deployment process and its code is included in the bytecode of the contract. This means that the gas cost of executing the constructor is only paid once during deployment. On the other hand, an initializer function must be called separately from the deployment process, which incurs additional gas costs.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#LL46C5-L76C6

#3 Heavy use of require() through out the agreements 
With the introduction of Solidity 0.8.4, custom errors can now be used instead of revert strings, resulting in more gas-efficient contracts.

#4 More than 32 bytes in require statement costs more gas 
Require statement can easily shorten to stay within the 32 byte limit and nor incur unnecessary gas
File: ActivePool.sol Line 107: 52 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L107
Vaults array length must match number of collaterals
File: ActivePool.sol Line 133: 36 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L133
Exceeds max allowed value of 500 BPS
File: BorrowerOperations.sol Line 525: 39 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L525
BorrowerOps: Invalid collateral address
File: BorrowerOperations.sol Line 529: 56 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L529
BorrowerOperations: Insufficient user collateral balance
File: BorrowerOperations.sol Line 530: 53 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L530
BorrowerOperations: Insufficient collateral allowance
File: BorrowerOperations.sol Line 534: 48 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L534
BorrowerOperations: Cannot withdraw and add coll
File: BorrowerOperations.sol Line 538: 70 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L538
BorrowerOps: There must be either a collateral change or a debt change
File: BorrowerOperations.sol Line 543: 46 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L543
BorrowerOps: Trove does not exist or is closed
File: BorrowerOperations.sol Line 552: 55 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L552
BorrowerOps: Debt increase requires non-zero debtChange
File: BorrowerOperations.sol Line 568: 62 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L568
BorrowerOps: Collateral withdrawal not permitted Recovery Mode
File: BorrowerOperations.sol Line 617: 73 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L617
BorrowerOps: An operation that would result in ICR < MCR is not permitted
File: BorrowerOperations.sol Line 621: 55 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L621
BorrowerOps: Operation must leave trove with ICR >= CCR
File: BorrowerOperations.sol Line 625: 62 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L625
BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode
File: BorrowerOperations.sol Line 629: 73 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L629
BorrowerOps: An operation that would result in TCR < CCR is not permitted
File: BorrowerOperations.sol Line 637: 67 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L637
BorrowerOps: Amount repaid must not be larger than the Trove's debt
File: BorrowerOperations.sol Line 641: 41 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L641
BorrowerOps: Caller is not Stability Pool
File: BorrowerOperations.sol Line 645: 61 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L645
BorrowerOps: Caller doesnt have enough LUSD to make repayment
File: BorrowerOperations.sol Line 653: 48 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653
Max fee percentage must be between 0.5% and 100%
File: CommunityIssuance.sol Line 133: 35 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L133
CommunityIssuance: caller is not SP
File: LQTYStaking.sol Line 262: 38 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L262
LQTYStaking: caller is not BorrowerOps
File: LQTYStaking.sol Line 266: 44 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L266
LQTYStaking: User must have a non-zero stake  
File: LQTYStaking.sol Line 270: 36 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L270
LQTYStaking: Amount must be non-zero
File: LUSDToken.sol Line 362: 43 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L362
LUSDToken: Caller is not BorrowerOperations
File: LUSDToken.sol Line 377: 37 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L377
LUSD: Caller is not the StabilityPool
File: LUSDToken.sol Line 390: 35 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L390
LUSDToken: Caller is not Governance
File: LUSDToken.sol Line 394: 38 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L394
LUSDToken: Minting is currently paused
File: ReaperBaseStrategyv4.sol Line 98: 33 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/ReaperBaseStrategyv4.sol#L98
Ammount must be less than balance
File: ReaperVaultV2.sol Line 150: 45 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L150
Cannot add strategy during emergency shutdown
File: ReaperVaultV2.sol Line 321: 40 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L321
Cannot deposit during emergency shutdown
File: ReaperVaultV2.sol Line 436: 47 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L436
Strategy loss cannot be greater than allocation
File: ReaperVaultV2.sol Line 619: 36 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L619
Degradation cannot be more than 100%
File: StabilityPool.sol Line 849: 39 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L849
StabilityPool: Caller is not ActivePool
File: StabilityPool.sol Line 853: 41 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L853
StabilityPool: Caller is not TroveManager
File: StabilityPool.sol Line 865: 68 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L865
StabilityPool: Cannot withdraw while there are troves with ICR < MCR
File: StabilityPool.sol Line 870: 48 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L870
StabilityPool: User must have a non-zero deposit
File: StabilityPool.sol Line 874: 38 bytes 
 https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L874
StabilityPool: Amount must be non-zero

#5 Pack struct items in descending order of size to save gas 
Packing the struct in descending order of size of its members can save gas by reducing the amount of storage needed for the struct. This is because the Solidity compiler will automatically pack the struct members into the smallest number of storage slots possible, based on their size and alignment requirements.

By ordering the members in descending order of size, we ensure that the largest members are packed first, reducing the likelihood of wasted storage space. This can also reduce the number of storage slots needed for the struct, which can save gas costs for operations that read or write the struct to storage.
Implementing the practice of evaluating the most appropriate byte size of each item and ordering them in descending order can lead to significant gas savings in smart contract development. By organizing the struct members in this manner, the compiler can pack them more efficiently, allowing for less memory to be used and reducing the overall gas cost of executing the contract.
For example, consider the LocalVariables_getSingularCollateralGain struct in the Stability Pool contract. Prior to reordering the members, the struct is defined as follows:
struct LocalVariables_getSingularCollateralGain {
        uint256 collDecimals;
        uint128 epochSnapshot;
        uint128 scaleSnapshot;
        uint P_Snapshot;
        uint S_Snapshot;
        uint firstPortion;
        uint secondPortion;
        uint gain;
}
For sake of simplicity, without entering into the merit of the most appropriate byte size of each item of the struct, by reordering the members in descending order, we can improve the efficiency of the struct and save gas. The reordered struct would look like this:
struct LocalVariables_getSingularCollateralGain {
        uint256 collDecimals;
        uint256 gain;
        uint256 secondPortion;
        uint256 firstPortion;
        uint256 S_Snapshot;
        uint256 P_Snapshot;
        uint128 scaleSnapshot;
        uint128 epochSnapshot;
}
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L646

#6 Use ++i instead of i++ and could be uncheck to save gas in certain cases
In terms of gas cost, using ++i instead of i++ or i += 1 is more efficient for unsigned integers. This is because pre-incrementing (++i) is cheaper per iteration, even with the optimizer enabled. When using i++, the value of i is incremented but the initial value is returned. On the other hand, ++i increments i and returns the new value. Therefore, it is recommended to use ++i when possible to optimize gas usage. 
Moreover, ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops

Instances:
File: ActivePool.sol | Line: 108 | for(uint256 i = 0; i < numCollaterals; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108
File: LQTYStaking.sol | Line: 206 | for (uint i = 0; i < assets.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206
File: LQTYStaking.sol | Line: 228 | for (uint i = 0; i < collaterals.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228
File: LQTYStaking.sol | Line: 240 | for (uint i = 0; i < numCollaterals; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240
File: StabilityPool.sol | Line: 351 | for (uint i = 0; i < numCollaterals; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351
File: StabilityPool.sol | Line: 397 | for (uint i = 0; i < numCollaterals; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L397
File: StabilityPool.sol | Line: 640 | for (uint i = 0; i < assets.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640
File: StabilityPool.sol | Line: 810 | for (uint i = 0; i < collaterals.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810
File: StabilityPool.sol | Line: 831 | for (uint i = 0; i < collaterals.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831
File: StabilityPool.sol | Line: 859 | for (uint i = 0; i < numCollaterals; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L859
File: TroveManager.sol | Line: 608 | for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608
File: TroveManager.sol | Line: 690 | for (vars.i = 0; vars.i < _n; vars.i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690
File: TroveManager.sol | Line: 808 | for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808
File: TroveManager.sol | Line: 882 | for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882

