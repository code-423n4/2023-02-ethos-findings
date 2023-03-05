[G-01] Gas griefing is possible on array iteration

Array length: The function takes three arrays as input parameters. If these arrays are too large, the gas cost of executing the function could become prohibitively high. Therefore, it is important to ensure that the size of these arrays is reasonable and does not exceed the block gas limit.

External function calls: The function calls an external function (checkContract) within a loop. If the loop is executed many times, the gas cost of these external function calls could also become prohibitively high.

Storage updates: The function updates several storage variables within a loop. Writing to storage is a relatively expensive operation in terms of gas cost. If the loop is executed many times, the gas cost of these storage updates could also become prohibitively high.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol

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

To mitigate these issues, it is important to carefully test and optimize the function to minimize gas costs. This may involve limiting the size of the input arrays, minimizing external function calls within loops, and optimizing storage updates. Additionally, it may be necessary to consider using off-chain or batch processing solutions to reduce the gas cost of executing the function.


[gas-02] Cache collateralConfig[_collateral] in the checkCollateral modifier

The checkCollateral modifier is called in several functions, and it accesses collateralConfig[_collateral] every time. Since the modifier already checks that _collateral is allowed, it would save gas to cache collateralConfig[_collateral] in a local variable in the modifier and pass it to the function as an argument.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol

 modifier checkCollateral(address _collateral) {
        require(collateralConfig[_collateral].allowed, "Invalid collateral address");
        _;
    }