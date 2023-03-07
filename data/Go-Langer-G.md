## Consider optimizing sendCollateral for batch operations in ActivePool.sol

```solidity 
    function sendCollateral(address _collateral, address _account, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveMorSP();
        _rebalance(_collateral, _amount);
        collAmount[_collateral] = collAmount[_collateral].sub(_amount);
        emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);
        emit CollateralSent(_collateral, _account, _amount);

        if (_account == defaultPoolAddress) {
            IERC20(_collateral).safeIncreaseAllowance(defaultPoolAddress, _amount);
            IDefaultPool(defaultPoolAddress).pullCollateralFromActivePool(_collateral, _amount);
        } else if (_account == collSurplusPoolAddress) {
            IERC20(_collateral).safeIncreaseAllowance(collSurplusPoolAddress, _amount);
            ICollSurplusPool(collSurplusPoolAddress).pullCollateralFromActivePool(_collateral, _amount);
        } else {
            IERC20(_collateral).safeTransfer(_account, _amount);
        }
    }

```

Refactored code:

```
    function batchSendCollateral(address[] calldata _collateral, address[] calldata _account, uint[] calldata _amount) external override {
        _requireCallerIsBOorTroveMorSP();
        uint256 length = _collateral.length;
        for (uint256 i = 0; i < length; i++) {
        _requireValidCollateralAddress(_collateral[i]);
            _rebalance(_collateral[i], _amount[i]);
         collAmount[_collateral[i]] = collAmount[_collateral[i]].sub(_amount[i]);
             emit ActivePoolCollateralBalanceUpdated(_collateral[i], collAmount[_collateral[i]]);
           emit CollateralSent(_collateral[i], _account[i], _amount[i]);
        }
     }
```

### There are 65 instances of this:

171: Ethos-Core/contracts/ActivePool.sol function sendCollaterall

172: Ethos-Core/contracts/BorrowerOperations.sol function openTrove

242: Ethos-Core/contracts/BorrowerOperations.sol function addColl

248: Ethos-Core/contracts/BorrowerOperations.sol function moveCollateralGainToTrove

259: Ethos-Core/contracts/BorrowerOperations.sol function withdrawLUSD

254: Ethos-Core/contracts/BorrowerOperations.sol function withdrawColl

264: Ethos-Core/contracts/BorrowerOperations.sol function repayLUSD
268: Ethos-Core/contracts/BorrowerOperations.sol function adjustTrove
282: Ethos-Core/contracts/BorrowerOperations.sol function _adjustTrove

419: Ethos-Core/contracts/BorrowerOperations.sol function _triggerBorrowingFee

432: Ethos-Core/contracts/BorrowerOperations.sol function _getUSDValue

455: Ethos-Core/contracts/BorrowerOperations.sol function _updateTroveFromAdjustment

476: Ethos-Core/contracts/BorrowerOperations.sol function _moveTokensAndCollateralfromAdjustment

511: Ethos-Core/contracts/BorrowerOperations.sol function _withdrawLUSD

517: Ethos-Core/contracts/BorrowerOperations.sol function _repayLUSD

528: Ethos-Core/contracts/BorrowerOperations.sol function _requireSufficientCollateralBalanceAndAllowance

533: Ethos-Core/contracts/BorrowerOperations.sol function _requireSingularCollChange

537: Ethos-Core/contracts/BorrowerOperations.sol function _requireNonZeroAdjustment

541: Ethos-Core/contracts/BorrowerOperations.sol function _requireTroveisActive

546: Ethos-Core/contracts/BorrowerOperations.sol function _requireTroveisNotActive

555: Ethos-Core/contracts/BorrowerOperations.sol function _requireNotInRecoveryMode

616: Ethos-Core/contracts/BorrowerOperations.sol function _requireICRisAboveMCR

620: Ethos-Core/contracts/BorrowerOperations.sol function _requireICRisAboveCCR

624: Ethos-Core/contracts/BorrowerOperations.sol function _requireNewICRisAboveOldICR

628: Ethos-Core/contracts/BorrowerOperations.sol function _requireNewTCRisAboveCCR

636: Ethos-Core/contracts/BorrowerOperations.sol function _requireValidLUSDRepayment

661: Ethos-Core/contracts/BorrowerOperations.sol function _getNewNominalICRFromTroveChange

682: Ethos-Core/contracts/BorrowerOperations.sol function _getNewICRFromTroveChange

703: Ethos-Core/contracts/BorrowerOperations.sol function _getNewTroveAmounts

724: Ethos-Core/contracts/BorrowerOperations.sol function _getNewTCRFromTroveChange

160: Ethos-Core/contracts/LUSDToken.sol function upgradeProtocol

196: Ethos-Core/contracts/LUSDToken.sol function sendToPool

201: Ethos-Core/contracts/LUSDToken.sol function returnFromPool


235: Ethos-Core/contracts/LUSDToken.sol function transferFrom

262: Ethos-Core/contracts/LUSDToken.sol function permit

410: Ethos-Core/contracts/StabilityPool.sol function depositSnapshots_S 

466: Ethos-Core/contracts/StabilityPool.sol function offset

939: Ethos-Core/contracts/TroveManager.sol function redeemCloseTrove

957: Ethos-Core/contracts/TroveManager.sol function reInsert

962: Ethos-Core/contracts/TroveManager.sol function updateDebtAndCollAndStakesPostRedemption

982: Ethos-Core/contracts/TroveManager.sol function burnLUSDAndEmitRedemptionEvent

1016: Ethos-Core/contracts/TroveManager.sol function redeemCollateral


1111: Ethos-Core/contracts/TroveManager.sol function updateTroveRewardSnapshots


1195: Ethos-Core/contracts/TroveManager.sol function updateStakeAndTotalStakes

1230: Ethos-Core/contracts/TroveManager.sol function _redistributeDebtAndColl

1273: Ethos-Core/contracts/TroveManager.sol function closeTrove

1278: Ethos-Core/contracts/TroveManager.sol function _closeTrove

1339: Ethos-Core/contracts/TroveManager.sol function _removeTroveOwner

1373: Ethos-Core/contracts/TroveManager.sol function _checkPotentialRecoveryMode

1397: Ethos-Core/contracts/TroveManager.sol function updateBaseRateFromRedemption

1448: Ethos-Core/contracts/TroveManager.sol function _calcRedemptionFee

1479: Ethos-Core/contracts/TroveManager.sol function _calcBorrowingFee

1534: Ethos-Core/contracts/TroveManager.sol function _requireTroveIsActive

1548: Ethos-Core/contracts/TroveManager.sol function getTroveStake

1552: Ethos-Core/contracts/TroveManager.sol function getTroveDebt

1556: Ethos-Core/contracts/TroveManager.sol function getTroveColl

1562: Ethos-Core/contracts/TroveManager.sol function setTroveStatus

1567: Ethos-Core/contracts/TroveManager.sol function increaseTroveColl

1574: Ethos-Core/contracts/TroveManager.sol function decreaseTroveColl

Ethos-Core/contracts/TroveManager.sol function increaseTroveDebt

1581: Ethos-Core/contracts/TroveManager.sol function increaseTroveDebt

1588: Ethos-Core/contracts/TroveManager.sol function decreaseTroveDebt

493: Ethos-Vault/contracts/ReaperVaultV2.sol function report

202: Ethos-Vault/contracts/ReaperVaultERC4626.sol function withdraw

258: Ethos-Vault/contracts/ReaperVaultERC4626.sol function redeem

269: Ethos-Vault/contracts/ReaperVaultERC4626.sol function roundUpDiv

## Unchecked For Loop

Consider moving i++ (change to ++i for gas optimization) outside of the for loop into an unchecked block to save on gas fees.

```
71: Ethos-Core/contracts/ActivePool.sol function setAddresses

        address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();
        uint256 numCollaterals = collaterals.length;
        require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");
            for(uint256 i = 0; i < numCollaterals; i++) {
            address collateral = collaterals[i];
            address vault = _erc4626vaults[i];
            require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");
            yieldGenerator[collateral] = vault;
        } 
```

Refactored code:

```
address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();
require(collaterals.length == _erc4626vaults.length, "Vaults array length must match number of collaterals");

for(uint256 i = 0; i < collaterals.length; i++) {
    address collateral = collaterals[i];
    address vault = _erc4626vaults[i];
    unchecked {
                i++;
                }
    require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");
    yieldGenerator[collateral] = vault;
}
```
There are 6 instances of this:

63: Ethos-Vault/contracts/ReaperVaultV2.sol function __ReaperBaseStrategy_init

114: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol function _harvestCore

160: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol function setHarvestSteps

111: Ethos-Vault/contracts/ReaperVaultV2.sol constructor

258: Ethos-Vault/contracts/ReaperVaultV2.sol function setWithdrawalQueue

359: Ethos-Vault/contracts/ReaperVaultV2.sol function _withdraw

## Multiple require statements

71: Ethos-Core/contracts/ActivePool.sol function setAddresses

Example:

```

    function setAddresses(
        address _collateralConfigAddress,
        address _borrowerOperationsAddress,
        address _troveManagerAddress,
        address _stabilityPoolAddress,
        address _defaultPoolAddress,
        address _collSurplusPoolAddress,
        address _treasuryAddress,
        address _lqtyStakingAddress,
        address[] calldata _erc4626vaults
    )
        external
        onlyOwner
    {
        require(!addressesSet, "Can call setAddresses only once");

        checkContract(_collateralConfigAddress);
        checkContract(_borrowerOperationsAddress);
        checkContract(_troveManagerAddress);
        checkContract(_stabilityPoolAddress);
        checkContract(_defaultPoolAddress);
        checkContract(_collSurplusPoolAddress);
        require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
        checkContract(_lqtyStakingAddress);
```
Refactored:

```
function setAddresses(
        address _collateralConfigAddress,
        address _borrowerOperationsAddress,
        address _troveManagerAddress,
        address _stabilityPoolAddress,
        address _defaultPoolAddress,
        address _collSurplusPoolAddress,
        address _treasuryAddress,
        address _lqtyStakingAddress,
        address[] calldata _erc4626vaults,
        bool _checkContracts
    )
        external
        onlyOwner
    {
        require(
            !addressesSet &&
            _treasuryAddress != address(0) &&
            ICollateralConfig(_collateralConfigAddress).getAllowedCollaterals().length == _erc4626vaults.length &&
            areContractsValid(_checkContracts, _collateralConfigAddress, _borrowerOperationsAddress, _troveManagerAddress, _stabilityPoolAddress, _defaultPoolAddress, _collSurplusPoolAddress, _lqtyStakingAddress),
            "Invalid addresses or vaults array length or contracts"
        );
```

46: Ethos-Core/contracts/CollateralConfig.sol function initialize

85: Ethos-Core/contracts/CollateralConfig.sol function updateCollateralRatio

346: Ethos-Core/contracts/LUSDToken.sol function requireValidRecipient

144: Ethos-Vault/contracts/ReaperVaultV2.sol function addStrategy

178: Ethos-Vault/contracts/ReaperVaultV2.sol function updateStrategyFeeBPS

191: Ethos-Vault/contracts/ReaperVaultV2.sol function updateStrategyAllocBPS

258: Ethos-Vault/contracts/ReaperVaultV2.sol function setWithdrawalQueue

319: Ethos-Vault/contracts/ReaperVaultV2.sol function _deposit

95: Ethos-Vault/contracts/ReaperBaseStrategyV4.sol function withdraw

## Multiple msg.sender variables

"395: Ethos-Core/contracts/ActivePool.sol function _requireCallerIsBOorTroveMorSP"

```
    function _requireCallerIsBOorTroveMorSP() internal view {
        address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());
        require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == stabilityPoolAddress,
            "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool");
    }
```

Multiple msg.sender checks can cost more in gas. Consider a function like this:

```
function _requireCallerIsBOorTroveMorSP() internal view {
     address sender = msg.sender;
     address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());
     require(
     sender == borrowerOperationsAddress ||
     sender == troveManagerAddress ||
     sender == redemptionHelper ||
     sender == stabilityPoolAddress,
     "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool");
} 
```
Other instances:

320: Ethos-Core/contracts/ActivePool.sol function _requireCallerIsBorrowerOperationsOrDefaultPool

## Indexing events

Consider indexing your events list like this for better performance and less gas:

```
event CollateralConfigAddressChanged(address indexed _newCollateralConfigAddress);
event BorrowerOperationsAddressChanged(address indexed _newBorrowerOperationsAddress);
event TroveManagerAddressChanged(address indexed _newTroveManagerAddress);
event CollSurplusPoolAddressChanged(address indexed _collSurplusPoolAddress);
event ActivePoolLUSDDebtUpdated(address indexed _collateral, uint _LUSDDebt);
event ActivePoolCollateralBalanceUpdated(address indexed _collateral, uint _amount);
event YieldingPercentageUpdated(address indexed _collateral, uint256 _bps);
event YieldingPercentageDriftUpdated(uint256 indexed _driftBps);
event YieldClaimThresholdUpdated(address indexed _collateral, uint256 _threshold);
event YieldDistributionParamsUpdated(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit);
```

Other instances:

- Contract BorrowerOperations
- Contract CollateralConfig
- Contract CommunityIssuance
- Contract LQTYStaking
- Contract LUSDToken
- Contract StabilityPool
- Contract TroveManager

## Use bytes32 instead of string in state variables

Use bytes 32 to replace "string" to save on gas fees. As output is fixed under 32 characters, using byes32 reduces gas costs.

#### Contract BorrowerOperations.sol

```
contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOperations {
    using SafeERC20 for IERC20;

    string constant public NAME = "BorrowerOperations";
```

Other instances:

- Contract ActivePool
- Contract CommunityIssuance
- Contract LQTYStaking
- Contract LUSDToken
- Contract StabilityPool