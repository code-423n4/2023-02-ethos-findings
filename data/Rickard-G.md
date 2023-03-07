# [G-01] `bytes constant` are more efficient than `string constant`

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L32-L34](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L32-L34)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23)
## Vulnerability details
If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30)
````solidity
	string constant public NAME = "ActivePool";
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21) 
````solidity
	string constant public NAME = "BorrowerOperations";
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L32-L34](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L32-L34)
````solidity
	string constant internal _NAME = "LUSD Stablecoin";
	string constant internal _SYMBOL = "LUSD";
	string constant internal _VERSION = "1";
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150) 
````solidity
	string constant public NAME = "StabilityPool";
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19) 
````solidity
	string constant public NAME = "CommunityIssuance";
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23)
````solidity
	string constant public NAME = "LQTYStaking";
````


# [G-02] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a `struct`, where appropriate

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L41-L47](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L41-L47)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L56-L64](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L56-L64)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L167](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L167)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L186-L187](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L186-L187)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L228](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L228)      
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L85-L120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L85-L120)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25-L32](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25-L32)
## Vulnerability details
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.       
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L41-L47](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L41-L47)
````solidity
    mapping (address => uint256) internal collAmount;  // collateral => amount tracker
    mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker


    mapping (address => uint256) public yieldingPercentage; // collateral => % to use for yield farming (in BPS, <= 10k)
    mapping (address => uint256) public yieldingAmount; // collateral => actual wei amount being used for yield farming
    mapping (address => address) public yieldGenerator; // collateral => corresponding ERC4626 vault
    mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L56-L64](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L56-L64) 
````solidity
    // User data for LUSD token
    mapping (address => uint256) private _balances;
    mapping (address => mapping (address => uint256)) private _allowances;  
    
    // --- Addresses ---
    // mappings store addresses of old versions so they can still burn (close troves)
    mapping (address => bool) public troveManagers;
    mapping (address => bool) public stabilityPools;
    mapping (address => bool) public borrowerOperations;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L167](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L167)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L186-L187](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L186-L187)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L228](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L228)
````solidity
    mapping (address => uint256) internal collAmounts;  // deposited collateral tracker
    
    mapping (address => Deposit) public deposits;  // depositor address -> Deposit struct
    mapping (address => Snapshots) public depositSnapshots;  // depositor address -> snapshots struct
    
    mapping (address => uint) public lastCollateralError_Offset;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L85-L120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L85-L120)
````solidity
    // user => (collateral type => trove)
    mapping (address => mapping (address => Trove)) public Troves;


    mapping (address => uint) public totalStakes;


    // Snapshot of the value of totalStakes for each collateral, taken immediately after the latest liquidation
    mapping (address => uint) public totalStakesSnapshot;


    // Snapshot of the total collateral across the ActivePool and DefaultPool, immediately after the latest liquidation.
    mapping (address => uint) public totalCollateralSnapshot;


    /*
    * L_Collateral and L_LUSDDebt track the sums of accumulated liquidation rewards per unit staked. During its lifetime, each stake earns:
    *
    * A collateral gain of ( stake * [L_Collateral - L_Collateral(0)] )
    * A LUSDDebt increase  of ( stake * [L_LUSDDebt - L_LUSDDebt(0)] )
    *
    * Where L_Collateral(0) and L_LUSDDebt(0) are snapshots of L_Collateral and L_LUSDDebt for the active Trove taken at the instant the stake was made
    */
    mapping (address => uint) public L_Collateral;
    mapping (address => uint) public L_LUSDDebt;


    // Map addresses with active troves to their RewardSnapshot
    // user => (collateral type => reward snapshot))
    mapping (address => mapping (address => RewardSnapshot)) public rewardSnapshots;


    // Object containing the Collateral and LUSD snapshots for a given active trove
    struct RewardSnapshot { uint collAmount; uint LUSDDebt;}


    // Array of all active trove addresses - used to to compute an approximate hint off-chain, for the sorted list insertion
    // collateral type => array of trove owners
    mapping (address => address[]) public TroveOwners;


    // Error trackers for the trove redistribution calculation
    mapping (address => uint) public lastCollateralError_Redistribution;
    mapping (address => uint) public lastLUSDDebtError_Redistribution;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25-L32](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25-L32)
````solidity
    mapping( address => uint) public stakes;
    uint public totalLQTYStaked;


    mapping (address => uint) public F_Collateral;  // Running sum of collateral fees per-LQTY-staked
    uint public F_LUSD; // Running sum of LUSD fees per-LQTY-staked


    // User snapshots of F_Collateral and F_LUSD, taken at the point at which their latest deposit was made
    mapping (address => Snapshot) public snapshots; 
````


# [G-03] `require()` or `revert()` statements that check input arguments should be at the top of the function

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L93)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L107](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L107)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L127](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L127)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L97)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L197)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L81)
## Vulnerability details
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L93)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L107](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L107)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L127](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L127) 
````solidity
	require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
	require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");
	require(_bps <= 10_000, "Invalid BPS value");
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L97)
````solidity
	require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754)
````solidity
	require(totals.totalDebtInSequence > 0);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L197)
````solidity
	require(totalAllocBPS <= PERCENT_DIVISOR, "Invalid BPS value");
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L81)
````solidity
	require(_multisigRoles.length == 3, "Invalid number of multisig roles");
````

### Recommended mitigation steps
Consider moving on top of the `require` statements listed above.

# [G-04] Using `unchecked` blocks to save gas - Increments in `for` loop can be unchecked ( save 30-40 gas per loop iteration)

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108)      
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L397](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L397)      
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L859](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L859)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690)      
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240)
## Vulnerability details
### Impact
The majority of Solidity for loops increment a uint256 variable that starts at 0. These increment operations never need to be checked for over/underflow because the variable will never reach the max number of uint256 (will run out of gas long before that happens). The default over/underflow check wastes gas in every iteration of virtually every for loop . eg.     
e.g Let’s work with a sample loop below.
````solidity
	for(uint256 i; i < 10; i++){
	//doSomething
	}
````
can be written as shown below.
````solidity
	for(uint256 i; i < 10;) {
	  // loop logic
	  unchecked { i++; }
	}
````
We can also write it as an inlined function like below.
````solidity
	function inc(i) internal pure returns (uint256) {
	  unchecked { return i + 1; }
	}
	for(uint256 i; i < 10; i = inc(i)) {
	  // doSomething
	}
````
### Proof of Concept
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108)  
````solidity
108:	for(uint256 i = 0; i < numCollaterals; i++) {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56)  
````solidity
56:	for(uint256 i = 0; i < _collaterals.length; i++) {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L397](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L397)      
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L859](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L859)    
````solidity
351:	for (uint i = 0; i < numCollaterals; i++) {

397:	for (uint i = 0; i < numCollaterals; i++) {

640:	for (uint i = 0; i < assets.length; i++) {

810:	for (uint i = 0; i < collaterals.length; i++) {

831:	for (uint i = 0; i < collaterals.length; i++) {

859:	for (uint i = 0; i < numCollaterals; i++) {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690)      
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882)  
````solidity
608:	for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {

690:	for (vars.i = 0; vars.i < _n; vars.i++) {

808:	for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

882:	for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240)
````solidity
206:	for (uint i = 0; i < assets.length; i++) {

228:	for (uint i = 0; i < collaterals.length; i++) {

240:	for (uint i = 0; i < numCollaterals; i++) {
````


# [G-05] Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement

## Vulnerability details
`require(a <= b); x = b - a` => `require(a <= b); unchecked { x = b - a }`    
There are 23 instances of this issue:     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L266](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L266)
````solidity
266:	vars.toWithdraw = vars.currentAllocated.sub(vars.finalYieldingAmount);
267:	vars.yieldingAmount = vars.yieldingAmount.sub(vars.toWithdraw);

271:	vars.toDeposit = vars.finalYieldingAmount.sub(vars.currentAllocated);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L338](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L338)
````solidity
338:	_requireAtLeastMinNetDebt(_getNetDebt(vars.debt).sub(vars.netDebtChange));
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L465](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L465)
````solidity
465:	debtToRedistribute = _debt.sub(debtToOffset);
466:	collToRedistribute = _coll.sub(collToSendToSP);

888:	vars.remainingLUSDInStabPool = vars.remainingLUSDInStabPool.sub(singleLiquidation.debtToOffset);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L88](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L88)
````solidity
88:	uint256 timePassed = endTimestamp.sub(lastIssuanceTimestamp);

107:	uint timeLeft = lastDistributionTime.sub(lastIssuanceTimestamp);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L81)
````solidity
81:	uint256 toReinvest = wantBalance - _debt;

100:	loss = _amountNeeded - liquidatedAmount;

132:	uint256 profit = totalAssets - allocated;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194)
````solidity
194:	totalAllocBPS -= strategies[_strategy].allocBPS;

244:	uint256 available = stratMaxAllocation - stratCurrentAllocation;

330:	_amount = balAfter - balBefore;

384:	uint256 remaining = value - vaultBalance;

390:	value -= loss;

421:	return lockedProfit - ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);

514:	strategy.allocated -= vars.debtPayment;
515:	totalAllocated -= vars.debtPayment;
516:	vars.debt -= vars.debtPayment;

535:	lockedProfit = vars.lockedProfitBeforeLoss - vars.loss;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L128](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L128)
````solidity
128:	repayment -= uint256(-roi);
````


# [G-06] `internal` functions only called once can be inlined to save gas

## Vulnerability details
Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.    
There are 57 instances of this issue:
````solidity
Ethos-Core/contracts/ActivePool.sol

320:	function _requireCallerIsBorrowerOperationsOrDefaultPool() internal view {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L320](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L320)
````solidity
Ethos-Core/contracts/BorrowerOperations.sol

438:	function _getCollChange(

455:	function _updateTroveFromAdjustment

476:	function _moveTokensAndCollateralfromAdjustment

533:	function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {

537:	function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {

546:	function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {

551:	function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {

555:	function _requireNotInRecoveryMode(

567:	function _requireNoCollWithdrawal(uint _collWithdrawal) internal pure {

571:	function _requireValidAdjustmentInCurrentMode

624:	function _requireNewICRisAboveOldICR(uint _newICR, uint _oldICR) internal pure {

636:	function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure { 

640:	function _requireCallerIsStabilityPool() internal view {

661:	function _getNewNominalICRFromTroveChange

682:	function _getNewICRFromTroveChange
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L438](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L438)
````solidity
Ethos-Core/contracts/LUSDToken.sol

320:	function _mint(address account, uint256 amount) internal {

328:	function _burn(address account, uint256 amount) internal {

360:	function _requireCallerIsBorrowerOperations() internal view {

365:	function _requireCallerIsBOorTroveMorSP() internal view {

375:	function _requireCallerIsStabilityPool() internal view {

380:	function _requireCallerIsTroveMorSP() internal view {

393:	function _requireMintingNotPaused() internal view {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L320](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L320)
````solidity
Ethos-Core/contracts/StabilityPool.sol

421:	function _updateG(uint _LQTYIssuance) internal {

439:	function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {

597:	function _moveOffsetCollAndDebt(address _collateral, uint _collToAdd, uint _debtToOffset) internal {

637:	function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {

693:	function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {

729:	function _getCompoundedDepositFromSnapshots(

776:	function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {

794:	function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {

848:	function _requireCallerIsActivePool() internal view {

852:	function _requireCallerIsTroveManager() internal view {

856:	function _requireNoUnderCollateralizedTroves() internal {

869:	function _requireUserHasDeposit(uint _initialDeposit) internal pure {

873:	function _requireNonZeroAmount(uint _amount) internal pure {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L421](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L421)
````solidity
Ethos-Core/contracts/TroveManager.sol

478:	function _getCappedOffsetVals

582:	function _getTotalsFromLiquidateTrovesSequence_RecoveryMode

671:	function _getTotalsFromLiquidateTrovesSequence_NormalMode

785:	function _getTotalFromBatchLiquidate_RecoveryMode

864:	function _getTotalsFromBatchLiquidate_NormalMode

1082:	function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {

1213:	function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {

1321:	function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {

1339:	function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {

1538:	function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L478](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L478)
````solidity
Ethos-Core/contracts/LQTY/LQTYStaking.sol

251:	function _requireCallerIsTroveManagerOrActivePool() internal view {

261:	function _requireCallerIsBorrowerOperations() internal view {

265:	function _requireUserHasStake(uint currentStake) internal pure {

269:	function _requireNonZeroAmount(uint _amount) internal pure {
````
[[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L128](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L128)](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L251)
````solidity
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

86:	function _liquidatePosition(uint256 _amountNeeded)

176:	function _deposit(uint256 toReinvest) internal {

206:	function _claimRewards() internal {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L86](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L86)
````solidity
Ethos-Vault/contracts/ReaperVaultV2.sol

462:	function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L462](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L462)
````solidity
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

228:	function _adjustPosition(uint256 _debt) internal virtual;

240:	function _liquidatePosition(uint256 _amountNeeded)

250:	function _liquidateAllPositions() internal virtual returns (uint256 amountFreed);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L228](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L228)


# [G-07] Structs can be packed into fewer storage slots

## Vulnerability details
Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.      

Subsequent reads as well as writes have smaller gas savings.     

There are 8 instances of this issue:
````solidity
Ethos-Core/contracts/BorrowerOperations.sol

/// @audit Variable ordering with 15 slots instead of the current 16:
///	uint256(32):collCCR, uint256(32):collMCR, uint256(32):collMCR, uint(32):price, uint(32):collChange, uint(32):netDebtChange, uint(32):debt, uint(32):coll, uint(32):oldICR, uint(32):newICR, uint(32):newTCR, uint(32):LUSDFee, uint(32):newDebt, uint(32):newColl, uint(32):stake, bool(1):isCollIncrease 
47     struct LocalVariables_adjustTrove {
48        uint256 collCCR;
49        uint256 collMCR;
50        uint256 collDecimals;
51        uint price;
52        uint collChange;
53        uint netDebtChange;
54        bool isCollIncrease;
55        uint debt;
56        uint coll;
57        uint oldICR;
58        uint newICR;
59        uint newTCR;
60        uint LUSDFee;
61        uint newDebt;
62        uint newColl;
63        uint stake;
64:    }
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L47-L64](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L47-L64)
````solidity
Ethos-Core/contracts/CollateralConfig.sol

/// @audit Variable ordering with 3 slots instead of the current 4:
///	uint256(32):decimals, uint256(32):MCR, uint256(32):CCR, bool(1):allowed
27    struct Config {
28        bool allowed;
29        uint256 decimals;
30        uint256 MCR;
31        uint256 CCR;
32:    }
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L27-L32](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L27-L32)
````solidity
Ethos-Core/contracts/StabilityPool.sol

/// @audit Variable ordering with 4 slots instead of the current 5:
///	mapping(address => uint):S, uint128(16):scale, uint128(16):epoch, uint(256):P, uint(256):G
178    struct Snapshots {
179        mapping (address => uint) S;
180        uint P;
181        uint G;
182        uint128 scale;
183        uint128 epoch;
184:    }

/// @audit Variable ordering with 7 slots instead of the current 8:
///	uint128(16):epochSnapshot, uint128(16):scaleSnapshot, uint(32):P_Snapshot, uint(32):S_Snapshot, uint(32):firstPortion, uint(32):secondPortion, uint(32):gain, uint256(32):collDecimals 
646	struct LocalVariables_getSingularCollateralGain {
647		uint256 collDecimals;
648		uint128 epochSnapshot;
649		uint128 scaleSnapshot;
650		uint P_Snapshot;
651		uint S_Snapshot;
652		uint firstPortion;
653		uint secondPortion;
654		uint gain;
655:	    }
````
[[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L88](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L88)](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L178-L184)
````solidity
Ethos-Core/contracts/TroveManager.sol

/// @audit Variable ordering with 4 slots instead of the current 5:
///	user-defined(32):status, uint128(16):arrayIndex, uint(32):debt, uint(32):coll, uint(32):stake
77    struct Trove {
78        uint debt;
79        uint coll;
80        uint stake;
81        Status status;
82        uint128 arrayIndex;
83:    }

/// @audit Variable ordering with 7 slots instead of the current 8:
///	uint(32):price, uint(32):LUSDInStabPool, uint(32):liquidatedDebt, uint(32):liquidatedColl, uint256(32):collDecimals, uint256(32):collCCR, uint256(32):collMCR, bool(1):recoveryModeAtStart 
129	struct LocalVariables_OuterLiquidationFunction {
130		uint256 collDecimals;
131		uint256 collCCR;
132		uint256 collMCR;
133		uint price;
134		uint LUSDInStabPool;
135		bool recoveryModeAtStart;
136		uint liquidatedDebt;
137		uint liquidatedColl;
138:	    }

/// @audit Variable ordering with 10 slots instead of the current 11:
///	uint(32):remainingLUSDInStabPool, uint(32):i, uint(32):ICR, uint(32):TCR, uint(32):entireSystemDebt, uint(32):entireSystemColl, uint256(32):collDecimals, uint256(32):collCCR, uint256(32):collMCR, address(20):user, bool(1):backToNormalMode
146    struct LocalVariables_LiquidationSequence {
147        uint remainingLUSDInStabPool;
148        uint i;
149        uint ICR;
150        uint TCR;
151        address user;
152        bool backToNormalMode;
153        uint entireSystemDebt;
154        uint entireSystemColl;
155        uint256 collDecimals;
156        uint256 collCCR;
157        uint256 collMCR;
158:    }
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L77-L83](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L77-L83)
````solidity
Ethos-Vault/contracts/ReaperVaultV2.sol

/// @audit Variable ordering with 9 slots instead of the current 10:
///	uint256(32):loss, uint256(32):gain, uint256(32):fees, int256(32):available, uint256(32):debt, uint256(32):credit, uint256(32):debtPayment, uint256(32):freeWantInStrat, uint256(32):lockedProfitBeforeLoss, address(20):stratAddr
473    struct LocalVariables_report {
474        address stratAddr;
475        uint256 loss;
476        uint256 gain;
477        uint256 fees;
478        int256 available;
479        uint256 debt;
480        uint256 credit;
481        uint256 debtPayment;
482        uint256 freeWantInStrat;
483        uint256 lockedProfitBeforeLoss;
484:    }
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L473-L484](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L473-L484)



# [G-08] Using `storage` instead of `memory` for structs/arrays saves gas

## Vulnerability details
When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.     
If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.    

There are 4 instances of this issue:
````solidity
Ethos-Core/contracts/BorrowerOperations.sol

175:	ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);

283:	ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L175](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L175)
````solidity
Ethos-Core/contracts/StabilityPool.sol

687:	Snapshots memory snapshots = depositSnapshots[_depositor];

722:	Snapshots memory snapshots = depositSnapshots[_depositor];
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L687](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L687)



# [G-09] Splitting `require()` statements that use `&&` saves gas

## Vulnerability details
See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by 3 gas.  

There are 4 instances of this issue:
````solidity
Ethos-Core/contracts/BorrowerOperations.sol

653            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
654:                "Max fee percentage must be between 0.5% and 100%");
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653-L654](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653-L654)
````solidity
Ethos-Core/contracts/LUSDToken.sol

347        require(
348            _recipient != address(0) && 
349            _recipient != address(this),
350            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
351:        );
352        require(
353            !stabilityPools[_recipient] &&
354            !troveManagers[_recipient] &&
355            !borrowerOperations[_recipient],
356            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
357:        );
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347-L357](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347-L357)

````solidity
Ethos-Core/contracts/TroveManager.sol

1539:        require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539)



# [G-10] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

## Vulnerability details
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)      
Each operation involving a `uint8` costs an extra [22-28 gas](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. The same goes for `uint16`/`uint128`.     
Use a larger size then downcast where needed.

There are 15 instances of this issue:
````solidity
Ethos-Core/contracts/LUSDToken.sol

35:	uint8 constant internal _DECIMALS = 18;

````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L35](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L35)
````solidity
Ethos-Core/contracts/StabilityPool.sol

200:	uint128 public currentScale;

203:	uint128 public currentEpoch;

558	uint128 currentScaleCached = currentScale;
559:	uint128 currentEpochCached = currentEpoch;

699	uint128 epochSnapshot = snapshots.epoch;
700:	uint128 scaleSnapshot = snapshots.scale;

738	uint128 scaleSnapshot = snapshots.scale;
739:	uint128 epochSnapshot = snapshots.epoch;

745:	uint128 scaleDiff = currentScale.sub(scaleSnapshot);

817	uint128 currentScaleCached = currentScale;
818:	uint128 currentEpochCached = currentEpoch;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L182-L183](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L182-L183)

````solidity
Ethos-Core/contracts/TroveManager.sol

1329:	index = uint128(TroveOwners[_collateral].length.sub(1));

1344:	uint128 index = Troves[_borrower][_collateral].arrayIndex;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1329](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1329)
````solidity
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

35:	uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35)


# [G-11] `public` functions not called by the contract should be declared `external` instead

## Vulnerability details
Contracts are allowed to override their parents’ functions and change the visibility from `external` to `public` and can save gas by doing so.     

`getNominalICR`
- [TroveManager.sol#L1045](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1045)

`getPricePerFullShare`
- [ReaperVaultV2.sol#L295](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L295)



# [G-12] Using Solidity version `0.8.17` will provide an overall gas optimization

## Vulnerability details
Using at least `0.8.10` will save gas due to skipped `extcodesize` check if there is a return value. Currently the contracts are compiled using version `0.6.11`. It is easily changeable to `0.8.17` using the command `sed -i 's/0\.8\.7/^0.8.0/' test/*.sol && sed -i '4isolc = "0.8.17"' foundry.toml`.    



# [G-13] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

## Vulnerability details
Using the addition operator instead of plus-equals saves gas.     

There are 22 instances of this issue:
````solidity
Ethos-Vault/contracts/ReaperVaultV2.sol

168:	totalAllocBPS += _allocBPS;

194:	totalAllocBPS -= strategies[_strategy].allocBPS;

196:	totalAllocBPS += _allocBPS;

214:	totalAllocBPS -= strategies[_strategy].allocBPS;

390	value -= loss;
391:	totalLoss += loss;

395:	strategies[stratAddr].allocated -= actualWithdrawn;
396:	totalAllocated -= actualWithdrawn;

444:	stratParams.allocBPS -= bpsChange;
445:	totalAllocBPS -= bpsChange;

450:	stratParams.losses += loss;
451:	stratParams.allocated -= loss; 
452:	totalAllocated -= loss;

505:	strategy.gains += vars.gain;

514	strategy.allocated -= vars.debtPayment;
515	totalAllocated -= vars.debtPayment;
516:	vars.debt -= vars.debtPayment;

520	strategy.allocated += vars.credit;
521:	totalAllocated += vars.credit;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168)
````solidity
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

133:	toFree += profit;

141:	roi -= int256(loss);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L133](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L133)
````solidity
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

128:	repayment -= uint256(-roi);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L128](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L128)



# [G-14] `keccak256()` should only need to be called on a specific string literal once

## Vulnerability details
It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.  

There are 3 instances of this issue:
````solidity
Ethos-Core/contracts/HintHelpers.sol

178:	latestRandomSeed = uint(keccak256(abi.encodePacked(latestRandomSeed)));
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L178](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L178)
````solidity
Ethos-Core/contracts/LUSDToken.sol

283        bytes32 digest = keccak256(abi.encodePacked('\x19\x01', 
284                         domainSeparator(), keccak256(abi.encode(
285                         _PERMIT_TYPEHASH, owner, spender, amount, 
286:                         _nonces[owner]++, deadline))));

305:        return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L283-L286](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L283-L286)



# [G-15] Remove the `initializer` modifier

## Vulnerability details
If we can just ensure that the `initialize()` function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

````solidity
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

62	    function initialize(
63		address _vault,
64		address[] memory _strategists,
65		address[] memory _multisigRoles,
66		IAToken _gWant
67:	    ) public initializer {
````
In the EVM, the constructor’s job is actually to return the bytecode that will live at the contract’s address. So, while inside a constructor, your address `(address(this))` will be the deployment address, but there will be no bytecode at that address!     
So if we check `address(this).code.length` before the constructor has finished, even from within a delegatecall, we will get 0. So now let’s update our `initialize()` function to only run if we are inside a constructor:
````solidity
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

62	    function initialize(
63		address _vault,
64		address[] memory _strategists,
65		address[] memory _multisigRoles,
66		IAToken _gWant
67:	    ) public initializer {
+		require(address(this).code.length == 0, 'not in constructor');
````
Now the Proxy contract’s constructor can still delegatecall initialize(), but if anyone attempts to call it again (after deployment) through the Proxy instance, or tries to call it directly on the above instance, it will revert because address(this).code.length will be nonzero.    

Also, because we no longer need to write to any state to track whether initialize() has been called, we can avoid the 20k storage gas cost. In fact, the cost for checking our own code size is only 2 gas, which means we have a 10,000x gas savings over the standard version.



# [G-16] Use constants instead of `type(uintx).max`

## Vulnerability details
`type(uint256)` uses more gas in the distribution process and also for each transaction than constant usage.

````solidity
Ethos-Vault/contracts/ReaperVaultERC4626.sol

81:	if (tvlCap == type(uint256).max) return type(uint256).max;

124:	if (tvlCap == type(uint256).max) return type(uint256).max;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L81)
````solidity
Ethos-Vault/contracts/ReaperVaultV2.sol

588:	updateTvlCap(type(uint256).max);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L588](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L588)
````solidity
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

74:	IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74)

# [G-17] Use nested `if` and, avoid multiple check combinations

## Vulnerability details
Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.         
     
10 results - 3 files:

````solidity
Ethos-Core/contracts/ActivePool.sol
264:	if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {

269:	} else if(vars.percentOfFinalBal < vars.yieldingPercentage && vars.yieldingPercentage.sub(vars.percentOfFinalBal) > yieldingPercentageDrift) {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L264](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L264)
````solidity
Ethos-Core/contracts/BorrowerOperations.sol
311:	if (_isDebtIncrease && !isRecoveryMode) {

337:	if (!_isDebtIncrease && _LUSDChange > 0) {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L311](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L311)
````solidity
Ethos-Core/contracts/TroveManager.sol
397:	} else if ((_ICR > _100pct) && (_ICR < _MCR)) {

415:	} else if ((_ICR >= _MCR) && (_ICR < _TCR) && (singleLiquidation.entireTroveDebt <= _LUSDInStabPool)) {

616:	if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { break; }

651:	else if (vars.backToNormalMode && vars.ICR < vars.collMCR) {

817:	if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { continue; }

853:	else if (vars.backToNormalMode && vars.ICR < vars.collMCR) {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L397](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L397)
### Recommended mitigation steps
````solidity
Ethos-Core/contracts/ActivePool.sol
- 264:	if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {
+ 264:	if (vars.percentOfFinalBal > vars.yieldingPercentage) {
+		if (vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {
+		}
+	}
````

# [G-18] Setting the constructor to `payable`

## Vulnerability details
You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of `msg.value == 0` and saves `13 gas` on deployment with no security risks.        
     
3 results - 3 files:

````solidity
Ethos-Core/contracts/LUSDToken.sol
84	    constructor
85	    ( 
86		address _troveManagerAddress,
87		address _stabilityPoolAddress,
88		address _borrowerOperationsAddress,
89		address _governanceAddress,
90		address _guardianAddress
91	    ) 
92		public 
93:	    {  
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L84-L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L84-L93)
````solidity
Ethos-Core/contracts/TroveManager.sol
225:    	constructor() public {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L225](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L225)
````solidity
Ethos-Vault/contracts/ReaperVaultV2.sol
111	    constructor(
112		address _token,
113		string memory _name,
114		string memory _symbol,
115		uint256 _tvlCap,
116		address _treasury,
117		address[] memory _strategists,
118		address[] memory _multisigRoles
119:	    ) ERC20(string(_name), string(_symbol)) {
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L119](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L119)
### Recommended mitigation steps
Set the constructor to `payable`.

# [G-19] Use `assembly` to write address storage values

## Vulnerability details
54 results - 9 files:

````solidity
Ethos-Core/contracts/ActivePool.sol
71	function setAddresses(
96:		collateralConfigAddress = _collateralConfigAddress;
97:		borrowerOperationsAddress = _borrowerOperationsAddress;
98:		troveManagerAddress = _troveManagerAddress;
99:		stabilityPoolAddress = _stabilityPoolAddress;
100:		defaultPoolAddress = _defaultPoolAddress;
101:		collSurplusPoolAddress = _collSurplusPoolAddress;
102:		treasuryAddress = _treasuryAddress;
103:		lqtyStakingAddress = _lqtyStakingAddress;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L96-L103](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L96-L103)
````solidity
Ethos-Core/contracts/HintHelpers.sol
27	function setAddresses(
39:		collateralConfig = ICollateralConfig(_collateralConfigAddress);
40:		sortedTroves = ISortedTroves(_sortedTrovesAddress);
41:		troveManager = ITroveManager(_troveManagerAddress);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L39-L41](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L39-L41)
````solidity
Ethos-Core/contracts/LUSDToken.sol
84	constructor
102:		troveManagerAddress = _troveManagerAddress;
106:		stabilityPoolAddress = _stabilityPoolAddress;        
110:		borrowerOperationsAddress = _borrowerOperationsAddress;       
114:		governanceAddress = _governanceAddress;        
117:		guardianAddress = _guardianAddress;

146	function updateGovernance(address _newGovernanceAddress) external {
149:		governanceAddress = _newGovernanceAddress;

153	function updateGuardian(address _newGuardianAddress) external {
156:		guardianAddress = _newGuardianAddress;

160	function upgradeProtocol(
170:		troveManagerAddress = _newTroveManagerAddress;		
174:		stabilityPoolAddress = _newStabilityPoolAddress;		
178:		borrowerOperationsAddress = _newBorrowerOperationsAddress;
````
````solidity
Ethos-Core/contracts/StabilityPool.sol
261	function setAddresses(
284:		borrowerOperations = IBorrowerOperations(_borrowerOperationsAddress);
285:		collateralConfig = ICollateralConfig(_collateralConfigAddress);
286:		troveManager = ITroveManager(_troveManagerAddress);
287:		activePool = IActivePool(_activePoolAddress);
288:		lusdToken = ILUSDToken(_lusdTokenAddress);
289:		sortedTroves = ISortedTroves(_sortedTrovesAddress);
290:		priceFeed = IPriceFeed(_priceFeedAddress);
291:		communityIssuance = ICommunityIssuance(_communityIssuanceAddress);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L284-L291](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L284-L291)
````solidity
Ethos-Core/contracts/TroveManager.sol
232		function setAddresses(
266:			borrowerOperationsAddress = _borrowerOperationsAddress;
267:			collateralConfig = ICollateralConfig(_collateralConfigAddress);
268:			activePool = IActivePool(_activePoolAddress);
269:			defaultPool = IDefaultPool(_defaultPoolAddress);
270:			stabilityPool = IStabilityPool(_stabilityPoolAddress);
271:			gasPoolAddress = _gasPoolAddress;
272:			collSurplusPool = ICollSurplusPool(_collSurplusPoolAddress);
273:			priceFeed = IPriceFeed(_priceFeedAddress);
274:			lusdToken = ILUSDToken(_lusdTokenAddress);
275:			sortedTroves = ISortedTroves(_sortedTrovesAddress);
276:			lqtyToken = IERC20(_lqtyTokenAddress);
277:			lqtyStaking = ILQTYStaking(_lqtyStakingAddress);
278:			redemptionHelper = IRedemptionHelper(_redemptionHelperAddress);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L266-L278](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L266-L278)
````solidity
Ethos-Core/contracts/LQTY/CommunityIssuance.sol
261	function setAddresses
74:		OathToken = IERC20(_oathTokenAddress);
75:		stabilityPoolAddress = _stabilityPoolAddress;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L74-L75](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L74-L75)
````solidity
Ethos-Core/contracts/LQTY/LQTYStaking.sol
66	function setAddresses
86:		lqtyToken = IERC20(_lqtyTokenAddress);
87:		lusdToken = ILUSDToken(_lusdTokenAddress);
88:		troveManagerAddress = _troveManagerAddress;
89:		borrowerOperationsAddress = _borrowerOperationsAddress;
90:		activePoolAddress = _activePoolAddress;
91:		collateralConfig = ICollateralConfig(_collateralConfigAddress);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L86-L91](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L86-L91)
````solidity
Ethos-Vault/contracts/ReaperVaultV2.sol
111	constructor(
120:		token = IERC20Metadata(_token);    
124:		treasury = _treasury;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L120)
````solidity
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
63	function __ReaperBaseStrategy_init(
72:		vault = _vault;
73:		want = _want;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L72-L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L72-L73)
### Recommended mitigation steps
````solidity
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
	function __ReaperBaseStrategy_init(
		assembly{
			sstore(vault.store = _vault)
			sstore(want.store = _store)
		}
	}
````

# [G-20] Use a more recent version of solidity

## Vulnerability details
In `0.8.15` the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.     
     
In `0.8.17` prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.        
````solidity
Ethos-Core/contracts/CollateralConfig.sol
3:	pragma solidity 0.6.11;

Ethos-Core/contracts/BorrowerOperations.sol
3:	pragma solidity 0.6.11;

Ethos-Core/contracts/TroveManager.sol
3:	pragma solidity 0.6.11;

Ethos-Core/contracts/ActivePool.sol
3:	pragma solidity 0.6.11;

Ethos-Core/contracts/StabilityPool.sol
3:	pragma solidity 0.6.11;

Ethos-Core/contracts/LQTY/CommunityIssuance.sol
3:	pragma solidity 0.6.11;

Ethos-Core/contracts/LQTY/LQTYStaking.sol
3:	pragma solidity 0.6.11;

Ethos-Core/contracts/LUSDToken.sol
3:	pragma solidity 0.6.11;

Ethos-Vault/contracts/ReaperVaultV2.sol
3:	pragma solidity ^0.8.0;

Ethos-Vault/contracts/ReaperVaultERC4626.sol
3:	pragma solidity ^0.8.0;

Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol
3:	pragma solidity ^0.8.0;

Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
3:	pragma solidity ^0.8.0;

````

# [G-21] Upgrade Solidity’s `optimizer`

## Vulnerability details
Make sure Solidity’s `optimizer` is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity `optimizer` at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the `optimizer` to a high number.     
     
Set the optimization value higher than `800` in your `hardhat.config.js` file.
````solidity
38	    solidity: {
39		compilers: [
40		    {
41			version: "0.4.23",
42			settings: {
43			    optimizer: {
44				enabled: true,
45				runs: 100
46			    }
47			}
48		    },
49		    {
50			version: "0.5.17",
51			settings: {
52			    optimizer: {
53				enabled: true,
54				runs: 100
55			    }
56			}
57		    },
58		    {
59			version: "0.6.11",
60			settings: {
61			    optimizer: {
62				enabled: true,
63				runs: 100
64			    }
65			}
66		    },
67		]
68	    },
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/hardhat.config.js#L38-L68](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/hardhat.config.js#L38-L68)
````solidity
11	  solidity: {
12	    compilers: [
13	      {
14		version: "0.8.11",
15		settings: {
16		  optimizer: {
17		    enabled: true,
18		    runs: 200,
19		  },
20		},
21	      },
22	    ],
23:	  },
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/hardhat.config.js#L11-L23](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/hardhat.config.js#L11-L23)
