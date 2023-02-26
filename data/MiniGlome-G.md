## Gas Optimizations
| |Issue|Instances|
|-|:-|:-:|
| [GAS-01] | Use custom errors rather than `revert()`/`require()` strings | 96 | 
| [GAS-02] | Don't Initialize Variables with Default Value | 27 | 
| [GAS-03] | Long Revert String | 36 | 
| [GAS-04] | Use Shift Right/Left instead of Division/Multiplication if possible | 2 | 
| [GAS-05] | `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow | 17 | 
| [GAS-06] | Splitting `require()` statements that use `&&` saves gas | 2 | 
| [GAS-07] | Superfluous event fields | 1 | 
| [GAS-08] | Setting the `constructor` to `payable` | 6 | 
| [GAS-09] | Usage of uint/int smaller than 32 bytes | 5 | 
| [GAS-10] | Use `<`/`>` instead of `>=`/`>=` | 3 | 
| [GAS-11] | Using fixed bytes is cheaper than using `string` | 18 | 
| [GAS-12] | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 9 | 
### [GAS-01] Use custom errors rather than `revert()`/`require()` strings
Custom errors are available from solidity version 0.8.4. Custom errors save [~50 gas](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas.

*Instances (96)*:
```solidity
File: Ethos\ActivePool.sol
85:        require(!addressesSet, "Can call setAddresses only once");

93:        require(_treasuryAddress != address(0), "Treasury cannot be 0 address");

107:        require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");

111:            require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");

127:        require(_bps <= 10_000, "Invalid BPS value");

133:        require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

145:        require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");

314:        require(
            ICollateralConfig(collateralConfigAddress).isCollateralAllowed(_collateral),
            "Invalid collateral address"
        );

321:        require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == defaultPoolAddress,
            "ActivePool: Caller is neither BO nor Default Pool");

329:        require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == stabilityPoolAddress,
            "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool");

338:        require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == troveManagerAddress,
            "ActivePool: Caller is neither BorrowerOperations nor TroveManager");

```

```solidity
File: Ethos\BorrowerOperations.sol
525:        require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");

529:        require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");

530:        require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");

534:        require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");

538:        require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");

543:        require(status == 1, "BorrowerOps: Trove does not exist or is closed");

548:        require(status != 1, "BorrowerOps: Trove is active");

552:        require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");

561:        require(
            !_checkRecoveryMode(_collateral, _price, _CCR, _collateralDecimals),
            "BorrowerOps: Operation not permitted during Recovery Mode"
        );

568:        require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");

617:        require(_newICR >= _MCR, "BorrowerOps: An operation that would result in ICR < MCR is not permitted");

621:        require(_newICR >= _CCR, "BorrowerOps: Operation must leave trove with ICR >= CCR");

625:        require(_newICR >= _oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");

629:        require(_newTCR >= _CCR, "BorrowerOps: An operation that would result in TCR < CCR is not permitted");

633:        require (_netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");

637:        require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");

641:        require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");

645:        require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");

650:            require(_maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must less than or equal to 100%");

653:            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must be between 0.5% and 100%");

```

```solidity
File: Ethos\CollateralConfig.sol
51:        require(!initialized, "Can only initialize once");

52:        require(_collaterals.length != 0, "At least one collateral required");

53:        require(_MCRs.length == _collaterals.length, "Array lengths must match");

54:        require(_CCRs.length == _collaterals.length, "Array lenghts must match");

66:            require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");

69:            require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");

91:        require(_MCR <= config.MCR, "Can only walk down the MCR");

92:        require(_CCR <= config.CCR, "Can only walk down the CCR");

94:        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");

97:        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");

129:        require(collateralConfig[_collateral].allowed, "Invalid collateral address");

```

```solidity
File: Ethos\LUSDToken.sol
134:        require(
            msg.sender == guardianAddress || msg.sender == governanceAddress,
            "LUSD: Caller is not guardian or governance"
        );

280:            revert('LUSD: Invalid s value');

282:        require(deadline >= now, 'LUSD: expired deadline');

288:        require(recoveredAddress == owner, 'LUSD: invalid signature');

347:        require(
            _recipient != address(0) && 
            _recipient != address(this),
            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
        );

352:        require(
            !stabilityPools[_recipient] &&
            !troveManagers[_recipient] &&
            !borrowerOperations[_recipient],
            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
        );

362:        require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");

367:        require(
            troveManagers[msg.sender] ||
            stabilityPools[msg.sender] ||
            borrowerOperations[msg.sender],
            "LUSD: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool"
        );

377:        require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");

384:        require(
            troveManagers[msg.sender] || stabilityPools[msg.sender],
            "LUSD: Caller is neither TroveManager nor StabilityPool");

390:        require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");

394:        require(!mintingPaused, "LUSDToken: Minting is currently paused");

```

```solidity
File: Ethos\ReaperBaseStrategyv4.sol
81:        require(_multisigRoles.length == 3, "Invalid number of multisig roles");

96:        require(msg.sender == vault, "Only vault can withdraw");

97:        require(_amount != 0, "Amount cannot be zero");

98:        require(_amount <= balanceOf(), "Ammount must be less than balance");

191:        require(
            upgradeProposalTime + UPGRADE_TIMELOCK < block.timestamp,
            "Upgrade cooldown not initiated or still ongoing"
        );

```

```solidity
File: Ethos\ReaperVaultERC4626.sol
270:        require(y != 0, "Division by 0");

```

```solidity
File: Ethos\ReaperVaultV2.sol
150:        require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");

151:        require(_strategy != address(0), "Invalid strategy address");

152:        require(strategies[_strategy].activation == 0, "Strategy already added");

153:        require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");

154:        require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");

155:        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");

156:        require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");

180:        require(strategies[_strategy].activation != 0, "Invalid strategy address");

181:        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");

193:        require(strategies[_strategy].activation != 0, "Invalid strategy address");

197:        require(totalAllocBPS <= PERCENT_DIVISOR, "Invalid BPS value");

261:        require(queueLength != 0, "Queue must not be empty");

267:            require(params.activation != 0, "Invalid strategy address");

321:        require(!emergencyShutdown, "Cannot deposit during emergency shutdown");

322:        require(_amount != 0, "Invalid amount");

324:        require(pool + _amount <= tvlCap, "Vault is full");

364:        require(_shares != 0, "Invalid amount");

404:            require(
                totalLoss <= ((value + totalLoss) * withdrawMaxLoss) / PERCENT_DIVISOR,
                "Withdraw loss exceeds slippage"
            );

436:        require(loss <= allocation, "Strategy loss cannot be greater than allocation");

497:        require(strategy.activation != 0, "Unauthorized strategy");

568:        require(_withdrawMaxLoss <= PERCENT_DIVISOR, "Invalid BPS value");

619:        require(degradation <= DEGRADATION_COEFFICIENT, "Degradation cannot be more than 100%");

629:        require(newTreasury != address(0), "Invalid address");

639:        require(_token != address(token), "!token");

```

```solidity
File: Ethos\StabilityPool.sol
849:        require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");

853:        require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");

865:            require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");

870:        require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');

874:        require(_amount > 0, 'StabilityPool: Amount must be non-zero');

```

```solidity
File: Ethos\LQTY\CommunityIssuance.sol
70:        require(!initialized, "issuance has been initialized");

102:        require(amount != 0, "cannot fund 0");

133:        require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");

```

```solidity
File: Ethos\LQTY\LQTYStaking.sol
253:        require(
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == activePoolAddress,
            "LQTYStaking: caller is not TroveM or ActivePool"
        );

262:        require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");

266:        require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');  

270:        require(_amount > 0, 'LQTYStaking: Amount must be non-zero');

```

### [GAS-02] Don't Initialize Variables with Default Value
Uninitialized variables are assigned with the types default value.<br>Explicitly initializing a variable with it's default value costs unnecessary gas.

*Instances (27)*:
```solidity
File: Ethos\ActivePool.sol
32:    bool public addressesSet = false;

108:        for(uint256 i = 0; i < numCollaterals; i++) {

```

```solidity
File: Ethos\CollateralConfig.sol
18:    bool public initialized = false;

56:        for(uint256 i = 0; i < _collaterals.length; i++) {

```

```solidity
File: Ethos\LUSDToken.sol
37:    bool public mintingPaused = false;

```

```solidity
File: Ethos\ReaperBaseStrategyv4.sol
77:        for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {

100:        uint256 amountFreed = 0;

112:        uint256 debt = 0;

117:        uint256 repayment = 0;

```

```solidity
File: Ethos\ReaperStrategyGranarySupplyOnly.sol
35:    uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;

117:        for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {

165:        for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {

```

```solidity
File: Ethos\ReaperVaultV2.sol
128:        for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {

264:        for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {

369:            uint256 totalLoss = 0;

371:            uint256 vaultBalance = 0;

372:            for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {

```

```solidity
File: Ethos\StabilityPool.sol
351:        for (uint i = 0; i < numCollaterals; i++) {

397:        for (uint i = 0; i < numCollaterals; i++) {

640:        for (uint i = 0; i < assets.length; i++) {

810:            for (uint i = 0; i < collaterals.length; i++) {

831:        for (uint i = 0; i < collaterals.length; i++) {

859:        for (uint i = 0; i < numCollaterals; i++) {

```

```solidity
File: Ethos\LQTY\CommunityIssuance.sol
21:    bool public initialized = false;

```

```solidity
File: Ethos\LQTY\LQTYStaking.sol
206:        for (uint i = 0; i < assets.length; i++) {

228:        for (uint i = 0; i < collaterals.length; i++) {

240:        for (uint i = 0; i < numCollaterals; i++) {

```

### [GAS-03] Long Revert String
`require()`/`revert()` strings longer than 32 bytes cost extra gas

*Instances (36)*:
```solidity
File: Ethos\ActivePool.sol
107:        require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");

133:        require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

```

```solidity
File: Ethos\BorrowerOperations.sol
525:        require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");

529:        require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");

530:        require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");

534:        require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");

538:        require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");

543:        require(status == 1, "BorrowerOps: Trove does not exist or is closed");

552:        require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");

568:        require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");

617:        require(_newICR >= _MCR, "BorrowerOps: An operation that would result in ICR < MCR is not permitted");

621:        require(_newICR >= _CCR, "BorrowerOps: Operation must leave trove with ICR >= CCR");

625:        require(_newICR >= _oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");

629:        require(_newTCR >= _CCR, "BorrowerOps: An operation that would result in TCR < CCR is not permitted");

633:        require (_netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");

637:        require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");

641:        require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");

645:        require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");

```

```solidity
File: Ethos\LUSDToken.sol
362:        require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");

377:        require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");

390:        require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");

394:        require(!mintingPaused, "LUSDToken: Minting is currently paused");

```

```solidity
File: Ethos\ReaperBaseStrategyv4.sol
98:        require(_amount <= balanceOf(), "Ammount must be less than balance");

```

```solidity
File: Ethos\ReaperVaultV2.sol
150:        require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");

321:        require(!emergencyShutdown, "Cannot deposit during emergency shutdown");

436:        require(loss <= allocation, "Strategy loss cannot be greater than allocation");

619:        require(degradation <= DEGRADATION_COEFFICIENT, "Degradation cannot be more than 100%");

```

```solidity
File: Ethos\StabilityPool.sol
849:        require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");

853:        require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");

865:            require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");

870:        require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');

874:        require(_amount > 0, 'StabilityPool: Amount must be non-zero');

```

```solidity
File: Ethos\LQTY\CommunityIssuance.sol
133:        require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");

```

```solidity
File: Ethos\LQTY\LQTYStaking.sol
262:        require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");

266:        require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');  

270:        require(_amount > 0, 'LQTYStaking: Amount must be non-zero');

```

### [GAS-04] Use Shift Right/Left instead of Division/Multiplication if possible
A division/multiplication by any number `x` being a power of 2 can be calculated by shifting `log2(x)` to the right/left.<br>While the `DIV` opcode uses 5 gas, the `SHR` opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

*Instances (2)*:
```solidity
File: Ethos\TroveManager.sol
51:     * (1/2) = d^720 => d = (1/2)^(1/720)

51:     * (1/2) = d^720 => d = (1/2)^(1/720)

```

### [GAS-05] `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

*Instances (17)*:
```solidity
File: Ethos\ActivePool.sol
108:        for(uint256 i = 0; i < numCollaterals; i++) {

```

```solidity
File: Ethos\CollateralConfig.sol
56:        for(uint256 i = 0; i < _collaterals.length; i++) {

```

```solidity
File: Ethos\LUSDToken.sol
286:                         _nonces[owner]++, deadline))));

```

```solidity
File: Ethos\ReaperVaultERC4626.sol
273:        if (x % y != 0) q++;

```

```solidity
File: Ethos\StabilityPool.sol
351:        for (uint i = 0; i < numCollaterals; i++) {

397:        for (uint i = 0; i < numCollaterals; i++) {

640:        for (uint i = 0; i < assets.length; i++) {

810:            for (uint i = 0; i < collaterals.length; i++) {

831:        for (uint i = 0; i < collaterals.length; i++) {

859:        for (uint i = 0; i < numCollaterals; i++) {

```

```solidity
File: Ethos\TroveManager.sol
608:        for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {

690:        for (vars.i = 0; vars.i < _n; vars.i++) {

808:        for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

882:        for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

```

```solidity
File: Ethos\LQTY\LQTYStaking.sol
206:        for (uint i = 0; i < assets.length; i++) {

228:        for (uint i = 0; i < collaterals.length; i++) {

240:        for (uint i = 0; i < numCollaterals; i++) {

```

### [GAS-06] Splitting `require()` statements that use `&&` saves gas
Instead of using operator `&&` on a single `require`. Using a two `require` can save more gas.
i.e. for `require(version == 1 && _tokenAmount > 0, "nope");` use:
```solidity
require(version == 1);
require(_tokenAmount > 0);
```

*Instances (2)*:
```solidity
File: Ethos\BorrowerOperations.sol
653:            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must be between 0.5% and 100%");

```

```solidity
File: Ethos\TroveManager.sol
1539:        require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

```

### [GAS-07] Superfluous event fields
`block.number` and `block.timestamp` are added to the event information by default, so adding them manually will waste additional gas.

*Instances (1)*:
```solidity
File: Ethos\TroveManager.sol
1505:            emit LastFeeOpTimeUpdated(block.timestamp);

```

### [GAS-08] Setting the `constructor` to `payable`
Saves ~13 gas per instance

*Instances (6)*:
```solidity
File: Ethos\LUSDToken.sol
84:    constructor
    ( 
        address _troveManagerAddress,
        address _stabilityPoolAddress,
        address _borrowerOperationsAddress,
        address _governanceAddress,
        address _guardianAddress
    ) 
        public 
    {  

```

```solidity
File: Ethos\ReaperBaseStrategyv4.sol
61:    constructor() initializer {}

```

```solidity
File: Ethos\ReaperVaultERC4626.sol
16:    constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}

```

```solidity
File: Ethos\ReaperVaultV2.sol
111:    constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) ERC20(string(_name), string(_symbol)) {

```

```solidity
File: Ethos\TroveManager.sol
225:    constructor() public {

```

```solidity
File: Ethos\LQTY\CommunityIssuance.sol
57:    constructor() public {

```

### [GAS-09] Usage of uint/int smaller than 32 bytes
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html<br>Use a larger size then downcast where needed.

*Instances (5)*:
```solidity
File: Ethos\LUSDToken.sol
35:    uint8 constant internal _DECIMALS = 18;

268:        uint8 v, 

407:    function decimals() external view override returns (uint8) {

```

```solidity
File: Ethos\ReaperStrategyGranarySupplyOnly.sol
35:    uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;

```

```solidity
File: Ethos\ReaperVaultV2.sol
650:    function decimals() public view override returns (uint8) {

```

### [GAS-10] Use `<`/`>` instead of `>=`/`>=`
In Solidity, there is no single op-code for <= or >= expressions. What happens under the hood is that the Solidity compiler executes the LT/GT (less than/greater than) op-code and afterwards it executes an ISZERO op-code to check if the result of the previous comparison (LT/ GT) is zero and validate it or not. Example:
```solidity
// Gas cost: 21394
function check() exernal pure returns (bool) {
		return 3 >= 3;
}
```
```solidity
// Gas cost: 21391
function check() exernal pure returns (bool) {
		return 3 > 2;
}
```
The gas cost between these contract differs by 3 which is the cost executing the ISZERO op-code,**making the use of < and > cheaper than <= and >=.**

*Instances (3)*:
```solidity
File: Ethos\ActivePool.sol
127:        require(_bps <= 10_000, "Invalid BPS value");

133:        require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

```

```solidity
File: Ethos\TroveManager.sol
372:        if (TroveOwners[_collateral].length <= 1) {return singleLiquidation;} // don't liquidate if last trove
```

### [GAS-11] Using fixed bytes is cheaper than using `string`
Use `bytes` for arbitrary-length raw byte data and string for arbitrary-length `string` (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of `bytes1`to `bytes32` because they are much cheaper. Example:
```solidity
// Before
string a;
function add(string str) {
	a = str;
}

// After
bytes32 a;
function add(bytes32 str) public {
	a = str;
}
```

*Instances (18)*:
```solidity
File: Ethos\ActivePool.sol
30:    string constant public NAME = "ActivePool";

```

```solidity
File: Ethos\BorrowerOperations.sol
21:    string constant public NAME = "BorrowerOperations";

```

```solidity
File: Ethos\LUSDToken.sol
32:    string constant internal _NAME = "LUSD Stablecoin";

33:    string constant internal _SYMBOL = "LUSD";

34:    string constant internal _VERSION = "1";

43:    // keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

43:    // keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

399:    function name() external view override returns (string memory) {

403:    function symbol() external view override returns (string memory) {

411:    function version() external view override returns (string memory) {

```

```solidity
File: Ethos\ReaperVaultERC4626.sol
18:        string memory _name,

19:        string memory _symbol,

```

```solidity
File: Ethos\ReaperVaultV2.sol
113:        string memory _name,

114:        string memory _symbol,

```

```solidity
File: Ethos\StabilityPool.sol
150:    string constant public NAME = "StabilityPool";

```

```solidity
File: Ethos\TroveManager.sol
19:    // string constant public NAME = "TroveManager";

```

```solidity
File: Ethos\LQTY\CommunityIssuance.sol
19:    string constant public NAME = "CommunityIssuance";

```

```solidity
File: Ethos\LQTY\LQTYStaking.sol
23:    string constant public NAME = "LQTYStaking";

```

### [GAS-12] `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables
Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/MiniGlome/f462d69a30f68c89175b0ce24ce37cae)**
Same for `-=`, `*=` and `/=`.

*Instances (9)*:

```solidity
File: Ethos\ReaperVaultV2.sol
168:        totalAllocBPS += _allocBPS;

194:        totalAllocBPS -= strategies[_strategy].allocBPS;

196:        totalAllocBPS += _allocBPS;

214:        totalAllocBPS -= strategies[_strategy].allocBPS;

395:                strategies[stratAddr].allocated -= actualWithdrawn;

396:                totalAllocated -= actualWithdrawn;

445:                totalAllocBPS -= bpsChange;

452:        totalAllocated -= loss;

521:            totalAllocated += vars.credit;

```