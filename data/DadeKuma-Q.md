
## Summary

---

### Low Risk Findings
|Id|Title|Instances|
|:--:|:-------|:--:|
|[L-01]| Lack of input validation in ERC4626 functions | 1 |
|[L-02]| Hardcoded addresses can be problematic | 1 |
|[L-03]| `LUSDToken` can mint to `StabilityPool`, `TroveManager`, and `BorrowerOperation` | 1 |
|[L-04]| `distributionPeriod` must be > 0 to avoid locked funds | 1 |
|[L-05]| Missing `transfer` value check for ERC-20 | 2 |
|[L-06]| Lack of two-step update for critical addresses | 15 |
|[L-07]| Missing events in sensitive functions | 7 |
|[L-08]| Owner can renounce ownership | 22 |

Total: 50 instances over 8 issues.

### Non Critical Findings
|Id|Title|Instances|
|:--:|:-------|:--:|
|[NC-01]| Low test coverage | 1 |
|[NC-02]| Unused named `return` is confusing | 26 |
|[NC-03]| Same constant redefined elsewhere | 8 |
|[NC-04]| Use of `constant` variables instead of `immutable` | 8 |
|[NC-05]| Use `require` instead of `assert` | 21 |
|[NC-06]| `require` without any message | 18 |
|[NC-07]| Some functions don't follow the Solidity naming conventions | 1 |
|[NC-08]| Old Solidity version | 33 |
|[NC-09]| Use of floating pragma | 18 |
|[NC-10]| Missing or incomplete NatSpec | 62 |

Total: 196 instances over 10 issues.

## Low Risk Findings

---

### [L-01] Lack of input validation in ERC4626 functions



Functions `deposit, mint, withdraw, redeem` lack a validation to check if the amount exceed the max.

```solidity
// Deposit
require(assets <= maxDeposit(receiver), "ERC4626: deposit more than max");

// Mint
require(shares <= maxMint(receiver), "ERC4626: mint more than max");

// Withdraw
require(assets <= maxWithdraw(owner), "ERC4626: withdraw more than max");

// Redeem
require(shares <= maxRedeem(owner), "ERC4626: redeem more than max");
```


*There is 1 instance of this issue.*


```solidity
File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol

110: 		    function deposit(uint256 assets, address receiver) external override returns (uint256 shares) {
111: 		        shares = _deposit(assets, receiver);
112: 		    }
154: 		    function mint(uint256 shares, address receiver) external override returns (uint256 assets) {
155: 		        assets = previewMint(shares); // previewMint rounds up so exactly "shares" should be minted and not 1 wei less
156: 		        _deposit(assets, receiver);
157: 		    }
202: 		    function withdraw(
203: 		        uint256 assets,
204: 		        address receiver,
205: 		        address owner
206: 		    ) external override returns (uint256 shares) {
207: 		        shares = previewWithdraw(assets); // previewWithdraw() rounds up so exactly "assets" are withdrawn and not 1 wei less
208: 		        if (msg.sender != owner) _spendAllowance(owner, msg.sender, shares);
209: 		        _withdraw(shares, receiver, owner);
210: 		    }
258: 		    function redeem(
259: 		        uint256 shares,
260: 		        address receiver,
261: 		        address owner
262: 		    ) external override returns (uint256 assets) {
263: 		        if (msg.sender != owner) _spendAllowance(owner, msg.sender, shares);
264: 		        assets = _withdraw(shares, receiver, owner);
265: 		    }

```

### [L-02] Hardcoded addresses can be problematic


There are some hardcoded addresses in `ReaperStrategyGranarySupplyOnly` that could lead to unexpected behaviour if the contract it's deployed on a different chain than what it's currently intended. Add the addresses to the constructor, and assign them to an immutable variable.


*There is 1 instance of this issue.*


```solidity
File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

24: 		    address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;
25: 		    ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =
26: 		        ILendingPoolAddressesProvider(0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6);
27: 		    IAaveProtocolDataProvider public constant DATA_PROVIDER =
28: 		        IAaveProtocolDataProvider(0x9546F673eF71Ff666ae66d01Fd6E7C6Dae5a9995);
29: 		    IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);

```

### [L-03] `LUSDToken` can mint to `StabilityPool`, `TroveManager`, and `BorrowerOperation`


Function `_requireValidRecipient` is used to avoid transferring `LUSDToken` to `StabilityPool`, `TroveManager`, and `BorrowerOperation`.

However, it's possible to mint `LUSDToken` directly to these addresses.

There is no `_requireValidRecipient` check in `mint`, and this could have unexpected consequences.

```solidity
File: Ethos-Core\contracts\LUSDToken.sol

236:     function transfer(
237:         address recipient,
238:         uint256 amount
239:     ) external override returns (bool) {
240:         _requireValidRecipient(recipient);
241:         _transfer(msg.sender, recipient, amount);
242:         return true;
243:     }

433:     function _requireValidRecipient(address _recipient) internal view {
434:         require(
435:             _recipient != address(0) && _recipient != address(this),
436:             "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
437:         );
438:         require(
439:             !stabilityPools[_recipient] &&
440:                 !troveManagers[_recipient] &&
441:                 !borrowerOperations[_recipient],
442:             "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
443:         );
444:     }

```


*There is 1 instance of this issue.*


```solidity
File: audits/2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

191: 		    function burn(address _account, uint256 _amount) external override {
192: 		        _requireCallerIsBOorTroveMorSP();
193: 		        _burn(_account, _amount);
194: 		    }
195: 		

```

### [L-04] `distributionPeriod` must be > 0 to avoid locked funds


If `distributionPeriod` is set to zero, it won't be possible to fund the contract as there would be a division by zero in `fund`. 

Add a `require(distributionPeriod > 0, "distributionPeriod = 0")` to the setter to avoid this situation.


*There is 1 instance of this issue.*


```solidity
File: audits/2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

121: 		        distributionPeriod = _newDistributionPeriod;
122: 		    }
123: 		

```

### [L-05] Missing `transfer` value check for ERC-20


Add a `require` to be sure that the transaction was a success, or use `safeTransfer`.


*There are 2 instances of this issue.*


```solidity
File: audits/2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

128: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

135: 		            lusdToken.transfer(msg.sender, LUSDGain);
171: 		        lusdToken.transfer(msg.sender, LUSDGain);

```

### [L-06] Lack of two-step update for critical addresses

Add a two-step process for any critical address changes.

*There are 15 instances of this issue.*

<details>
<summary>Expand findings</summary>


```solidity
File: audits/2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

71: 		    function setAddresses(
72: 		        address _collateralConfigAddress,
73: 		        address _borrowerOperationsAddress,
74: 		        address _troveManagerAddress,
75: 		        address _stabilityPoolAddress,
76: 		        address _defaultPoolAddress,
77: 		        address _collSurplusPoolAddress,
78: 		        address _treasuryAddress,
79: 		        address _lqtyStakingAddress,
80: 		        address[] calldata _erc4626vaults
81: 		    )
82: 		        external
83: 		        onlyOwner
84: 		    {
85: 		        require(!addressesSet, "Can call setAddresses only once");
86: 		
87: 		        checkContract(_collateralConfigAddress);
88: 		        checkContract(_borrowerOperationsAddress);
89: 		        checkContract(_troveManagerAddress);
90: 		        checkContract(_stabilityPoolAddress);
91: 		        checkContract(_defaultPoolAddress);
92: 		        checkContract(_collSurplusPoolAddress);
93: 		        require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
94: 		        checkContract(_lqtyStakingAddress);
95: 		
96: 		        collateralConfigAddress = _collateralConfigAddress;
97: 		        borrowerOperationsAddress = _borrowerOperationsAddress;
98: 		        troveManagerAddress = _troveManagerAddress;
99: 		        stabilityPoolAddress = _stabilityPoolAddress;
100: 		        defaultPoolAddress = _defaultPoolAddress;
101: 		        collSurplusPoolAddress = _collSurplusPoolAddress;
102: 		        treasuryAddress = _treasuryAddress;
103: 		        lqtyStakingAddress = _lqtyStakingAddress;
104: 		
105: 		        address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();
106: 		        uint256 numCollaterals = collaterals.length;
107: 		        require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");
108: 		        for(uint256 i = 0; i < numCollaterals; i++) {
109: 		            address collateral = collaterals[i];
110: 		            address vault = _erc4626vaults[i];
111: 		            require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");
112: 		            yieldGenerator[collateral] = vault;
113: 		        }
114: 		
115: 		        emit CollateralConfigAddressChanged(_collateralConfigAddress);
116: 		        emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);
117: 		        emit TroveManagerAddressChanged(_troveManagerAddress);
118: 		        emit StabilityPoolAddressChanged(_stabilityPoolAddress);
119: 		        emit DefaultPoolAddressChanged(_defaultPoolAddress);
120: 		        emit CollSurplusPoolAddressChanged(_collSurplusPoolAddress);
121: 		
122: 		        addressesSet = true;
123: 		    }

125: 		    function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
126: 		        _requireValidCollateralAddress(_collateral);
127: 		        require(_bps <= 10_000, "Invalid BPS value");
128: 		        yieldingPercentage[_collateral] = _bps;
129: 		        emit YieldingPercentageUpdated(_collateral, _bps);
130: 		    }

138: 		    function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
139: 		        _requireValidCollateralAddress(_collateral);
140: 		        yieldClaimThreshold[_collateral] = _threshold;
141: 		        emit YieldClaimThresholdUpdated(_collateral, _threshold);
142: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

110: 		    function setAddresses(
111: 		        address _collateralConfigAddress,
112: 		        address _troveManagerAddress,
113: 		        address _activePoolAddress,
114: 		        address _defaultPoolAddress,
115: 		        address _stabilityPoolAddress,
116: 		        address _gasPoolAddress,
117: 		        address _collSurplusPoolAddress,
118: 		        address _priceFeedAddress,
119: 		        address _sortedTrovesAddress,
120: 		        address _lusdTokenAddress,
121: 		        address _lqtyStakingAddress
122: 		    )
123: 		        external
124: 		        override
125: 		        onlyOwner
126: 		    {
127: 		        // This makes impossible to open a trove with zero withdrawn LUSD
128: 		        assert(MIN_NET_DEBT > 0);
129: 		
130: 		        checkContract(_collateralConfigAddress);
131: 		        checkContract(_troveManagerAddress);
132: 		        checkContract(_activePoolAddress);
133: 		        checkContract(_defaultPoolAddress);
134: 		        checkContract(_stabilityPoolAddress);
135: 		        checkContract(_gasPoolAddress);
136: 		        checkContract(_collSurplusPoolAddress);
137: 		        checkContract(_priceFeedAddress);
138: 		        checkContract(_sortedTrovesAddress);
139: 		        checkContract(_lusdTokenAddress);
140: 		        checkContract(_lqtyStakingAddress);
141: 		
142: 		        collateralConfig = ICollateralConfig(_collateralConfigAddress);
143: 		        troveManager = ITroveManager(_troveManagerAddress);
144: 		        activePool = IActivePool(_activePoolAddress);
145: 		        defaultPool = IDefaultPool(_defaultPoolAddress);
146: 		        stabilityPoolAddress = _stabilityPoolAddress;
147: 		        gasPoolAddress = _gasPoolAddress;
148: 		        collSurplusPool = ICollSurplusPool(_collSurplusPoolAddress);
149: 		        priceFeed = IPriceFeed(_priceFeedAddress);
150: 		        sortedTroves = ISortedTroves(_sortedTrovesAddress);
151: 		        lusdToken = ILUSDToken(_lusdTokenAddress);
152: 		        lqtyStakingAddress = _lqtyStakingAddress;
153: 		        lqtyStaking = ILQTYStaking(_lqtyStakingAddress);
154: 		
155: 		        emit CollateralConfigAddressChanged(_collateralConfigAddress);
156: 		        emit TroveManagerAddressChanged(_troveManagerAddress);
157: 		        emit ActivePoolAddressChanged(_activePoolAddress);
158: 		        emit DefaultPoolAddressChanged(_defaultPoolAddress);
159: 		        emit StabilityPoolAddressChanged(_stabilityPoolAddress);
160: 		        emit GasPoolAddressChanged(_gasPoolAddress);
161: 		        emit CollSurplusPoolAddressChanged(_collSurplusPoolAddress);
162: 		        emit PriceFeedAddressChanged(_priceFeedAddress);
163: 		        emit SortedTrovesAddressChanged(_sortedTrovesAddress);
164: 		        emit LUSDTokenAddressChanged(_lusdTokenAddress);
165: 		        emit LQTYStakingAddressChanged(_lqtyStakingAddress);
166: 		
167: 		        _renounceOwnership();
168: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

85: 		    function updateCollateralRatios(
86: 		        address _collateral,
87: 		        uint256 _MCR,
88: 		        uint256 _CCR
89: 		    ) external onlyOwner checkCollateral(_collateral) {
90: 		        Config storage config = collateralConfig[_collateral];
91: 		        require(_MCR <= config.MCR, "Can only walk down the MCR");
92: 		        require(_CCR <= config.CCR, "Can only walk down the CCR");
93: 		
94: 		        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
95: 		        config.MCR = _MCR;
96: 		
97: 		        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
98: 		        config.CCR = _CCR;
99: 		        emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
100: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol

41: 		    function setAddresses(
42: 		        address _collateralConfigAddress,
43: 		        address _borrowerOperationsAddress,
44: 		        address _troveManagerAddress,
45: 		        address _activePoolAddress
46: 		    )
47: 		        external
48: 		        override
49: 		        onlyOwner
50: 		    {
51: 		        checkContract(_collateralConfigAddress);
52: 		        checkContract(_borrowerOperationsAddress);
53: 		        checkContract(_troveManagerAddress);
54: 		        checkContract(_activePoolAddress);
55: 		
56: 		        collateralConfigAddress = _collateralConfigAddress;
57: 		        borrowerOperationsAddress = _borrowerOperationsAddress;
58: 		        troveManagerAddress = _troveManagerAddress;
59: 		        activePoolAddress = _activePoolAddress;
60: 		
61: 		        emit CollateralConfigAddressChanged(_collateralConfigAddress);
62: 		        emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);
63: 		        emit TroveManagerAddressChanged(_troveManagerAddress);
64: 		        emit ActivePoolAddressChanged(_activePoolAddress);
65: 		
66: 		        _renounceOwnership();
67: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol

40: 		    function setAddresses(
41: 		        address _collateralConfigAddress,
42: 		        address _troveManagerAddress,
43: 		        address _activePoolAddress
44: 		    )
45: 		        external
46: 		        onlyOwner
47: 		    {
48: 		        checkContract(_collateralConfigAddress);
49: 		        checkContract(_troveManagerAddress);
50: 		        checkContract(_activePoolAddress);
51: 		
52: 		        collateralConfigAddress = _collateralConfigAddress;
53: 		        troveManagerAddress = _troveManagerAddress;
54: 		        activePoolAddress = _activePoolAddress;
55: 		
56: 		        emit CollateralConfigAddressChanged(_collateralConfigAddress);
57: 		        emit TroveManagerAddressChanged(_troveManagerAddress);
58: 		        emit ActivePoolAddressChanged(_activePoolAddress);
59: 		
60: 		        _renounceOwnership();
61: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol

27: 		    function setAddresses(
28: 		        address _collateralConfigAddress,
29: 		        address _sortedTrovesAddress,
30: 		        address _troveManagerAddress
31: 		    )
32: 		        external
33: 		        onlyOwner
34: 		    {
35: 		        checkContract(_collateralConfigAddress);
36: 		        checkContract(_sortedTrovesAddress);
37: 		        checkContract(_troveManagerAddress);
38: 		
39: 		        collateralConfig = ICollateralConfig(_collateralConfigAddress);
40: 		        sortedTroves = ISortedTroves(_sortedTrovesAddress);
41: 		        troveManager = ITroveManager(_troveManagerAddress);
42: 		
43: 		        emit CollateralConfigAddressChanged(_collateralConfigAddress);
44: 		        emit SortedTrovesAddressChanged(_sortedTrovesAddress);
45: 		        emit TroveManagerAddressChanged(_troveManagerAddress);
46: 		
47: 		        _renounceOwnership();
48: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol

90: 		    function setAddresses(
91: 		        address _collateralConfigAddress,
92: 		        address[] calldata _priceAggregatorAddresses,
93: 		        address _tellorCallerAddress,
94: 		        bytes32[] calldata _tellorQueryIds
95: 		    )
96: 		        external
97: 		        onlyOwner
98: 		    {
99: 		        require(!initialized, "Can only initialize once");
100: 		        checkContract(_collateralConfigAddress);
101: 		        collateralConfig = ICollateralConfig(_collateralConfigAddress);
102: 		        emit CollateralConfigAddressChanged(_collateralConfigAddress);
103: 		
104: 		        address[] memory collaterals = collateralConfig.getAllowedCollaterals();
105: 		        uint256 numCollaterals = collaterals.length;
106: 		        require(numCollaterals != 0, "At least one collateral required");
107: 		        require(_priceAggregatorAddresses.length == numCollaterals, "Array lengths must match");
108: 		        require(_tellorQueryIds.length == numCollaterals, "Array lengths must match");
109: 		
110: 		        checkContract(_tellorCallerAddress);
111: 		        tellorCaller = ITellorCaller(_tellorCallerAddress);
112: 		
113: 		        for (uint256 i = 0; i < numCollaterals; i++) {
114: 		            address collateral = collaterals[i];
115: 		            address priceAggregatorAddress = _priceAggregatorAddresses[i];
116: 		            bytes32 queryId = _tellorQueryIds[i];
117: 		
118: 		            checkContract(priceAggregatorAddress);
119: 		            require(queryId != bytes32(0), "Invalid Tellor Query ID");
120: 		
121: 		            priceAggregator[collateral] = AggregatorV3Interface(priceAggregatorAddress);
122: 		            tellorQueryId[collateral] = queryId;
123: 		
124: 		            // Explicitly set initial system status
125: 		            status[collateral] = Status.chainlinkWorking;
126: 		
127: 		            // Get an initial price from Chainlink to serve as first reference for lastGoodPrice
128: 		            ChainlinkResponse memory chainlinkResponse = _getCurrentChainlinkResponse(collateral);
129: 		            ChainlinkResponse memory prevChainlinkResponse = _getPrevChainlinkResponse(collateral, chainlinkResponse.roundId, chainlinkResponse.decimals);
130: 		
131: 		            require(!_chainlinkIsBroken(chainlinkResponse, prevChainlinkResponse) && !_chainlinkIsFrozen(chainlinkResponse), 
132: 		                "PriceFeed: Chainlink must be working and current");
133: 		
134: 		            _storeChainlinkPrice(collateral, chainlinkResponse);
135: 		        }
136: 		
137: 		        initialized = true;
138: 		    }

143: 		    function updateChainlinkAggregator(
144: 		        address _collateral,
145: 		        address _priceAggregatorAddress
146: 		    ) external onlyOwner {
147: 		        _requireValidCollateralAddress(_collateral);
148: 		        checkContract(_priceAggregatorAddress);
149: 		        priceAggregator[_collateral] = AggregatorV3Interface(_priceAggregatorAddress);
150: 		
151: 		        // Explicitly set initial system status
152: 		        status[_collateral] = Status.chainlinkWorking;
153: 		
154: 		        // Get an initial price from Chainlink to serve as first reference for lastGoodPrice
155: 		        ChainlinkResponse memory chainlinkResponse = _getCurrentChainlinkResponse(_collateral);
156: 		        ChainlinkResponse memory prevChainlinkResponse = _getPrevChainlinkResponse(_collateral, chainlinkResponse.roundId, chainlinkResponse.decimals);
157: 		
158: 		        require(!_chainlinkIsBroken(chainlinkResponse, prevChainlinkResponse) && !_chainlinkIsFrozen(chainlinkResponse),
159: 		            "PriceFeed: Chainlink must be working and current");
160: 		
161: 		        _storeChainlinkPrice(_collateral, chainlinkResponse);
162: 		    }

167: 		    function updateTellorCaller(
168: 		        address _tellorCallerAddress
169: 		    ) external onlyOwner {
170: 		        checkContract(_tellorCallerAddress);
171: 		        tellorCaller = ITellorCaller(_tellorCallerAddress);
172: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol

80: 		    function setParams(address _troveManagerAddress, address _borrowerOperationsAddress) external override onlyOwner {
81: 		        checkContract(_troveManagerAddress);
82: 		        checkContract(_borrowerOperationsAddress);
83: 		
84: 		        troveManager = ITroveManager(_troveManagerAddress);
85: 		        borrowerOperationsAddress = _borrowerOperationsAddress;
86: 		
87: 		        emit TroveManagerAddressChanged(_troveManagerAddress);
88: 		        emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);
89: 		
90: 		        _renounceOwnership();
91: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

261: 		    function setAddresses(
262: 		        address _borrowerOperationsAddress,
263: 		        address _collateralConfigAddress,
264: 		        address _troveManagerAddress,
265: 		        address _activePoolAddress,
266: 		        address _lusdTokenAddress,
267: 		        address _sortedTrovesAddress,
268: 		        address _priceFeedAddress,
269: 		        address _communityIssuanceAddress
270: 		    )
271: 		        external
272: 		        override
273: 		        onlyOwner
274: 		    {
275: 		        checkContract(_borrowerOperationsAddress);
276: 		        checkContract(_collateralConfigAddress);
277: 		        checkContract(_troveManagerAddress);
278: 		        checkContract(_activePoolAddress);
279: 		        checkContract(_lusdTokenAddress);
280: 		        checkContract(_sortedTrovesAddress);
281: 		        checkContract(_priceFeedAddress);
282: 		        checkContract(_communityIssuanceAddress);
283: 		
284: 		        borrowerOperations = IBorrowerOperations(_borrowerOperationsAddress);
285: 		        collateralConfig = ICollateralConfig(_collateralConfigAddress);
286: 		        troveManager = ITroveManager(_troveManagerAddress);
287: 		        activePool = IActivePool(_activePoolAddress);
288: 		        lusdToken = ILUSDToken(_lusdTokenAddress);
289: 		        sortedTroves = ISortedTroves(_sortedTrovesAddress);
290: 		        priceFeed = IPriceFeed(_priceFeedAddress);
291: 		        communityIssuance = ICommunityIssuance(_communityIssuanceAddress);
292: 		
293: 		        emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);
294: 		        emit CollateralConfigAddressChanged(_collateralConfigAddress);
295: 		        emit TroveManagerAddressChanged(_troveManagerAddress);
296: 		        emit ActivePoolAddressChanged(_activePoolAddress);
297: 		        emit LUSDTokenAddressChanged(_lusdTokenAddress);
298: 		        emit SortedTrovesAddressChanged(_sortedTrovesAddress);
299: 		        emit PriceFeedAddressChanged(_priceFeedAddress);
300: 		        emit CommunityIssuanceAddressChanged(_communityIssuanceAddress);
301: 		
302: 		        _renounceOwnership();
303: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

61: 		    function setAddresses
62: 		    (
63: 		        address _oathTokenAddress, 
64: 		        address _stabilityPoolAddress
65: 		    ) 
66: 		        external 
67: 		        onlyOwner 
68: 		        override 
69: 		    {
70: 		        require(!initialized, "issuance has been initialized");
71: 		        checkContract(_oathTokenAddress);
72: 		        checkContract(_stabilityPoolAddress);
73: 		
74: 		        OathToken = IERC20(_oathTokenAddress);
75: 		        stabilityPoolAddress = _stabilityPoolAddress;
76: 		
77: 		        initialized = true;
78: 		
79: 		        emit OathTokenAddressSet(_oathTokenAddress);
80: 		        emit StabilityPoolAddressSet(_stabilityPoolAddress);
81: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

66: 		    function setAddresses
67: 		    (
68: 		        address _lqtyTokenAddress,
69: 		        address _lusdTokenAddress,
70: 		        address _troveManagerAddress, 
71: 		        address _borrowerOperationsAddress,
72: 		        address _activePoolAddress,
73: 		        address _collateralConfigAddress
74: 		    ) 
75: 		        external 
76: 		        onlyOwner 
77: 		        override 
78: 		    {
79: 		        checkContract(_lqtyTokenAddress);
80: 		        checkContract(_lusdTokenAddress);
81: 		        checkContract(_troveManagerAddress);
82: 		        checkContract(_borrowerOperationsAddress);
83: 		        checkContract(_activePoolAddress);
84: 		        checkContract(_collateralConfigAddress);
85: 		
86: 		        lqtyToken = IERC20(_lqtyTokenAddress);
87: 		        lusdToken = ILUSDToken(_lusdTokenAddress);
88: 		        troveManagerAddress = _troveManagerAddress;
89: 		        borrowerOperationsAddress = _borrowerOperationsAddress;
90: 		        activePoolAddress = _activePoolAddress;
91: 		        collateralConfig = ICollateralConfig(_collateralConfigAddress);
92: 		
93: 		        emit LQTYTokenAddressSet(_lqtyTokenAddress);
94: 		        emit LQTYTokenAddressSet(_lusdTokenAddress);
95: 		        emit TroveManagerAddressSet(_troveManagerAddress);
96: 		        emit BorrowerOperationsAddressSet(_borrowerOperationsAddress);
97: 		        emit ActivePoolAddressSet(_activePoolAddress);
98: 		        emit CollateralConfigAddressSet(_collateralConfigAddress);
99: 		
100: 		        _renounceOwnership();
101: 		    }

```
</details>

---
### [L-07] Missing events in sensitive functions

Events should be emitted when sensitive changes are made to the contracts, but some functions lack them.

*There are 7 instances of this issue.*

<details>
<summary>Expand findings</summary>


```solidity

File: audits/2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol

167: 		    function updateTellorCaller(
168: 		        address _tellorCallerAddress
169: 		    ) external onlyOwner {
170: 		        checkContract(_tellorCallerAddress);
171: 		        tellorCaller = ITellorCaller(_tellorCallerAddress);
172: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol

54: 		    function setAddresses(
55: 		        IActivePool _activePool,
56: 		        IDefaultPool _defaultPool,
57: 		        ITroveManager _troveManager,
58: 		        ICollateralConfig _collateralConfig,
59: 		        IERC20 _lqtyToken,
60: 		        IPriceFeed _priceFeed,
61: 		        ILUSDToken _lusdToken,
62: 		        ISortedTroves _sortedTroves,
63: 		        ILQTYStaking _lqtyStaking
64: 		    ) external onlyOwner {
65: 		        activePool = _activePool;
66: 		        defaultPool = _defaultPool;
67: 		        troveManager = _troveManager;
68: 		        collateralConfig = _collateralConfig;
69: 		        lqtyToken = _lqtyToken;
70: 		        priceFeed = _priceFeed;
71: 		        lusdToken = _lusdToken;
72: 		        sortedTroves = _sortedTroves;
73: 		        lqtyStaking = _lqtyStaking;
74: 		
75: 		        _renounceOwnership();
76: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

1562: 		    function setTroveStatus(address _borrower, address _collateral, uint _num) external override {
1563: 		        _requireCallerIsBorrowerOperations();
1564: 		        Troves[_borrower][_collateral].status = Status(_num);
1565: 		    }


File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

160: 		    function setHarvestSteps(address[2][] calldata _newSteps) external {
161: 		        _atLeastRole(ADMIN);
162: 		        delete steps;
163: 		
164: 		        uint256 numSteps = _newSteps.length;
165: 		        for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
166: 		            address[2] memory step = _newSteps[i];
167: 		            require(step[0] != address(0));
168: 		            require(step[1] != address(0));
169: 		            steps.push(step);
170: 		        }
171: 		    }


File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

627: 		    function updateTreasury(address newTreasury) external {
628: 		        _atLeastRole(DEFAULT_ADMIN_ROLE);
629: 		        require(newTreasury != address(0), "Invalid address");
630: 		        treasury = newTreasury;
631: 		    }


File: audits/2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

120: 		    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
121: 		        distributionPeriod = _newDistributionPeriod;
122: 		    }


File: audits/2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

156: 		    function setEmergencyExit() external {
157: 		        _atLeastRole(GUARDIAN);
158: 		        emergencyExit = true;
159: 		        IVault(vault).revokeStrategy(address(this));
160: 		    }

```
</details>

---
### [L-08] Owner can renounce ownership

The contract's owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

Openzeppelin's `Ownable` used in this project implements `renounceOwnership`. This can represent a certain risk if the ownership is renounced for any other reason than by design.

Renouncing ownership will leave the contract without an Owner, therefore removing any functionality that needs authority.

*There are 22 instances of this issue.*

<details>
<summary>Expand findings</summary>


```solidity
File: audits/2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

83: 		        onlyOwner

125: 		    function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

132: 		    function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138: 		    function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144: 		    function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

214: 		    function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {


File: audits/2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

125: 		        onlyOwner


File: audits/2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

50: 		    ) external override onlyOwner {

89: 		    ) external onlyOwner checkCollateral(_collateral) {


File: audits/2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol

49: 		        onlyOwner


File: audits/2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol

46: 		        onlyOwner


File: audits/2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol

33: 		        onlyOwner


File: audits/2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol

97: 		        onlyOwner

146: 		    ) external onlyOwner {

169: 		    ) external onlyOwner {


File: audits/2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol

64: 		    ) external onlyOwner {


File: audits/2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol

80: 		    function setParams(address _troveManagerAddress, address _borrowerOperationsAddress) external override onlyOwner {


File: audits/2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

273: 		        onlyOwner


File: audits/2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

67: 		        onlyOwner 

101: 		    function fund(uint amount) external onlyOwner {

120: 		    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {


File: audits/2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

76: 		        onlyOwner 

```
</details>

---
## Non Critical Findings

---

### [NC-01] Low test coverage


Improve the test coverage to 100% for all contracts to minimize the risk of bugs and vulnerabilities.

Ethos-Vault
```sh
--------------------------------------|----------|----------|----------|----------|----------------|
File                                  |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
--------------------------------------|----------|----------|----------|----------|----------------|
 contracts\                           |    83.26 |     62.5 |    84.13 |    81.55 |                |
  ReaperStrategyGranarySupplyOnly.sol |    90.38 |    66.67 |    86.67 |    89.23 |... 136,214,215 |
  ReaperVaultERC4626.sol              |    97.22 |     87.5 |    94.44 |    96.88 |             30 |
  ReaperVaultV2.sol                   |    77.48 |    54.46 |    76.67 |    76.89 |... 637,638,639 |
 contracts\abstract\                  |    81.58 |    36.36 |       90 |    82.14 |                |
  ReaperBaseStrategyv4.sol            |    81.58 |    36.36 |       90 |    82.14 |... 190,191,195 |
 contracts\interfaces\                |      100 |      100 |      100 |      100 |                |
  IAToken.sol                         |      100 |      100 |      100 |      100 |                |
  IAaveIncentivesController.sol       |      100 |      100 |      100 |      100 |                |
  IAaveProtocolDataProvider.sol       |      100 |      100 |      100 |      100 |                |
  IERC20Minimal.sol                   |      100 |      100 |      100 |      100 |                |
  IERC4626Events.sol                  |      100 |      100 |      100 |      100 |                |
  IERC4626Functions.sol               |      100 |      100 |      100 |      100 |                |
  IInitializableAToken.sol            |      100 |      100 |      100 |      100 |                |
  ILendingPool.sol                    |      100 |      100 |      100 |      100 |                |
  ILendingPoolAddressesProvider.sol   |      100 |      100 |      100 |      100 |                |
  IRewardsController.sol              |      100 |      100 |      100 |      100 |                |
  IRewardsDistributor.sol             |      100 |      100 |      100 |      100 |                |
  IScaledBalanceToken.sol             |      100 |      100 |      100 |      100 |                |
  IStrategy.sol                       |      100 |      100 |      100 |      100 |                |
  IVault.sol                          |      100 |      100 |      100 |      100 |                |
  IVeloPair.sol                       |      100 |      100 |      100 |      100 |                |
  IVeloRouter.sol                     |      100 |      100 |      100 |      100 |                |
 contracts\libraries\                 |    18.75 |        5 |    66.67 |    10.81 |                |
  Babylonian.sol                      |        0 |        0 |        0 |        0 |... 50,51,52,53 |
  DistributionTypes.sol               |      100 |      100 |      100 |      100 |                |
  ReaperMathUtils.sol                 |      100 |      100 |      100 |      100 |                |
  SafeERC20Minimal.sol                |      100 |       50 |      100 |      100 |                |
 contracts\mixins\                    |    63.33 |       50 |       50 |    66.67 |                |
  ReaperAccessControl.sol             |      100 |      100 |      100 |      100 |                |
  VeloSolidMixin.sol                  |       50 |       20 |       40 |    53.57 |... 78,88,89,90 |
--------------------------------------|----------|----------|----------|----------|----------------|
All files                             |    78.02 |    53.98 |    81.71 |    74.38 |                |
--------------------------------------|----------|----------|----------|----------|----------------|
```

Ethos-Core
```sh
-------------------------------|----------|----------|----------|----------|----------------|
File                           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
-------------------------------|----------|----------|----------|----------|----------------|
 contracts\                    |    98.22 |    90.96 |    98.33 |    98.31 |                |
  ActivePool.sol               |      100 |    86.96 |      100 |      100 |                |
  BorrowerOperations.sol       |    98.64 |    90.63 |    95.35 |    98.64 |    249,250,643 |
  CollSurplusPool.sol          |      100 |       90 |      100 |      100 |                |
  CollateralConfig.sol         |      100 |    77.27 |      100 |      100 |                |
  DefaultPool.sol              |      100 |    83.33 |      100 |      100 |                |
  GasPool.sol                  |      100 |      100 |      100 |      100 |                |
  HintHelpers.sol              |    96.43 |    83.33 |    83.33 |    96.55 |        168,200 |
  LUSDToken.sol                |    93.22 |    92.31 |      100 |    93.28 |... 163,311,337 |
  PriceFeed.sol                |    95.65 |    88.68 |    95.45 |    95.85 |... 211,524,526 |
  RedemptionHelper.sol         |      100 |    92.86 |      100 |      100 |                |
  SortedTroves.sol             |    99.06 |      100 |    95.24 |       99 |            285 |
  StabilityPool.sol            |    97.61 |    95.45 |      100 |    97.61 |... 572,573,574 |
  TroveManager.sol             |    99.58 |     92.5 |      100 |    99.78 |            496 |
 contracts\Dependencies\       |    46.59 |    39.19 |       62 |    44.94 |                |
  Address.sol                  |       45 |    21.43 |    44.44 |    45.45 |... 152,156,161 |
  AggregatorV3Interface.sol    |      100 |      100 |      100 |      100 |                |
  BaseMath.sol                 |      100 |      100 |      100 |      100 |                |
  CheckContract.sol            |      100 |      100 |      100 |      100 |                |
  IERC2362.sol                 |      100 |      100 |      100 |      100 |                |
  IERC4626.sol                 |      100 |      100 |      100 |      100 |                |
  IMappingContract.sol         |      100 |      100 |      100 |      100 |                |
  ITellor.sol                  |      100 |      100 |      100 |      100 |                |
  LiquityBase.sol              |      100 |      100 |      100 |      100 |                |
  LiquityMath.sol              |    97.06 |    85.71 |      100 |    96.77 |            124 |
  LiquitySafeMath128.sol       |      100 |      100 |      100 |      100 |                |
  SafeERC20.sol                |    63.64 |    33.33 |    66.67 |    63.64 |    42,45,54,55 |
  TellorCaller.sol             |      100 |      100 |      100 |      100 |                |
  UsingTellor.sol              |      2.5 |        0 |    14.29 |     2.41 |... 345,359,360 |
 contracts\LQTY\               |      100 |    96.67 |      100 |      100 |                |
  CommunityIssuance.sol        |      100 |      100 |      100 |      100 |                |
  LQTYStaking.sol              |      100 |       95 |      100 |      100 |                |
 contracts\Proxy\              |    86.96 |    59.09 |    70.97 |    86.84 |                |
  BorrowerOperationsScript.sol |    29.41 |        0 |    22.22 |    29.41 |... 50,51,53,57 |
  BorrowerWrappersScript.sol   |    98.73 |       65 |      100 |    98.72 |            178 |
  ERC20TransferScript.sol      |      100 |      100 |      100 |      100 |                |
  LQTYStakingScript.sol        |      100 |      100 |      100 |      100 |                |
  StabilityPoolScript.sol      |       75 |      100 |    66.67 |       75 |             24 |
  TokenScript.sol              |     87.5 |      100 |    85.71 |     87.5 |             24 |
  TroveManagerScript.sol       |      100 |      100 |      100 |      100 |                |
-------------------------------|----------|----------|----------|----------|----------------|
All files                      |    93.53 |    83.97 |    91.81 |    93.35 |                |
-------------------------------|----------|----------|----------|----------|----------------|
```




### [NC-02] Unused named `return` is confusing

Declaring named returns, but not using them, is confusing. Consider either completely removing them (by declaring just the type without a name), or remove the return statement and do a variable assignment.

This would improve the readability of the code, and it may also help reduce regressions during future code refactors.

*There are 26 instances of this issue.*

<details>
<summary>Expand findings</summary>


```solidity
File: audits/2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol

162: 		        returns (address hintAddress, uint diff, uint latestRandomSeed)
168: 		            return (address(0), 0, _inputRandomSeed);


File: audits/2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol

557: 		    function _getCurrentTellorResponse(address _collateral) internal view returns (TellorResponse memory tellorResponse) {
571: 		            return (tellorResponse);

557: 		    function _getCurrentTellorResponse(address _collateral) internal view returns (TellorResponse memory tellorResponse) {
574: 		            return (tellorResponse);


File: audits/2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

511: 		        returns (uint collGainPerUnitStaked, uint LUSDLossPerUnitStaked)
543: 		        return (collGainPerUnitStaked, LUSDLossPerUnitStaked);

626: 		    function getDepositorCollateralGain(address _depositor) public view override returns (address[] memory assets, uint[] memory amounts) {
632: 		            return _getCollateralGainFromSnapshots(initialDeposit, snapshots);


File: audits/2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

369: 		        returns (LiquidationValues memory singleLiquidation)
433: 		            return zeroVals;

1316: 		    function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index) {
1318: 		        return _addTroveOwnerToArray(_borrower, _collateral);


File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

104: 		    function _liquidateAllPositions() internal override returns (uint256 amountFreed) {
106: 		        return balanceOfWant();


File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol

29: 		    function asset() external view override returns (address assetTokenAddress) {
30: 		        return address(token);

37: 		    function totalAssets() external view override returns (uint256 totalManagedAssets) {
38: 		        return balance();

51: 		    function convertToShares(uint256 assets) public view override returns (uint256 shares) {
52: 		        if (totalSupply() == 0 || _freeFunds() == 0) return assets;

51: 		    function convertToShares(uint256 assets) public view override returns (uint256 shares) {
53: 		        return (assets * totalSupply()) / _freeFunds();

66: 		    function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
67: 		        if (totalSupply() == 0) return shares;

66: 		    function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
68: 		        return (shares * _freeFunds()) / totalSupply();

79: 		    function maxDeposit(address) external view override returns (uint256 maxAssets) {
80: 		        if (emergencyShutdown || balance() >= tvlCap) return 0;

79: 		    function maxDeposit(address) external view override returns (uint256 maxAssets) {
81: 		        if (tvlCap == type(uint256).max) return type(uint256).max;

79: 		    function maxDeposit(address) external view override returns (uint256 maxAssets) {
82: 		        return tvlCap - balance();

96: 		    function previewDeposit(uint256 assets) external view override returns (uint256 shares) {
97: 		        return convertToShares(assets);

122: 		    function maxMint(address) external view override returns (uint256 maxShares) {
123: 		        if (emergencyShutdown || balance() >= tvlCap) return 0;

122: 		    function maxMint(address) external view override returns (uint256 maxShares) {
124: 		        if (tvlCap == type(uint256).max) return type(uint256).max;

122: 		    function maxMint(address) external view override returns (uint256 maxShares) {
125: 		        return convertToShares(tvlCap - balance());

139: 		    function previewMint(uint256 shares) public view override returns (uint256 assets) {
140: 		        if (totalSupply() == 0) return shares;

165: 		    function maxWithdraw(address owner) external view override returns (uint256 maxAssets) {
166: 		        return convertToAssets(balanceOf(owner));

183: 		    function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {
184: 		        if (totalSupply() == 0 || _freeFunds() == 0) return 0;

220: 		    function maxRedeem(address owner) external view override returns (uint256 maxShares) {
221: 		        return balanceOf(owner);

240: 		    function previewRedeem(uint256 shares) external view override returns (uint256 assets) {
241: 		        return convertToAssets(shares);

```
</details>

---
### [NC-03] Same constant redefined elsewhere

Keeping the same constants in different files may cause some problems or errors, reading constants from a single file is preferable. This should also be preferred for gas optimizations.

*There are 8 instances of this issue.*


```solidity
File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

41: 		    uint256 public constant PERCENT_DIVISOR = 10000;

74: 		    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75: 		    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76: 		    bytes32 public constant ADMIN = keccak256("ADMIN");


File: audits/2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

23: 		    uint256 public constant PERCENT_DIVISOR = 10_000;

50: 		    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51: 		    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52: 		    bytes32 public constant ADMIN = keccak256("ADMIN");

```

### [NC-04] Use of `constant` variables instead of `immutable`

Constant variables and immutable variables serve different purposes and should be used accordingly.

To clarify, constant variables are used for literal values in the code, whereas immutable variables are ideal for values calculated or passed into the constructor.

*There are 8 instances of this issue.*


```solidity
File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

73: 		    bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74: 		    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75: 		    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76: 		    bytes32 public constant ADMIN = keccak256("ADMIN");


File: audits/2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

49: 		    bytes32 public constant KEEPER = keccak256("KEEPER");

50: 		    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51: 		    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52: 		    bytes32 public constant ADMIN = keccak256("ADMIN");

```

### [NC-05] Use `require` instead of `assert`

Prior to Solidity 0.8.0, the big difference between the `assert` and `require` is that the former uses up all the remaining gas and reverts all the changes made when the predicate is false.

It should be avoided even after Solidity version 0.8.0 as the documentation states:

> Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix.

*There are 21 instances of this issue.*


```solidity
File: audits/2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

128: 		        assert(MIN_NET_DEBT > 0);

197: 		        assert(vars.compositeDebt > 0);

301: 		        assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

331: 		        assert(_collWithdrawal <= vars.coll); 


File: audits/2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

312: 		        assert(sender != address(0));

313: 		        assert(recipient != address(0));

321: 		        assert(account != address(0));

329: 		        assert(account != address(0));

337: 		        assert(owner != address(0));

338: 		        assert(spender != address(0));


File: audits/2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol

128: 		        assert(lusdToken.balanceOf(_redeemer) <= totals.totalLUSDSupplyAtStart);


File: audits/2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

526: 		        assert(_debtToOffset <= _totalLUSDDeposits);

551: 		        assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);

591: 		        assert(newP > 0);


File: audits/2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

417: 		            assert(_LUSDInStabPool != 0);

1224: 		            assert(totalStakesSnapshot[_collateral] > 0);

1279: 		        assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

1342: 		        assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

1348: 		        assert(index <= idxLast);

1414: 		        assert(newBaseRate > 0); // Base rate is always non-zero after redemption

1489: 		        assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0

```

### [NC-06] `require` without any message

If a transaction reverts, it can be confusing to debug if there aren't any messages. Add a descriptive message to all require statements.

*There are 18 instances of this issue.*


```solidity
File: audits/2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol

179: 		        require(totals.totalCollateralDrawn > 0);

304: 		        require(_maxFeePercentage >= troveManager.REDEMPTION_FEE_FLOOR() && _maxFeePercentage <= DECIMAL_PRECISION);

309: 		        require(block.timestamp >= systemDeploymentTime.add(BOOTSTRAP_PERIOD));

313: 		        require(_getTCR(_collateral, _price, _collDecimals) >= _MCR);

317: 		        require(_amount > 0);

321: 		        require(_lusdToken.balanceOf(_redeemer) >= _amount);


File: audits/2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

250: 		        require(msg.sender == owner);

551: 		        require(totals.totalDebtInSequence > 0);

716: 		        require(_troveArray.length != 0);

754: 		        require(totals.totalDebtInSequence > 0);

1450: 		        require(redemptionFee < _collateralDrawn);

1523: 		        require(msg.sender == borrowerOperationsAddress);

1527: 		        require(msg.sender == address(redemptionHelper));

1531: 		        require(msg.sender == borrowerOperationsAddress || msg.sender == address(redemptionHelper));

1535: 		        require(Troves[_borrower][_collateral].status == Status.active);

1539: 		        require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);


File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

167: 		            require(step[0] != address(0));

168: 		            require(step[1] != address(0));

```

### [NC-07] Some functions don't follow the Solidity naming conventions

Follow the Solidity naming convention by adding an underscore before the function name, for any `private` and `internal` functions.

*There is 1 instance of this issue.*


```solidity
File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol

269: 		    function roundUpDiv(uint256 x, uint256 y) internal pure returns (uint256) {

```

### [NC-08] Old Solidity version

Use a more recent version of Solidity.

*There are 33 instances of this issue.*

<details>
<summary>Expand findings</summary>


```solidity
File: audits/2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/GasPool.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Migrations.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/IActivePool.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/IBorrowerOperations.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/ICollateralConfig.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/ICollSurplusPool.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/ICommunityIssuance.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/IDefaultPool.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/ILiquityBase.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/ILQTYStaking.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/ILUSDToken.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/IPool.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/IPriceFeed.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/IRedemptionHelper.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/ISortedTroves.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/IStabilityPool.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/ITellorCaller.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/Interfaces/ITroveManager.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

3: 		pragma solidity 0.6.11;


File: audits/2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

3: 		pragma solidity 0.6.11;

```
</details>

---
### [NC-09] Use of floating pragma

Locking the pragma helps avoid accidental deploys with an outdated compiler version that may introduce bugs and unexpected vulnerabilities.

Floating pragma is meant to be used for libraries and contracts that have external users and need backward compatibility.

*There are 18 instances of this issue.*

<details>
<summary>Expand findings</summary>


```solidity
File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IAaveIncentivesController.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IAaveProtocolDataProvider.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IAToken.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IERC20Minimal.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IERC4626Events.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IERC4626Functions.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IInitializableAToken.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/ILendingPool.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/ILendingPoolAddressesProvider.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IScaledBalanceToken.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IStrategy.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IVault.sol

3: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IVeloPair.sol

2: 		pragma solidity ^0.8.0;


File: audits/2023-02-ethos/Ethos-Vault/contracts/interfaces/IVeloRouter.sol

2: 		pragma solidity ^0.8.0;

```
</details>

---
### [NC-10] Missing or incomplete NatSpec

Some functions miss a NatSpec, or they have an incomplete one.
It is recommended that contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI).

Include either `@notice` and/or `@dev` to describe how the function is supposed to work, a `@param` to describe each parameter, and a `@return` for any return values.

*There are 62 instances of this issue.*

<details>
<summary>Expand findings</summary>


```solidity
File: audits/2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

69: 		    // --- Contract setters ---
70: 		
71: 		    function setAddresses(

125: 		    function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

132: 		    function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138: 		    function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144: 		    function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

169: 		    // --- Pool functionality ---
170: 		
171: 		    function sendCollateral(address _collateral, address _account, uint _amount) external override {

190: 		    function increaseLUSDDebt(address _collateral, uint _amount) external override {

197: 		    function decreaseLUSDDebt(address _collateral, uint _amount) external override {

204: 		    function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override {

214: 		    function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {


File: audits/2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

108: 		    // --- Dependency setters ---
109: 		
110: 		    function setAddresses(

170: 		    // --- Borrower Trove Operations ---
171: 		
172: 		    function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

241: 		    // Send more collateral to an existing trove
242: 		    function addColl(address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override {

247: 		    // Send collateral gain to a trove. Called by only the Stability Pool.
248: 		    function moveCollateralGainToTrove(address _borrower, address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override {

253: 		    // Withdraw collateral from a trove
254: 		    function withdrawColl(address _collateral, uint _collWithdrawal, address _upperHint, address _lowerHint) external override {

258: 		    // Withdraw LUSD tokens from a trove: mint new LUSD tokens to the owner, and increase the trove's debt accordingly
259: 		    function withdrawLUSD(address _collateral, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

263: 		    // Repay LUSD tokens to a Trove: Burn the repaid LUSD tokens, and reduce the trove's debt accordingly
264: 		    function repayLUSD(address _collateral, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

268: 		    function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override {

375: 		    function closeTrove(address _collateral) external override {

409: 		    /**
410: 		     * Claim remaining collateral from a redemption or from a liquidation with ICR > MCR in Recovery Mode
411: 		     */
412: 		    function claimCollateral(address _collateral) external override {


File: audits/2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

84: 		    // Doing this irresponsibly can permanently harm the protocol.
85: 		    function updateCollateralRatios(


File: audits/2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol

39: 		    // --- Contract setters ---
40: 		
41: 		    function setAddresses(

81: 		    // --- Pool functionality ---
82: 		
83: 		    function accountSurplus(address _account, address _collateral, uint _amount) external override {

93: 		    function claimColl(address _account, address _collateral) external override {

108: 		    function pullCollateralFromActivePool(address _collateral, uint _amount) external override {


File: audits/2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol

38: 		    // --- Dependency setters ---
39: 		
40: 		    function setAddresses(

80: 		    // --- Pool functionality ---
81: 		
82: 		    function sendCollateralToActivePool(address _collateral, uint _amount) external override {

94: 		    function increaseLUSDDebt(address _collateral, uint _amount) external override {

101: 		    function decreaseLUSDDebt(address _collateral, uint _amount) external override {

108: 		    function pullCollateralFromActivePool(address _collateral, uint _amount) external override {


File: audits/2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol

25: 		    // --- Dependency setters ---
26: 		
27: 		    function setAddresses(


File: audits/2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol

88: 		    // --- Dependency setters ---
89: 		    
90: 		    function setAddresses(

142: 		    // !!!PLEASE USE EXTREME CARE AND CAUTION!!!
143: 		    function updateChainlinkAggregator(

166: 		    // !!!PLEASE USE EXTREME CARE AND CAUTION!!!
167: 		    function updateTellorCaller(

176: 		    /*
177: 		    * fetchPrice():
178: 		    * Returns the latest price obtained from the Oracle. Called by Liquity functions that require a current price.
179: 		    *
180: 		    * Also callable by anyone externally.
181: 		    *
182: 		    * Non-view function - it stores the last good price seen by Liquity.
183: 		    *
184: 		    * Uses a main oracle (Chainlink) and a fallback oracle (Tellor) in case Chainlink fails. If both fail, 
185: 		    * it uses the last good price seen by Liquity.
186: 		    *
187: 		    */
188: 		    function fetchPrice(address _collateral) external override returns (uint) {


File: audits/2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol

54: 		    function setAddresses(

78: 		    /* Send _LUSDamount LUSD to the system and redeem the corresponding amount of collateral from as many Troves as are needed to fill the redemption
79: 		    * request.  Applies pending rewards to a Trove before reducing its debt and coll.
80: 		    *
81: 		    * Note that if _amount is very large, this function can run out of gas, specially if traversed troves are small. This can be easily avoided by
82: 		    * splitting the total _amount in appropriate chunks and calling the function multiple times.
83: 		    *
84: 		    * Param `_maxIterations` can also be provided, so the loop through Troves is capped (if it’s zero, it will be ignored).This makes it easier to
85: 		    * avoid OOG for the frontend, as only knowing approximately the average cost of an iteration is enough, without needing to know the “topology”
86: 		    * of the trove list. It also avoids the need to set the cap in stone in the contract, nor doing gas calculations, as both gas price and opcode
87: 		    * costs can vary.
88: 		    *
89: 		    * All Troves that are redeemed from -- with the likely exception of the last one -- will end up with no debt left, therefore they will be closed.
90: 		    * If the last Trove does have some remaining debt, it has a finite ICR, and the reinsertion could be anywhere in the list, therefore it requires a hint.
91: 		    * A frontend should use getRedemptionHints() to calculate what the ICR of this Trove will be after redemption, and pass a hint for its position
92: 		    * in the sortedTroves list along with the ICR value that the hint was found for.
93: 		    *
94: 		    * If another transaction modifies the list between calling getRedemptionHints() and passing the hints to redeemCollateral(), it
95: 		    * is very likely that the last (partially) redeemed Trove would end up with a different ICR than what the hint is for. In this case the
96: 		    * redemption will stop after the last completely redeemed Trove and the sender will keep the remaining LUSD amount, which they can attempt
97: 		    * to redeem later.
98: 		    */
99: 		    function redeemCollateral(


File: audits/2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol

78: 		    // --- Dependency setters ---
79: 		
80: 		    function setParams(address _troveManagerAddress, address _borrowerOperationsAddress) external override onlyOwner {

154: 		    function remove(address _collateral, address _id) external override {


File: audits/2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

259: 		    // --- Contract setters ---
260: 		
261: 		    function setAddresses(

317: 		    /*  provideToSP():
318: 		    *
319: 		    * - Triggers a LQTY issuance, based on time passed since the last issuance. The LQTY issuance is shared between *all* depositors
320: 		    * - Sends depositor's accumulated gains to depositor
321: 		    * - Increases depositor's deposit, and takes new snapshot.
322: 		    */
323: 		    function provideToSP(uint _amount) external override {

359: 		    /*  withdrawFromSP():
360: 		    *
361: 		    * - Triggers a LQTY issuance, based on time passed since the last issuance. The LQTY issuance is shared between *all* depositors
362: 		    * - Sends all depositor's accumulated gains to depositor
363: 		    * - Decreases depositor's deposit, and takes new snapshot.
364: 		    *
365: 		    * If _amount > userDeposit, the user withdraws all of their compounded deposit.
366: 		    */
367: 		    function withdrawFromSP(uint _amount) external override {

461: 		    /*
462: 		    * Cancels out the specified debt against the LUSD contained in the Stability Pool (as far as possible)
463: 		    * and transfers the Trove's collateral from ActivePool to StabilityPool.
464: 		    * Only called by liquidation functions in the TroveManager.
465: 		    */
466: 		    function offset(address _collateral, uint _debtToOffset, uint _collToAdd) external override {

481: 		    /*
482: 		    * Updates the reward sum for the specified collateral. A trimmed down version of "offset()" that doesn't
483: 		    * concern itself with any debt to offset or LUSD loss. Only called by ActivePool when distributing
484: 		    * yield farming rewards.
485: 		    */
486: 		    function updateRewardSum(address _collateral, uint _collToAdd) external override {


File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

58: 		    /**
59: 		     * @dev Initializes the strategy. Sets parameters, saves routes, and gives allowances.
60: 		     * @notice see documentation for each variable above its respective declaration.
61: 		     */
62: 		    function initialize(

144: 		    /**
145: 		     * Only {STRATEGIST} or higher roles may update the swap path for a token.
146: 		     */
147: 		    function updateVeloSwapPath(

156: 		    /**
157: 		     * Only {ADMIN} or higher roles may set the array
158: 		     * of steps executed as part of harvest.
159: 		     */
160: 		    function setHarvestSteps(address[2][] calldata _newSteps) external {

210: 		    /**
211: 		     * @dev Attempts to safely withdraw {_amount} from the pool.
212: 		     */
213: 		    function authorizedWithdrawUnderlying(uint256 _amount) external {


File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol

109: 		    // Note that most implementations will require pre-approval of the Vault with the Vault’s underlying asset token.
110: 		    function deposit(uint256 assets, address receiver) external override returns (uint256 shares) {

153: 		    // Note that most implementations will require pre-approval of the Vault with the Vault’s underlying asset token.
154: 		    function mint(uint256 shares, address receiver) external override returns (uint256 assets) {

201: 		    // Those methods should be performed separately.
202: 		    function withdraw(

257: 		    // Those methods should be performed separately.
258: 		    function redeem(


File: audits/2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

486: 		    /**
487: 		     * @notice Main contact point where each strategy interacts with the vault during its harvest
488: 		     * to report profit/loss as well as any repayment of debt.
489: 		     * @param _roi The return on investment (positive or negative) given as the total amount
490: 		     * gained or lost from the harvest.
491: 		     * @param _repayment The repayment of debt by the strategy.
492: 		     */
493: 		    function report(int256 _roi, uint256 _repayment) external returns (uint256) {

591: 		    /**
592: 		     * Activates or deactivates Vault mode where all Strategies go into full
593: 		     * withdrawal.
594: 		     * During Emergency Shutdown:
595: 		     * 1. No Users may deposit into the Vault (but may withdraw as usual.)
596: 		     * 2. New Strategies may not be added.
597: 		     * 3. Each Strategy must pay back their debt as quickly as reasonable to
598: 		     * minimally affect their position.
599: 		     *
600: 		     * If true, the Vault goes into Emergency Shutdown. If false, the Vault
601: 		     * goes back into Normal Operation.
602: 		     */
603: 		    function setEmergencyShutdown(bool _active) external {

624: 		    /**
625: 		     * @notice Only DEFAULT_ADMIN_ROLE can update treasury address.
626: 		     */
627: 		    function updateTreasury(address newTreasury) external {


File: audits/2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

64: 		    // --- Functions ---
65: 		
66: 		    function setAddresses

103: 		    // If caller has a pre-existing stake, send any accumulated collateral and LUSD gains to them. 
104: 		    function stake(uint _LQTYamount) external override {

141: 		    // If requested amount > stake, send their entire stake.
142: 		    function unstake(uint _LQTYamount) external override {

175: 		    // --- Reward-per-unit-staked increase functions. Called by Liquity core contracts ---
176: 		
177: 		    function increaseF_Collateral(address _collateral, uint _collFee) external override {

187: 		    function increaseF_LUSD(uint _LUSDFee) external override {


File: audits/2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

90: 		    /**
91: 		     * @dev Withdraws funds and sends them back to the vault. Can only
92: 		     *      be called by the vault. _amount must be valid and security fee
93: 		     *      is deducted up-front.
94: 		     */
95: 		    function withdraw(uint256 _amount) external override returns (uint256 loss) {

105: 		    /**
106: 		     * @dev harvest() function that takes care of logging. Subcontracts should
107: 		     *      override _harvestCore() and implement their specific logic in it.
108: 		     */
109: 		    function harvest() external override returns (int256 roi) {

```
</details>

---