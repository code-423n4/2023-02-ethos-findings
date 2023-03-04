### [G] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

    168: totalAllocBPS += _allocBPS;
    214: totalAllocBPS -= strategies[_strategy].allocBPS;
    396: totalAllocated -= actualWithdrawn;
    445: totalAllocBPS -= bpsChange;
    452: totalAllocated -= loss;

### [G] CHANGES IN RETURNS COULD SAVE SOME GAS

if you remove the return statement at the end of the function and instead return the variable directly from the function definition, it will save some gas. This is because you would be avoiding the unnecessary assignment operation and just directly returning the value.

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

    462: function _chargeFees(address strategy, uint256 gain) internal returns (uint256 + performanceFee ) {
        uint256 performanceFee = (gain * strategies[strategy].feeBPS) / PERCENT_DIVISOR;
        if (performanceFee != 0) {
            uint256 supply = totalSupply();
            uint256 shares = supply == 0 ? performanceFee : (performanceFee * supply) / _freeFunds();
            _mint(treasury, shares);
        }
    - return performanceFee; 
    }

Similar cases:

    659: function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {

**\Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol**

    230: function balanceOfPool() public view returns (uint256) {

**\Ethos-Core\contracts\TroveManager.sol**

    353: return singleLiquidation; //can be omitted
    436: return singleLiquidation; //can be omitted
    913: return newTotals; //can be omitted
    1046: function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {
    1059: ) public view override returns (uint) {
    1067: function _getCurrentTroveAmounts(address _borrower, address _collateral) internal view returns (uint, uint) {
    1124: function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (unit)
    1139: function getPendingLUSDDebtReward(address _borrower, address _collateral) public view override returns (uint) {
    1202: function _updateStakeAndTotalStakes(address _borrower, address _collateral) internal returns (uint) {
    1214: function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {
    1333: return index; //can be omitted
    1403: ) external override returns (uint) {
    1423: return newBaseRate; //can be omitted
    1449: function _calcRedemptionFee(uint _redemptionRate, uint _collateralDrawn) internal pure returns (uint) {
    1568: function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint) {
    1575: function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint) {

    1583: function increaseTroveDebt(address _borrower, address _collateral, uint _debtIncrease) external override returns (uint) {
    1590: function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint) {

**\Ethos-Core\contracts\StabilityPool.sol**

    439: function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {
    543: return (collGainPerUnitStaked, LUSDLossPerUnitStaked); //can be omitted
    683: function getDepositorLQTYGain(address _depositor) public view override returns (uint) {
    693: function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {
    718: function getCompoundedLUSDDeposit(address _depositor) public view override returns (uint) {
    735: returns (unit)

**\Ethos-Core\contracts\LQTY\LQTYStaking.sol**

    217: function _getPendingLUSDGain(address _user) internal view returns (uint) {

**\Ethos-Core\contracts\BorrowerOperations.sol**

    432: function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {
    466: returns (uint, unit)
    673: returns (unit)
    695: returns (unit)
    713: returns (uint, unit)
    736: returns (uint) 



### [G] Setting a variable to zero is cheaper than delete

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

     263: delete withdrawalQueue;

**\Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol**

    162: delete steps;

for arrays, setting the length to zero (withdrawalQueue.length = 0) is cheaper than using delete withdrawalQueue because the delete keyword also clears the storage slot for the array, which is an additional gas cost.

### [G] REQUIRE() OR REVERT() STATEMENTS THAT CHECK INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION
By doing these checks first, the function is able to revert before gas in a function that may ultimately revert in the unhappy case.

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

     627: function updateTreasury(address newTreasury) external {
        _atLeastRole(DEFAULT_ADMIN_ROLE);
        require(newTreasury != address(0), "Invalid address"); //move require up to the first line of the function

     637: function inCaseTokensGetStuck(address _token) external {
        _atLeastRole(ADMIN); 
        require(_token != address(token), "!token"); //move require up to the first line of the function

**\Ethos-Core\contracts\ActivePool.sol**

     93: require(_treasuryAddress != address(0), "Treasury cannot be 0 address"); //move require up to the first line of the function
    127: require(_bps <= 10_000, "Invalid BPS value"); "); //move require up to the first line of the function

**\Ethos-Core\contracts\CollateralConfig.sol**

    66: require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
    69: require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");

**\Ethos-Core\contracts\CollateralConfig.sol**

    94: require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
    97: require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");


### [G] COMBINE 2 REQUIRE STATEMENTS WITH THE SAME ERROR TO SAVE GAS

When we use two separate require statements, the EVM will execute two JUMPI and REVERT instructions if either of the conditions is not met. However, when you use require(condition1 || condition2, "error"), the EVM will only execute one JUMPI instruction, and then another JUMPI instruction to skip the next REVERT instruction if either of the conditions is met. This can save some gas compared to using two separate require statements.

**\Ethos-Core\contracts\CollateralConfig.sol**

    53: -  require(_MCRs.length == _collaterals.length, "Array lengths must match"); 
       -  require(_CCRs.length == _collaterals.length, "Array lenghts must match");
    + require(_MCRs.length == _collaterals.length || _CCRs.length == _collaterals.length , "Array lengths must match");

**\Ethos-Core\contracts\BorrowerOperations.sol**

There are different error messages in require statements in the following code, but in theory, they could be combined the same way as in the previous example. 

    529: require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance"); 
        require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");

### [G] USING FIXED BYTES IS CHEAPER THAN USING STRING

If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

**\Ethos-Core\contracts\ActivePool.sol**

    30: string constant public NAME = "ActivePool"

**\Ethos-Core\contracts\BorrowerOperations.sol**

    21: string constant public NAME = "BorrowerOperations";

**\Ethos-Core\contracts\LUSDToken.sol**

    32: string constant internal _NAME = "LUSD Stablecoin"; 
    string constant internal _SYMBOL = "LUSD";
    string constant internal _VERSION = "1";

**\Ethos-Core\contracts\StabilityPool.sol**

    150: string constant public NAME = "StabilityPool";

**\Ethos-Core\contracts\LQTY\CommunityIssuance.sol**

    19: string constant public NAME = "CommunityIssuance";

### [G] SIMPLE IF STATEMENT CAN BE WRITTEN AS A TERNARY OPERATOR TO SAVE SOME GAS

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

    338: if (totalSupply() == 0) { 
            shares = _amount;
        } else {
            
            shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"
        }
Can be swapped with:
     `shares = (totalSupply() == 0) ? _amount : (_amount * totalSupply()) / freeFunds;`


### [G] USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

**\Ethos-Core\contracts\ActivePool.sol**

    `240: LocalVariables_rebalance memory vars;`

**\Ethos-Core\contracts\BorrowerOperations.sol**

    `ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);`
    `LocalVariables_openTrove memory vars;`

**\Ethos-Core\contracts\TroveManager.sol**

    `518: LocalVariables_OuterLiquidationFunction memory vars;`
        `LiquidationTotals memory totals;`

    684: LocalVariables_LiquidationSequence memory vars;
        LiquidationValues memory singleLiquidation;
    722: LocalVariables_OuterLiquidationFunction memory vars;
        LiquidationTotals memory totals;
    797: LocalVariables_LiquidationSequence memory vars;
    801: LiquidationValues memory singleLiquidation;



**\Ethos-Vault\contracts\ReaperVaultV2.sol**

    494: LocalVariables_report memory vars;

### [G] ASSIGNING A VALUE MSG.SENDER TO A VARIABLE IN SOLIDITY REQUIRES AN ADDITIONAL OPERATION COMPARED TO SIMPLY ACCESSING THE MSG.SENDER DIRECTLY.

Also if you use msg.sender several times within a function, only one CALLER opcode will be executed to set the value of msg.sender in memory, and subsequent references to msg.sender will simply read from that memory location, without incurring additional gas costs for a CALLER opcode.

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

Before: 5128 gas

    225: function availableCapital() public view returns (int256) {
        address stratAddr = msg.sender;
        if (totalAllocBPS == 0 || emergencyShutdown) {
            return -int256(strategies[stratAddr].allocated);
        }

        uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;
        uint256 stratCurrentAllocation = strategies[stratAddr].allocated;

After: 5114 gas

    function availableCapital() public view returns (int256) {
        
        if (totalAllocBPS == 0 || emergencyShutdown) {
            return -int256(strategies[msg.sender].allocated);
        }

        uint256 stratMaxAllocation = (strategies[msg.sender].allocBPS * balance()) / PERCENT_DIVISOR;
        uint256 stratCurrentAllocation = strategies[msg.sender].allocated;

### [G] OPTIMIZE FUNCTIONS’ NAMES TO SAVE SOME GAS

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas is added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

For example, during tests, I noticed that the function availableCapital() is the most often called function in the Vault contract. So, we can modify its name to save some gas.

The current function’s Signature is `199cb7d8` and as we know a function with 00 at the beginning of the signature will be the cheapest one to call.

So `availableCapital()` can be renamed to `availableCapital_3()`, then its signature will be 0094169d.

Similar optimization could be done for the most often callable functions from other contracts.

### [G] UPGRADE SOLIDITY’S OPTIMIZER

Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.

Set the optimization value higher than 800 in your hardhat.config.js file.

### [G] USE CONSTANTS INSTEAD OF TYPE(UINTX).MAX

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

    624: updateTvlCap(type(uint256).max);

**\Ethos-Vault\contracts\ReaperVaultERC4626.sol**

    81: if (tvlCap == type(uint256).max) return type(uint256).max;
    124: if (tvlCap == type(uint256).max) return type(uint256).max;

**\Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol**

    79: IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);

### [G] USE ASSEMBLY TO WRITE ADDRESS STORAGE VALUES

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

    111: constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) ERC20(string(_name), string(_symbol)) { 
        
        token = IERC20Metadata(_token);
        constructionTime = block.timestamp;
        lastReport = block.timestamp;
        tvlCap = _tvlCap;
	- //treasury = _treasury;
        +  assembly {
            sstore(treasury.slot, _treasury)
        }
    663: function updateTreasury(address newTreasury) external {
        _atLeastRole(DEFAULT_ADMIN_ROLE); 
        require(newTreasury != address(0), "Invalid address");
	- treasury = newTreasury;
	+  assembly {
	            sstore(treasury.slot, newTreasury)
	        }

    }

**\Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol**

    63: function __ReaperBaseStrategy_init(
        address _vault,
        address _want,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) internal onlyInitializing {
        __UUPSUpgradeable_init();
        __AccessControlEnumerable_init();

       -// vault = _vault;
       -// want = _want;
       + assembly {
            sstore(vault.slot, _vault)
            sstore(want.slot, _want)
        }

        IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);

        uint256 numStrategists = _strategists.length;
        for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
            _grantRole(STRATEGIST, _strategists[i]);
        }

        require(_multisigRoles.length == 3, "Invalid number of multisig roles");
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(DEFAULT_ADMIN_ROLE, _multisigRoles[0]);
        _grantRole(ADMIN, _multisigRoles[1]);
        _grantRole(GUARDIAN, _multisigRoles[2]);

        clearUpgradeCooldown();
    }

There are several similar cases in Ethos-Core, but with the current solidity version 0.6.11; it doesn’t work there.

### [G] SETTING THE CONSTRUCTOR TO PAYABLE

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

**E:\audits\2023-02-ethos\Ethos-Vault\contracts\ReaperVaultERC4626.sol**

    16: constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) + payable ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}

**\Ethos-Core\contracts\TroveManager.sol**

    225: constructor() public {
        // makeshift ownable implementation to circumvent contract size limit
        owner = msg.sender;
    }

**\Ethos-Core\contracts\LQTY\CommunityIssuance.sol**

    57: constructor() public {
        distributionPeriod = 14 days;
    }
