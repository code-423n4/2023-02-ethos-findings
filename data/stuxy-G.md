Hello,
```initialize``` function of the CollateralConfig contract can be optimized to save gas by taking value of ```_collaterals.length``` into a local variable instead of using it repeatedly as shown below:
```
    function initialize(
        address[] calldata _collaterals,
        uint256[] calldata _MCRs,
        uint256[] calldata _CCRs
    ) external override onlyOwner {
        uint clen = _collaterals.length;
        require(!initialized, "Can only initialize once");
        require(clen != 0, "At least one collateral required");
        require(_MCRs.length == clen, "Array lengths must match");
        require(_CCRs.length == clen, "Array lenghts must match");
        
        for(uint256 i = 0; i < clen; i++) {
            address collateral = _collaterals[i];
            checkContract(collateral);
            collaterals.push(collateral);

            Config storage config = collateralConfig[collateral];
            config.allowed = true;
            uint256 decimals = IERC20(collateral).decimals();
            config.decimals = decimals;

            require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
            config.MCR = _MCRs[i];

            require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
            config.CCR = _CCRs[i];

            emit CollateralWhitelisted(collateral, decimals, _MCRs[i], _CCRs[i]);
        }

        initialized = true;
    }
```

This way ```_collaterals.length``` would only be calculated once.