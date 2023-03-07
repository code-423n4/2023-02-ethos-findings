## Summary

### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]| `initialize()` function can be called by anybody | 3 |
|[L-02]| Hardcode the address causes no future updates | 4 |
|[L-03]| Critical Set Changes Should Use Two-step Procedure | 1 |
|[L-04]| Lack of ACK during owner change | 6 |
|[L-05]| Use a more recent version of Solidity | ALL CONTRACT |
|[L-06]| Open TODOs in Contract | 2 |
|[L-07]| Rug vectors by the owner frontrun | 2 |
|[L-08]| ReaperVault implmentation is not fully up to EIP-4626's specification | 2 |
|[L-09]| Lack of Input Validation | 10 |
|[L-10]| Loss of precision due to rounding | 2 |


Total 24 issues

### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01]|Use require instead of assert|24|
| [N-02]|NatSpec comments should be increased in contracts|ALL CONTRACT|
| [N-03]|For modern and more readable code; update import usages|ALL CONTRACT|
| [N-04]|Function writing that does not comply with the Solidity Style Guide|ALL CONTRACT|
| [N-05]|Include return parameters in NatSpec comments|ALL CONTRACT|
| [N-06]|Use a single file for all system-wide constants|8|
| [N-07]|Add NatSpec comments to the variables defined in Storage|3|


Total 11 issues


### [L-01] `initialize()` function can be called by anybody
The `initialize()` function is a common function used in smart contracts to initialize the state variables of the contract. It is typically called only once during the deployment of the contract and is meant to set the initial values for the contract's state variables.

However, if the `initialize()` function is not properly secured, anyone can call it even after the contract has already been initialized. This can cause unexpected behavior or even create security vulnerabilities in the contract.

Furthermore, if the `initialize()` function modifies the state variable owner, then anyone who calls the function will become the new owner of the contract, which can give them full control over the contract. This is because the owner state variable typically has special permissions or privileges within the contract.

Therefore, it is important to ensure that the `initialize()` function is properly secured and only callable by authorized parties, such as the contract owner or a specific set of authorized addresses. This can help prevent unauthorized modifications to the contract's state variables and protect against potential security vulnerabilities.

``` solidity
ReaperStrategyGranarySupplyOnly.sol
62:     function initialize(
63:         address _vault,
64:         address[] memory _strategists,
65:         address[] memory _multisigRoles,
66:         IAToken _gWant
67:     ) public initializer {
```

``` solidity
CollateralConfig.sol
46:    function initialize(
47:         address[] calldata _collaterals,
48:         uint256[] calldata _MCRs,
49:         uint256[] calldata _CCRs
50:     ) external override onlyOwner {    
```

``` solidity
ReaperBaseStrategyv4.sol
63:  function __ReaperBaseStrategy_init(
64:         address _vault,
65:         address _want,
66:         address[] memory _strategists,
67:         address[] memory _multisigRoles
68:     ) internal onlyInitializing {
```

## Recommended Mitigation Steps
Add a control that makes initialize() only call the Deployer Contract or EOA
EG.
``` solidity
if (msg.sender != DEPLOYER_ADDRESS) {
            revert NotDeployer();
}
```

### [L-02] Hardcode the address causes no future updates

When an address is hardcoded as a constant, there is no option to update the address in the future without modifying the source code. Therefore, it is recommended to add an update option with the "onlyOwner" modifier. This way, only the contract owner can update the address if necessary, and changes to the address can be easily made without having to manually modify the source code.

``` solidity
ReaperStrategyGranarySupplyOnly.sol
24:     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;
25:     ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =
26:         ILendingPoolAddressesProvider(0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6);
27:     IAaveProtocolDataProvider public constant DATA_PROVIDER =
28:         IAaveProtocolDataProvider(0x9546F673eF71Ff666ae66d01Fd6E7C6Dae5a9995);
29:     IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);
```
## Recommended Mitigation Steps
It is important to consider the possibility of address changes in the future and make the source code flexible by adding the appropriate update option to avoid potential problems that may arise in the future.


### [L-03] Critical Set Changes Should Use Two-step Procedure

A two-step procedure for critical set changes can be beneficial in reducing errors and improving the overall reliability of critical operations. The first step of the procedure should involve a review and approval process, where the proposed change is evaluated by a team of experts to ensure that it is necessary, appropriate, and safe.

``` solidity
TroveManager.sol
1562:     function setTroveStatus(address _borrower, address _collateral, uint _num) external override {
1563:         _requireCallerIsBorrowerOperations();
1564:         Troves[_borrower][_collateral].status = Status(_num);
1565:     }
```

## Recommended Mitigation Steps
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions

### [L-04] Lack of ACK during owner change
It's possible to lose the ownership under specific circumstances.

Because an human error it's possible to set a new invalid owner. When you want to change the owner's address it's better to propose a new owner, and then accept this ownership with the new wallet.

``` solidity
ActivePool.sol
26: contract ActivePool is Ownable, CheckContract, IActivePool {
BorrowerOperations.sol
18: contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOperations {
CollateralConfig.sol
15: contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {
CommunityIssuance.sol
14: contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMath {
LQTYStaking.sol
18: contract LQTYStaking is ILQTYStaking, Ownable, CheckContract, BaseMath {
StabilityPool.sol
146: contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
```

### [L-05] Use a more recent version of Solidity

Context: All contracts

Description: For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

Recommendation: Old version of Solidity change to version can be used (0.8.17)

### [L-06] Open TODOs in Contract

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

``` solidity
StabilityPool.sol
335: /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
336:          * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
337:          * If needed could create a separate event just to report this.
338:          */


StabilityPool.sol
380:  /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
381:          * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
382:          * If needed could create a separate event just to report this.
383:          */
```


### [L-07] Rug vectors by the owner frontrun
setYieldingPercentage(address _collateral, uint256 _bps):
This function is used to set the yield percentage that will be received by the owner of a certain type of collateral, with the parameter _collateral being the address of that collateral and _bps being the number of basis points (BPS) of the yield to be received (1 BPS = 0.01%) only be accessed by the owner.

setYieldingPercentageDrift(uint256 _driftBps):
This function is used to set the drift value of the yield, with the parameter _driftBps being the number of BPS of the drift only be accessed by the owner.

setYieldClaimThreshold(address _collateral, uint256 _threshold):
This function is used to set the threshold value of the yield claim, with the parameter _collateral being the address of the collateral to be set and _threshold being the desired threshold value only be accessed by the owner.

setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit):
This function is used to set the distribution of yield between three parties (treasury, SP, and staking), with the parameters _treasurySplit, _SPSplit, and _stakingSplit being the number of BPS allocated to each party only be accessed by the owner.
``` solidity
File: ActivePool.sol
125:     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
126:         _requireValidCollateralAddress(_collateral);
127:         require(_bps <= 10_000, "Invalid BPS value");
128:         yieldingPercentage[_collateral] = _bps;
129:         emit YieldingPercentageUpdated(_collateral, _bps);
130:     }
131: 
132:     function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
133:         require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
134:         yieldingPercentageDrift = _driftBps;
135:         emit YieldingPercentageDriftUpdated(_driftBps);
136:     }
137: 
138:     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
139:         _requireValidCollateralAddress(_collateral);
140:         yieldClaimThreshold[_collateral] = _threshold;
141:         emit YieldClaimThresholdUpdated(_collateral, _threshold);
142:     }
143: 
144:     function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
145:         require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
146:         yieldSplitTreasury = _treasurySplit;
147:         yieldSplitSP = _SPSplit;
148:         yieldSplitStaking = _stakingSplit;
149:         emit YieldDistributionParamsUpdated(_treasurySplit, _SPSplit, _stakingSplit);
150:     }
```
## Recommended Mitigation Steps
As a mitigation add a timelock and make sure the owner is a multisig and not an EOA.

### [L-08] ReaperVault implmentation is not fully up to EIP-4626's specification

Must return the maximum amount of shares mint would allow to be deposited to receiver and not cause a revert, which must not be higher than the actual maximum that would be accepted (it should underestimate if necessary).

This assumes that the user has infinite assets, i.e. must not rely on balanceOf of asset.


ReaperVaultERC4626.sol
79:     function maxDeposit(address) external view override returns (uint256 maxAssets) {
80:         if (emergencyShutdown || balance() >= tvlCap) return 0;
81:         if (tvlCap == type(uint256).max) return type(uint256).max;
82:         return tvlCap - balance();
83:     }
84: 

ReaperVaultERC4626.sol
122:     function maxMint(address) external view override returns (uint256 maxShares) {
123:         if (emergencyShutdown || balance() >= tvlCap) return 0;
124:         if (tvlCap == type(uint256).max) return type(uint256).max;
125:         return convertToShares(tvlCap - balance());
126:     }

## Recommended Mitigation Steps
maxMint() and maxDeposit() should reflect the limitation of maxSupply.

Consider changing maxMint() and maxDeposit() to:

``` solidity

function maxMint(address) public view virtual returns (uint256) {
    if (totalSupply >= maxSupply) {
        return 0;
    }
    return maxSupply - totalSupply;
}

function maxDeposit(address) public view virtual returns (uint256) {
    return convertToAssets(maxMint(address(0)));
}

```

### [L-09] Lack of Input Validation
Lack of input validation is the process of checking input data to ensure that it is valid and safe to process. Therefore, it is important for contracts to consider the lack of input validation to ensure that what is produced is secure and protected from potential security vulnerabilities. This can be achieved by ensuring that every user input is thoroughly checked and validated at every level of data processing, from user interface to database. By doing so, the application can be reliable and protected from security threats that may arise due to lack of input validation.

## Recommended Mitigation Steps

```diff
ReaperVaultERC4626.sol
110:    function deposit(uint256 assets, address receiver) external override returns (uint256 shares) {
111:         shares = _deposit(assets, receiver);
112:     }
+	         require(amount <= maxDeposit(receiver), "deposit more than max");

ReaperVaultERC4626.sol
154:     function mint(uint256 shares, address receiver) external override returns (uint256 assets) {
155:         assets = previewMint(shares); // previewMint rounds up so exactly "shares" should be minted and not 1 wei less
156:         _deposit(assets, receiver);
157:     }
+	         require(amount <= maxDeposit(receiver), "deposit more than max");

ReaperVaultERC4626.sol
202:     function withdraw(
203:         uint256 assets,
204:         address receiver,
205:         address owner
206:     ) external override returns (uint256 shares) {
207:         shares = previewWithdraw(assets); // previewWithdraw() rounds up so exactly "assets" are withdrawn and not 1 wei less
208:         if (msg.sender != owner) _spendAllowance(owner, msg.sender, shares);
209:         _withdraw(shares, receiver, owner);
210:     }
+                require(assets <= maxWithdraw(owner), "withdraw more than max");

ReaperVaultERC4626.sol
258:     function redeem(
259:         uint256 shares,
260:         address receiver,
261:         address owner
262:     ) external override returns (uint256 assets) {
263:         if (msg.sender != owner) _spendAllowance(owner, msg.sender, shares);
264:         assets = _withdraw(shares, receiver, owner);
265:     }
+               require(shares <= maxRedeem(owner), "redeem more than max");

```

Parameters using the isContract function from the OpenZeppelin library

``` diff
BorrowerOperations.sol
172:  function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {
+    require(_collAmount > 0, "Collateral amount must be greater than zero");
+    require(_LUSDAmount > 0, "LUSD amount must be greater than zero");
```

``` diff
LQTYStaking.sol
187:     function increaseF_LUSD(uint _LUSDFee) external override {
+    require(_LUSDFee > 0, "LUSDFee must be greater than zero");    
}

```

``` diff
function _transfer(address sender, address recipient, uint256 amount) internal {
+   require(amount > 0, "Transfer amount must be greater than 0");
}

function _mint(address account, uint256 amount) internal {
+   require(amount > 0, "Transfer amount must be greater than 0");
}
function _burn(address account, uint256 amount) internal {
+   require(amount > 0, "Transfer amount must be greater than 0");
}
function _harvestCore(uint256 _debt) internal override returns (int256 roi, uint256 repayment) {
+    require(_debt <= balanceOf(), "Debt cannot exceed total assets");
+    require(startToken.balanceOf(address(this)) >= amount, "Insufficient startToken balance");
+    require(allocated <= totalAssets, "Allocated amount cannot exceed total assets");
+    require(amountFreed >= repayment, "Amount freed is less than debt");
}
```

### [L-10] Loss of precision due to rounding
``` solidity
ReaperVaultERC4626.sol
51:     function convertToShares(uint256 assets) public view override returns (uint256 shares) {
52:         if (totalSupply() == 0 || _freeFunds() == 0) return assets;
53:         return (assets * totalSupply()) / _freeFunds();
54:     }


ReaperVaultERC4626.sol
65: 
66:     function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
67:         if (totalSupply() == 0) return shares;
68:         return (shares * _freeFunds()) / totalSupply();
69:     }
70: 
```


### [N-1] Use require instead of assert

Description: Assert should not be used except for tests, require should be used

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process's available gas instead of returning it, as request()/revert() did.

assert() and ruqire(); The big difference between the two is that the assert()function when false, uses up all the remaining gas and reverts all the changes made. Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states "The Assert function generates an error of type Panic(uint256).Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there's a mistake".


``` solidity
File: BorrowerOperations.sol
128:       assert(MIN_NET_DEBT > 0);
File: BorrowerOperations.sol
197:    assert(vars.compositeDebt > 0);
File: BorrowerOperations.sol
301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0)); 
File: BorrowerOperations.sol
331:         assert(_collWithdrawal <= vars.coll); 
File: LUSDToken.sol
312:         assert(sender != address(0));
313:         assert(recipient != address(0));
File: LUSDToken.sol
312:         assert(sender != address(0));
313:         assert(recipient != address(0));
File: LUSDToken.sol
321:         assert(account != address(0));
File: LUSDToken.sol
329:         assert(account != address(0));       
File: LUSDToken.sol
337:         assert(owner != address(0));
338:         assert(spender != address(0));
File: LUSDToken.sol
337:         assert(owner != address(0));
338:         assert(spender != address(0));
File: StabilityPool.sol
526:         assert(_debtToOffset <= _totalLUSDDeposits);
File: StabilityPool.sol
551:        assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
File: StabilityPool.sol
591:         assert(newP > 0);
File: TroveManager.sol
417:             assert(_LUSDInStabPool != 0);
File: TroveManager.sol
1224:             assert(totalStakesSnapshot[_collateral] > 0);
File: TroveManager.sol
1279:         assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
File: TroveManager.sol
1342:         assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
File: TroveManager.sol
1348:         assert(index <= idxLast); 
File: TroveManager.sol
1414:         assert(newBaseRate > 0); // Base rate is always non-zero after redemption
File: TroveManager.sol
1489:         assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
``` 

### [N-02] NatSpec comments should be increased in contracts
Context: All Contracts

Description: It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

Recommendation: NatSpec comments should be increased in contracts

### [N-03] For modern and more readable code; update import usages
Description: Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

## Recommendation
import {contract1 , contract2} from "filename.sol";

### [N-04] Function writing that does not comply with the Solidity Style Guide

Context: All Contracts

Description: Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor receive function (if exists) fallback function (if exists) external public internal private within a grouping, place the view and pure functions last


### [N-05] Include return parameters in NatSpec comments

Context: All Contracts

Description: It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## Recommendation
Include return parameters in NatSpec comments

``` diff
+/// @return add comment for return parameters
eaperVaultERC4626.sol
240:    function previewRedeem(uint256 shares) external view override returns (uint256 assets) {
241:         return convertToAssets(shares);
242:     }
```

### [N-06] Include return parameters in NatSpec comments

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values)

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.left Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

``` solidity
CollateralConfig.sol
20:     // Smallest allowed value for the minimum collateral ratio for individual troves in each market (collateral)
21:     uint256 constant public MIN_ALLOWED_MCR = 1.1 ether; // 110%
22: 
23:     // Smallest allowed value for Critical system collateral ratio.
24:     // If a market's (collateral's) total collateral ratio (TCR) falls below the CCR, Recovery Mode is triggered.
25:     uint256 constant public MIN_ALLOWED_CCR = 1.5 ether; // 150%

LUSDToken.sol
41:     // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)");
42:     bytes32 private constant _PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;
43:     // keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");
44:     bytes32 private constant _TYPE_HASH = 0x8b73c3c69bb8fe3d512ecc4cf759cc79239f7b179b0ffacaa9a75d522b39400f;
45: 
46:     // Cache the domain separator as an immutable value, but also store the chain id that it corresponds to, in order to
47:     // invalidate the cached domain separator if the chain id changes.
48:     bytes32 private immutable _CACHED_DOMAIN_SEPARATOR;
49:     uint256 private immutable _CACHED_CHAIN_ID;
50: 
51:     bytes32 private immutable _HASHED_NAME;
52:     bytes32 private immutable _HASHED_VERSION;


ReaperBaseStrategyv4.sol
23:     uint256 public constant PERCENT_DIVISOR = 10_000;
24:     uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF
25:     uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;

ReaperBaseStrategyv4.sol
49:     bytes32 public constant KEEPER = keccak256("KEEPER");
50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52:     bytes32 public constant ADMIN = keccak256("ADMIN");

FReaperStrategyGranarySupplyOnly.sol
24:     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;
25:     ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =
26:         ILendingPoolAddressesProvider(0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6);
27:     IAaveProtocolDataProvider public constant DATA_PROVIDER =
28:         IAaveProtocolDataProvider(0x9546F673eF71Ff666ae66d01Fd6E7C6Dae5a9995);
29:     IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);
30: 

ReaperStrategyGranarySupplyOnly.sol
35:     uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;

TroveManager.sol
53:     uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;
54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
55:     uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%

FTroveManager.sol
61:     uint constant public BETA = 2;
```


### [N-07] Add NatSpec comments to the variables defined in Storage

Description: I recommend adding NatSpec comments explaining the variables defined in Storage, their slots, their contents and definitions.

This improves code readability and control quality

Current Code;

``` solidity
ActivePool.sol
30:   string constant public NAME = "ActivePool";
31: 
32:     bool public addressesSet = false;
33:     address public collateralConfigAddress;
34:     address public borrowerOperationsAddress;
35:     address public troveManagerAddress;
36:     address public stabilityPoolAddress;
37:     address public defaultPoolAddress;
38:     address public collSurplusPoolAddress;
39:     address public treasuryAddress;
40:     address public lqtyStakingAddress;


LQTYStaking.sol
39:     IERC20 public lqtyToken;
40:     ILUSDToken public lusdToken;
41:     ICollateralConfig public collateralConfig;
42: 
43:     address public troveManagerAddress;
44:     address public borrowerOperationsAddress;
45:     address public activePoolAddress;

LUSDToken.sol
31:     uint256 private _totalSupply;
32:     string constant internal _NAME = "LUSD Stablecoin";
33:     string constant internal _SYMBOL = "LUSD";
34:     string constant internal _VERSION = "1";
35:     uint8 constant internal _DECIMALS = 18;
36: 
37:     bool public mintingPaused = false;
```
