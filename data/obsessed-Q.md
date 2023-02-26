# Report

## QA Findings


| |Issue|Instances|
|-|:-|:-:|
| [L-1] | Usage of outdated solidity compiler | 9 |
| [L-2] | Not following the NATSPEC convention | Many instances |
| [L-3] | Function names are not readable | 2 |
| [L-4] | Function _burn should also clear approval for underlying token | 1 |
| [NC-1] | Unjustified usage of the '_' symbol in numbers | 3 |
| [NC-2] | Code layout is out of order | 1 |
| [NC-3] | Function and modifier have the same functionality | 2 |
| [NC-4] | Interchangeable usage of uint and uint256 | Many instances |
| [NC-5] | Incorrect use of uppercase letters for non constant variable | 1 |
| [NC-6] | Local and state variables should use mixedCase letters | 2 |
| [NC-7] | Defined events are not used | 2 |
| [NC-8] | Usage of keyword 'super' will improve readability | 1 |
| [NC-9] | One letter and uppercased variables | 1 |

### [L-1] Usage of outdated solidity compiler
Usage of outdated solidity compiler is a security risk because of well known old vulnerabilites, should be avoided.

*Instances (9)*:
```solidity
File: ActivePool.sol
File: BorrowerOperations.sol
File: CollateralConfig.sol
File: CommunityIssuance.sol
File: LQTYStaking.sol
File: LUSDToken.sol
File: StabilityPool.sol
File: TroveManager.sol
```

### [L-2] Not following the NATSPEC convention
Not following the NATSPEC convention leads to poor documentation and very low readability. In many places there are no comments whatsoever.

### [L-3] Function names are not readable 
Function names that are not readable lowers clarity of the code.

*Instances (2)*:
```solidity
File: ActivePool.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L327-L342

    function _requireCallerIsBOorTroveMorSP() internal view {
    function _requireCallerIsBOorTroveM() internal view {

File: LUSDToken.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L365-L373

    function _requireCallerIsBOorTroveMorSP() internal view {
```

### [L-4] Function burn should also clear approval for underlying token
Function with burn capabilites should also clear approval/allowance for underlying token to avoid unwanted behaviour.

*Instances (1)*:
```solidity
File: LUSDToken.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L191-194

    function burn(address _account, uint256 _amount) external override {
        _requireCallerIsBOorTroveMorSP();
        _burn(_account, _amount);
    }
```

### [NC-1] Unjustified usage of the '_' symbol in numbers
Unjustified usage of the '_' symbol is more confusing than helpful. Those values can easily be typed as: 2000, 4000, etc.

*Instances (3)*:
```solidity
File: ActivePool.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L52-L54

     uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
     uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
     uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS
```

### [NC-2] Code layout is out of order
Code layout is out of order, struct shouldn't be in the middle of the file between different functions.

*Instances (1)*:
```solidity
File: ActivePool.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L220-L237

    struct LocalVariables_rebalance {
        uint256 currentAllocated;
        IERC4626 yieldGenerator;
        uint256 ownedShares;
        uint256 sharesToAssets;
        uint256 profit;
        uint256 finalBalance;
        uint256 percentOfFinalBal;
        uint256 yieldingPercentage;
        uint256 toDeposit;
        uint256 toWithdraw;
        uint256 yieldingAmount;
        uint256 finalYieldingAmount;
        int256 netAssetMovement;
        uint256 treasurySplit;
        uint256 stakingSplit;
        uint256 stabilityPoolSplit;
    }
```

### [NC-3] Function and modifier have the same functionality
Function isCollateralAllowed does exactly the same thing as modifier checkCollateral therefore I recommend to delete one.

*Instances (2)*:
```solidity
File: CollateralConfig.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L106-L108
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L128-L131

    function isCollateralAllowed(address _collateral) external override view returns (bool) {
        return collateralConfig[_collateral].allowed;
    }

    modifier checkCollateral(address _collateral) {
        require(collateralConfig[_collateral].allowed, "Invalid collateral address");
        _;
    }
```

### [NC-4] Interchangeable usage of uint and uint256
Sticking to one type improves readability and decreases possibility of making mistakes.

*Instances (Many instances)*:
```solidity
File: BorrowerOperations.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L47-L64
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L66-L78

struct LocalVariables_adjustTrove {
        uint256 collCCR;
        uint256 collMCR;
        uint256 collDecimals;
        uint price;
        uint collChange;
        uint netDebtChange;
        bool isCollIncrease;
        uint debt;
        uint coll;
        uint oldICR;
        uint newICR;
        uint newTCR;
        uint LUSDFee;
        uint newDebt;
        uint newColl;
        uint stake;
}

struct LocalVariables_openTrove {
        uint256 collCCR;
        uint256 collMCR;
        uint256 collDecimals;
        uint price;
        uint LUSDFee;
        uint netDebt;
        uint compositeDebt;
        uint ICR;
        uint NICR;
        uint stake;
        uint arrayIndex;
}

File: LUSDToken.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L75

    uint internal immutable deploymentStartTime;

File: StabilityPool.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L170-L229
    
```

### [NC-5] Incorrect use of uppercase letters for non constant variable
It's a good practice to use solidity standards to improve readability and avoid confusion/errors.

*Instances (1)*:
```solidity
File: LQTYStaking.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L29

    uint public F_LUSD; // Running sum of LUSD fees per-LQTY-staked
```

### [NC-6] Local and state variables should use mixedCase letters
It's a good practice to use solidity standards to improve readability and avoid confusion/errors.

*Instances (2)*:
```solidity
File: LQTYStaking.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L28
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L34-L37

    mapping (address => uint) public F_Collateral;  // Running sum of collateral fees per-LQTY-staked

    struct Snapshot {
        mapping (address => uint) F_Collateral_Snapshot;
        uint F_LUSD_Snapshot;
    }
```

### [NC-7] Defined events are not used
Not used events should be deleted to avoid confusion.

*Instances (2)*:
```solidity
File: LQTYStaking.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L50

    event LUSDTokenAddressSet(address _lusdTokenAddress);

File: StabilityPool.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L240

    event DefaultPoolAddressChanged(address _newDefaultPoolAddress);


```

### [NC-8] Usage of keyword 'super' will improve readability
Usage of 'super' keyword when inheriting function from different contract will improve readability.

*Instances (1)*:
```solidity
File: ReaperVaultERC4626.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L37-L39

    function totalAssets() external view override returns (uint256 totalManagedAssets) {
        return balance();
    }


```

### [NC-9] One letter and uppercased variables
It's a good practice to use solidity standards and use descriptive names for variables.

*Instances (2)*:
```solidity
File: StabilityPool.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L178-L184
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L195

    struct Snapshots {
        mapping (address => uint) S;
        uint P;
        uint G;
        uint128 scale;
        uint128 epoch;
    }

    uint public P = DECIMAL_PRECISION;


```