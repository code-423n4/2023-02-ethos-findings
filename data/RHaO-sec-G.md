[G-1] Using private rather than public for constants.

[CollateralConfig.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L21)
[CollateralConfig.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L25)  


[TroveManager.sol#L48-L61](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L48-L61)

[ActivePool.sol#L30](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30)


[StabilityPool.sol#L150](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150)
[StabilityPool.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L197)


[CommunityIssuance.sol#L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19)

[LQTYStaking.sol#L23](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23)

[ReaperVaultV2.sol#L40-L41](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40-L41)
[ReaperVaultV2.sol#L73-L76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73-L76)

[ReaperBaseStrategyv4.sol#L23-L25](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L23-L25)
[ReaperBaseStrategyv4.sol#L49-L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49-L52)

[SupplyOnly.sol#L24-L29](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L24-L29)


[G-2] Packing structs [ref](https://dev.to/javier123454321/solidity-gas-optimizations-pt-3-packing-structs-23f4)

[BorrowerOperations.sol#L54](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L54)

[StabilityPool.sol#L646-L655](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L646-L655)

[ReaperVaultV2.sol#L473-L484](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L473-L484)

[G-3] Use a more recent version of Solidity

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

Use a Solidity version of at least 0.8.12 to get string.concat() to be used instead of abi.encodePacked(<str>,<str>).

Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions.

[G-4] Should use arguments instead of state variable

[ReaperVaultV2.sol#L581](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L581)

[G-5] Use assembly to check for address(0)
[ActivePool.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L93)

[LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312-L313)
[LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L321)
[LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347-L351)
[LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L337-L338)
[LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L329)

[ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L151)
[ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L629)

[SupplyOnly.sol#L167-L168](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167-L168)

[G-6] initialize() :_MCRs AND _CCRs SHOULD BE CACHED

function initialize(
        address[] calldata _collaterals,
        uint256[] calldata _MCRs,
        uint256[] calldata _CCRs
    ) external override onlyOwner {
        require(!initialized, "Can only initialize once");
        require(_collaterals.length != 0, "At least one collateral required");
        require(_MCRs.length == _collaterals.length, "Array lengths must match"); //@audit gas: should cache "_MCRs" (SLOAD 1)
        require(_CCRs.length == _collaterals.length, "Array lenghts must match"); //@audit gas: should cache "_CCRs" (SLOAD 1)
        
        for(uint256 i = 0; i < _collaterals.length; i++) {
            address collateral = _collaterals[i];
            checkContract(collateral);
            collaterals.push(collateral);

            Config storage config = collateralConfig[collateral];
            config.allowed = true;
            uint256 decimals = IERC20(collateral).decimals();
            config.decimals = decimals;

            require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum"); //@audit gas: should cache "_MCRs" (SLOAD 2)
            config.MCR = _MCRs[i];

            require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum"); //@audit gas: should cache "_CCRs" (SLOAD 2)
            config.CCR = _CCRs[i];

            emit CollateralWhitelisted(collateral, decimals, _MCRs[i], _CCRs[i]);
        }

        initialized = true;
    }

[G-7] updateCollateralRatios() : _MCR & _CCR SHOULD ME CACHED

    function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
    ) external onlyOwner checkCollateral(_collateral) {
        Config storage config = collateralConfig[_collateral];
        require(_MCR <= config.MCR, "Can only walk down the MCR");  //@audit gas: should cache "_MCR" (SLOAD 1)
        require(_CCR <= config.CCR, "Can only walk down the CCR"); //@audit gas: should cache "_CCR" (SLOAD 1)

        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum"); //@audit gas: should cache "_MCR" (SLOAD 2)
        config.MCR = _MCR;

        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum"); //@audit gas: should cache "_CCR" (SLOAD 2)
        config.CCR = _CCR;
        emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
    }

[G-8]  _requireSufficientCollateralBalanceAndAllowance() : IERC20(_collateral) SHOLUD BE CACHED

function _requireSufficientCollateralBalanceAndAllowance(address _user, address _collateral, uint _collAmount) internal view {
        require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");  //@audit gas: should cache "IERC20(_collateral)" (SLOAD 1)
        require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");  //@audit gas: should cache "IERC20(_collateral)" (SLOAD 2)
    }

[G-9] withdraw() : _amount SHOULD BE CACHED

 function (uint256 _amount) external override returns (uint256 loss) {
        require(msg.sender == vault, "Only vault can withdraw"); 
        require(_amount != 0, "Amount cannot be zero");  //@audit gas: should cache "l_amount" (SLOAD 1)
        require(_amount <= balanceOf(), "Ammount must be less than balance");  //@audit gas: should cache "_amount" (SLOAD 2)



