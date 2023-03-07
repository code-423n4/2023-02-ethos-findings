CollateralConfig.sol – duplicates can be initialized

Level: Low. Issue only materializes if there is human error on initialization. 

The initialize() function uses arrays to keep track of whitelisted collateral and does not check for duplicates.

Double-counting of collateral value: If the same collateral is added to the whitelist twice, it could result in double-counting of its value. This could lead to an overestimation of the system's total collateral value, which could result in incorrect calculations of the MCR and CCR

Inconsistent collateral configuration: If the same collateral is added to the whitelist with different MCR and CCR values, it could result in inconsistent collateral configuration. This could lead to unexpected behavior in the system, such as triggering recovery mode at the wrong time or not triggering it when it should be triggered.


proposed solution:
mapping(address => bool) memory addedCollaterals; <-- add a mapping
function initialize(
        address[] calldata _collaterals,
        uint256[] calldata _MCRs,
        uint256[] calldata _CCRs
    ) external override onlyOwner {
        require(!initialized, "Can only initialize once");
        require(_collaterals.length != 0, "At least one collateral required");
        require(_MCRs.length == _collaterals.length, "Array lengths must match");
        require(_CCRs.length == _collaterals.length, "Array lenghts must match");
        
        for(uint256 i = 0; i < _collaterals.length; i++) {
             address collateral = _collaterals[i];
             require(!addedCollaterals[collateral], "Duplicate collateral"); <--check for duplicate
             addedCollaterals[collateral] = true; <--add to mapping

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

