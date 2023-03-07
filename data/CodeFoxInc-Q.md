# Low Risk and Non-Critical Issues

## ****Codebase Impressions & Summary****

The core contracts extended the functionality of the collaterals categories, with not only Ether can be added as collateral. And the protocol added a Vault function besides it to make the collateral assets’ efficiency better so that the stable coin’s backing assets generate interest as well. 

A single Liquity code base can already be a good example of project to get audited. The Ethos protocol was built upon it and added several customized functions. These functionalities combined with each other makes the protocol more complex. 

This audit contest partially does not include the Liquity protocol code base but a good understanding of it is needed. And Ethos Reserve expanded the functionality over it. As a result, the code base is quite large. 

The overall quality of the code is good, inline comments provided are helpful in describing the function of the code. But in terms of documentations, there are improvements can be made, e.g.,  natspec documentation style is recommended in my opinion. Tests ran without issues and can be modified to run POCs easily. 

The findings are made mainly focused on the best practice and code quality, documentations, etc. At the same time, there are some low-risk issues about the upgradeability and function arguments’ validations. 

## L-01 `_treasuryAddress` is not `checkContract`ed

In the ActivePool contract, the `_treasuryAddress` is not `checkContract`ed. I understand it can be an EOA, but as a big protocol which is going to generate millions of revenue and fees. I suggest put the `checkContract` function and make sure it is a multi-sig wallet. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L93)

```diff
+       checkContract(_treasuryAddress);
-       　require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
```

## L-02 Consider mitigating the centralization risk

The code base includes quite a lot of `renounceOwnership()` function to make sure the protocol is decentralized enough to be permission-less and trusted. 

At the same time, `priceFeed` contract is quite an important feature and it should not be controlled by a single entity. Price feed can be updated by the owner address in this the current situation. 

Team should consider set it as `guardianAddress` or `governanceAddress` instead like what is done in the `LUSDToken` contract. Or you can make the transition into a community governable feature. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/PriceFeed.sol#L143-L146](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/PriceFeed.sol#L143-L146)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/PriceFeed.sol#L24](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/PriceFeed.sol#L24)

And same thing can also happen in `CommunityIssuance` contract. 

## L-03 Should check the `token` address and make sure it is a contract address

Should check the address to make sure it is a contract address or zero address. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L120](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L120)

### Recommendation

Should import the `checkContract` function and use it as a checker for `_token`. 

```diff
+   import "../Dependencies/CheckContract.sol";

......

-   　contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessControlEnumerable, ReentrancyGuard {
+   contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessControlEnumerable, ReentrancyGuard, checkContract {

......

+	checkContract(_token);
    　　　　token = IERC20Metadata(_token);
```

## L-04 The `tvlCap` should be initialized as a reasonable value

The `tvlCap` value is set to zero when the input of it is zero. This makes the Vault not usable at the beginning. When the `tvlCap` is zero usually it means there is no limit, so it is reasonable to set the value as unlimited value(`type(uint256).max`). If the dev wants to set the vault as unusable in the beginning it also can be done by setting the value extremely low, e.g. 1 wei. Anyways, if dev deploys the contract it is supposed to work in the first place. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L123](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L123)

### Recommendation

Make a change as below: 

```diff
	token = IERC20Metadata(_token);
        constructionTime = block.timestamp;
        lastReport = block.timestamp;
+	    if (_tvlCap == 0) {
+		    tvlCap = type(uint256).max;  // @audit the tvlCap should be set to a reasonable value. If the tvlCap is 0, the tvlCap will be set to the maximum value of uint256.
+		} else {         
			tvlCap = _tvlCap;
+		}
        treasury = _treasury;
        lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```

## L-05 Consider adding a length boundary for the `_collaterals` and `_strategies` array

During the initialization of the CollateralConfig contract, consider adding a length boundary for `_collaterals` array. Too many collaterals can make the loop cost too much gas and it may fail to fulfill its functions. The upper limit of the collateral’s length can be set as a number which is usually not likely to reach. Also should take the block’s gas limit into account. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L47](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L47)

Same thing can be mentioned about `ReaperVaultV2` contract as below.  

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L127-L130](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L127-L130)

Also it is the same about the `_withdrawalQueue` in contract `ReaperVaultV2`.

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L258-L271](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L258-L271)

If the queue is too long, it can block normal users’ withdraw in `_withdraw` function. 

### Recommendation

Recommendation examples: (also use cache to make it more gas efficient)

```diff
function initialize(
        address[] calldata _collaterals,
        uint256[] calldata _MCRs,
        uint256[] calldata _CCRs
    ) external override onlyOwner {
	　require(!initialized, "Can only initialize once");
+       uint256 length = _collaterals.length;
-	　require(_collaterals.length != 0, "At least one collateral required");
+	　require(length != 0, "At least one collateral required");
+       require(length <= 50, "too many kinds of collaterals");
```

```diff
	　uint256 numStrategists = _strategists.length;
+       require(numStrategiests <= 10, "too many strategists")
        　for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
         　   _grantRole(STRATEGIST, _strategists[i]);
        　}
```

## L-06 Add `__gap` into the contract

When the contract is upgradeable, it is best practice to add a `__gap` into the last part of the contract for future upgrade. 

[As this mentioned](https://docs.openzeppelin.com/contracts/3.x/upgradeable#storage_gaps), `gap` is an empty reserved space in the storage for contract’s future upgrading. 

It can be tricky if `gap` is not in the inherited contract and you want to add more storages into the contract in the future upgrading version. Special cares need to be taken then. 

So it is recommended that developers implement this and follow the best practice. 

### Code snippet

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L274-L277](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L274-L277)

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/mixins/VeloSolidMixin.sol#L106-L111](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/mixins/VeloSolidMixin.sol#L106-L111)

### Recommedation

```diff
*      caller must be returned.
     */
    function _harvestCore(uint256 _debt) internal virtual returns (int256 roi, uint256 repayment);
		
// @audit gap shbould be added here
+	/**
+    * @dev This empty reserved space is put in place to allow future versions to add new
+    * variables without shifting down storage in the inheritance chain.
+    * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
+    */
+   　uint256[24] private __gap; 
    // [24] should equal to [50 -  (the number of how many used slots exist)]
}
```

```diff
function updateVeloSwapPath(
        address _tokenIn,
        address _tokenOut,
        address[] calldata _path
    ) external virtual;

// @audit gap shbould be added here
+	/**
+    * @dev This empty reserved space is put in place to allow future versions to add new
+    * variables without shifting down storage in the inheritance chain.
+    * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
+    */
+   　uint256[49] private __gap; 
    // [49] should equal to [50 - (the number of how many used slots exist)]
}
```

## L-07 Implementation contract not being initialized can become a problem

If the implementation's `initialize` function is not called, there is such risk that any attacker can have a chance to `selfdestruct` the implementation contract to make the whole system break in the worst case scenario. 

In the contract `ReaperStrategyGranarySupplyOnly`, the `initialize` function can be left uncalled. And even though developers have a plan to call it, it has a chance of being front-run. 

I recommend follow the practice of OpenZeppelin’s documentation and call this function in the constructor to make sure no one can play with it. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L72](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L72)

### Recommended Mitigation

Add this into the code base: 

```diff
+    /// @custom:oz-upgrades-unsafe-allow constructor
+    constructor() {
+        _disableInitializers();
+    }
```

## NC-01 Separate the number with 3 digits

This is a code style thing. I recommend follow the change as below. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L52-L54](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L52-L54)

```diff
// @audit the `_` is supposed to be used before 3 digits numbers, e.g. 2_000, but it is used with 2 digits numbers.
-    uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
-    uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
-    uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS
+    uint256 public yieldSplitTreasury = 2_000; // amount of yield to direct to treasury in BPS
+    uint256 public yieldSplitSP = 4_000; // amount of yield to direct to stability pool in BPS
+    uint256 public yieldSplitStaking = 4_000; // amount of yield to direct to OATH Stakers in BPS
```

## NC-02. Error message should include the contract name

The best practice of solidity suggest error message should include contract’s name. This rule was adopted in some contract but it is not followed in some places. And I suggest fix them. 

This is the good example showed in the `BorrowerOperations` contract by the sponsor.

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L534](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L534)

But here the error message is inconsistent with the other part in the same contract. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L538](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L538)

And here the error message has no contract name at all. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L651](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L651)

### Recommendation

Developers should follow the best practice and add the contract name into the error message to make improvement in terms of readability throughout the code base. I will give several examples in the code base here. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L651](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L651)

```diff
　　　　　　　if (_isRecoveryMode) {
            require(_maxFeePercentage <= DECIMAL_PRECISION,
-               "Max fee percentage must less than or equal to 100%");
+               "BorrowerOperations: Max fee percentage must less than or equal to 100%");
        } else {
            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
-                "Max fee percentage must be between 0.5% and 100%");
+                "BorrowerOperations: Max fee percentage must be between 0.5% and 100%");
        }
```

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L636-L638](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L636-L638)

```diff
function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure {
-        require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");
+        require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOperations: Amount repaid must not be larger than the Trove's debt"); 
   }
```

The same rule is also adoptable for other parts of the contracts in the Ethos-Core code base. You should follow this best practice through all of the code base. 

e.g., [https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L85](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L85)

```diff
-   　 require(!addressesSet, "Can call setAddresses only once");
+    require(!addressesSet, "ActivePool: Can call setAddresses only once");
```

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L238](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L238)

```diff
-    　_approve(sender, msg.sender, _allowances[sender][msg.sender].sub(amount, "ERC20: transfer amount exceeds allowance"));
+    _approve(sender, msg.sender, _allowances[sender][msg.sender].sub(amount, "LUSDToken: transfer amount exceeds allowance"));
```

## NC-03 Natspec documentation is needed for improving readability

Please consider adding more [**natspec documentation**](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html) in the code base to improve the readability for developers. 

Some very good examples of **natspec documentation** have been made by the developer. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L138-L148](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L138-L148)

## NC-04 Unnecessary `import` and wrong error messages

It is not necessary to import these in the production phase. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L15](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L15)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/Dependencies/LiquityMath.sol#L6](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/Dependencies/LiquityMath.sol#L6)

Another minor fix to the wrong error message: 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L651](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L651)

```diff
	require(_maxFeePercentage <= DECIMAL_PRECISION,
-                "Max fee percentage must less than or equal to 100%"); 
+                "Max fee percentage must be less than or equal to 100%"); 
```

## NC-06 Unnecessary library used

This library is not necessary in the contract. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L16](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L16)

## NC-07 Should multiply before doing the division calculation

Always do the multiply first and then division. Follow the best practice. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L54-L56](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L54-L56)

```diff
-　　　　uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%  
-　　　　uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5% 
+   uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION * 5 / 1000; // 0.5%  // @audit-ok should multiply first then divide to follow the best practice.
+    uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION * 5 / 100; // 5% // @audit-ok should multiply first then divide to follow the best practice.
```

## NC-08 Follow other contracts’ pattern of using `Ownable` contract

In `TroveManager` contract, the ownership pattern is like this: 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L225-L228](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L225-L228)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L294](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L294)

But in other part of the contracts, it is like this: 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L13](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L13)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L18](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L18)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L110-L168](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L110-L168)

It is not consistent with the ownership pattern in the code base. 

### Recommendation

I suggest use the ownership pattern of inheriting the `Ownable` contract and call the `renounceOwnership` function. Do not use two different patterns in the code base. 

## NC-09 Use `uint256` not `uint`

The `LQTYStaking` contract is using `uint` not `uint256.` and it is bad for not following the best practice. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L20](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L20)

The `LiquityMath` contract should use `uint256` instead of `uint` for following the best practice. (I understand this may be out of scope but just in case)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/Dependencies/LiquityMath.sol#L9](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/Dependencies/LiquityMath.sol#L9)

```diff
-    using SafeMath for uint; 
+   using SafeMath for uint256; // @audit use uint256 to follow best practice.
```

## NC-10 Do not use a floating solidity version

Throughout the Vault contracts’ code base the solidity version is as below: 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3)

It is recommended use a static version of solidity instead of a floating version in production phase. 

As recommended in the [slither document](https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-versions-of-solidity), `0.8.16` is a good option. 

```diff
-     pragma solidity ^0.8.0;
+    pragma soliidty 0.8.16;
```

## NC-11 Function name does not make sense

The function name is `balanceOf()` , but the actual thing it does is to calculate the total want held by the strategy. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L222](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L222)

So a better name like `totalWant()` , or `totalBalance()` is preferred generally. 

## NC-12 Should separate the number digits as 3 digits

You should separate the number digits as 3 digits. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L41](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L41)

```diff
-     uint256 public constant PERCENT_DIVISOR = 10000;
+    uint256 public constant PERCENT_DIVISOR = 10_000;
```

## NC-13 Should emit events when storage is changed

It should emits events when contract’s storage is changed. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136)

### Recommendation

```diff
    event StrategyReported(
         address indexed strategy,
         uint256 gain,
         uint256 loss,
         uint256 debtPaid,
         uint256 gains,
         uint256 losses,
         uint256 allocated,
         uint256 allocationAdded,
         uint256 allocBPS
    );
+   event TokenIsSet(address tokenAddress);
+   event ConstructionTime(uint256 constructionTime);
+   event LastReportTime(uint256 lastReportTime);
+   event TreasuryAdded(address index treasuryAddress);

		
		......
			
		/**
     * @notice Initializes the vault's own 'RF' token.
     * This token is minted when someone does a deposit. It is burned in order
     * to withdraw the corresponding portion of the underlying assets.
     * @param _token the token to maximize.
     * @param _name the name of the vault token.
     * @param _symbol the symbol of the vault token.
     * @param _tvlCap initial deposit cap for scaling TVL safely
     */
    constructor(
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
        treasury = _treasury;
        lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks

        uint256 numStrategists = _strategists.length;
        for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
            _grantRole(STRATEGIST, _strategists[i]);
        }

        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(DEFAULT_ADMIN_ROLE, _multisigRoles[0]);
        _grantRole(ADMIN, _multisigRoles[1]);
        _grantRole(GUARDIAN, _multisigRoles[2]);
+      emit TokenIsSet(_token);
+      emit ConstructionTime(block.timestamp);
+      emit LastReportTime(block.timestamp);
+      emit TvlCapUpdated(_tvlCap);
+      emit TreasuryAdded(_treasury);
+      emit LockedProfitDegradationUpdated(lockedProfitDegradation);
    }
```

## NC-14 Wrong error mesages

The error messages below should be "Fee cannot be higher than 20 percent” or "Fee cannot be higher than 2000 BPS”, not "Fee cannot be higher than 20 BPS”. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L181](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L181)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L155](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L155)