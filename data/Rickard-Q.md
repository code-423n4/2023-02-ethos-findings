# [L-01] Owner can renounce Ownership

## Vulnerability details
### Proof of Concept
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L14-L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L14-L19)
````solidity
7 results - 7 files
/ActivePool.sol
13: import "./Dependencies/Ownable.sol";
26: contract ActivePool is Ownable, CheckContract, IActivePool {

/BorrowerOperations.sol
13: import "./Dependencies/Ownable.sol";
18: contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOperations {

/CollateralConfig.sol
6:  import "./Dependencies/Ownable.sol";
15: contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {

/HintHelpers.sol
9:  import "./Dependencies/Ownable.sol";
12: contract HintHelpers is LiquityBase, Ownable, CheckContract {

/StabilityPool.sol
16: import "./Dependencies/Ownable.sol";
146:  contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {

/CommunityIssuance.sol
9:  import "./Dependencies/Ownable.sol";
14: contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMath { 

/LQTYStaking.sol
7:  import "./Dependencies/Ownable.sol";
18: contract LQTYStaking is ILQTYStaking, Ownable, CheckContract, BaseMath {
````
Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.     

The OpenZeppelin’s Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design.     

Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.     

`onlyOwner` functions:
````solidity
/ActivePool.sol
71: function setAddresses(
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
125:  function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

132:  function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138:  function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144:  function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

214:  function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {

/BorrowerOperations.sol
110:  function setAddresses(
        address _collateralConfigAddress,
        address _troveManagerAddress,
        address _activePoolAddress,
        address _defaultPoolAddress,
        address _stabilityPoolAddress,
        address _gasPoolAddress,
        address _collSurplusPoolAddress,
        address _priceFeedAddress,
        address _sortedTrovesAddress,
        address _lusdTokenAddress,
        address _lqtyStakingAddress
    )
        external
        override
        onlyOwner
    {

/CollateralConfig.sol
46: function initialize(
        address[] calldata _collaterals,
        uint256[] calldata _MCRs,
        uint256[] calldata _CCRs
    ) external override onlyOwner {

85: function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
    ) external onlyOwner checkCollateral(_collateral) {

/HintHelpers.sol
27: function setAddresses(
        address _collateralConfigAddress,
        address _sortedTrovesAddress,
        address _troveManagerAddress
    )
        external
        onlyOwner
    {

/StabilityPool.sol
261:  function setAddresses(
        address _borrowerOperationsAddress,
        address _collateralConfigAddress,
        address _troveManagerAddress,
        address _activePoolAddress,
        address _lusdTokenAddress,
        address _sortedTrovesAddress,
        address _priceFeedAddress,
        address _communityIssuanceAddress
    )
        external
        override
        onlyOwner
    {

/CommunityIssuance.sol
61: function setAddresses
    (
        address _oathTokenAddress, 
        address _stabilityPoolAddress
    ) 
        external 
        onlyOwner 
        override 
    {

101:  function fund(uint amount) external onlyOwner {

120:  function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {

/LQTYStaking.sol
66: function setAddresses
    (
        address _lqtyTokenAddress,
        address _lusdTokenAddress,
        address _troveManagerAddress, 
        address _borrowerOperationsAddress,
        address _activePoolAddress,
        address _collateralConfigAddress
    ) 
        external 
        onlyOwner 
        override 
    {
````

### Recommended Mitigation Steps
We recommend either reimplementing the function to disable it or to clearly specify if it is part of the contract design.
# [L-02] Missing `event` for critical parameters init and change

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L125)
## Vulnerability details
### Proof of Concept
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L125)
````solidity
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
````
Events help non-contract tools to track changes, and events prevent users from being surprised by changes

### Recommended Mitigation Steps
Add Event-Emit.

# [L-03] Missing ReEntrancy Guard to `withdraw` function

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95-L103](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95-L103)
## Vulnerability details
### Proof of Concept
ReaperBaseStrategyv4.sol contract has no Re-Entrancy protection in withdraw function.    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95-L103](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95-L103)
````solidity
    function withdraw(uint256 _amount) external override returns (uint256 loss) {
        require(msg.sender == vault, "Only vault can withdraw");
        require(_amount != 0, "Amount cannot be zero");
        require(_amount <= balanceOf(), "Ammount must be less than balance");


        uint256 amountFreed = 0;
        (amountFreed, loss) = _liquidatePosition(_amount);
        IERC20Upgradeable(want).safeTransfer(vault, amountFreed);
    }
````
If the mint was initiated by a contract without reentrancy guard, it will allow an attacker controlled contract to call the mint again, which may not be desirable to some parties, like allowing minting more than allowed.     
[https://www.paradigm.xyz/2021/08/the-dangers-of-surprising-code](https://www.paradigm.xyz/2021/08/the-dangers-of-surprising-code)

### Recommended Mitigation Steps
Use Openzeppelin or Solmate Re-Entrancy pattern.    
Here is a example of a re-entrancy guard:
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;
contract ReEntrancyGuard {
    bool internal locked;
    modifier noReentrant() {
        require(!locked, "No re-entrancy");
        locked = true;
        _;
        locked = false;
    }
}
````

# [L-04] Lack of ReEntrancy Guards

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L142-L173](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L142-L173)
## Vulnerability details
### Proof of Concept
Whenever `lqtyToken.safeTransfer(msg.sender, LQTYToWithdraw);` is called, the `msg.sender` address can re-enter back to the contracts.    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L142-L173](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L142-L173)
````solidity
    function unstake(uint _LQTYamount) external override {
        uint currentStake = stakes[msg.sender];
        _requireUserHasStake(currentStake);


        // Grab any accumulated ETH and LUSD gains from the current stake
        (address[] memory collGainAssets, uint[] memory collGainAmounts) = _getPendingCollateralGain(msg.sender);
        uint LUSDGain = _getPendingLUSDGain(msg.sender);
        
        _updateUserSnapshots(msg.sender);


        if (_LQTYamount > 0) {
            uint LQTYToWithdraw = LiquityMath._min(_LQTYamount, currentStake);


            uint newStake = currentStake.sub(LQTYToWithdraw);


            // Decrease user's stake and total LQTY staked
            stakes[msg.sender] = newStake;
            totalLQTYStaked = totalLQTYStaked.sub(LQTYToWithdraw);
            emit TotalLQTYStakedUpdated(totalLQTYStaked);


            // Transfer unstaked LQTY to user
            lqtyToken.safeTransfer(msg.sender, LQTYToWithdraw);


            emit StakeChanged(msg.sender, newStake);
        }


        emit StakingGainsWithdrawn(msg.sender, LUSDGain, collGainAssets, collGainAmounts);


        // Send accumulated LUSD and ETH gains to the caller
        lusdToken.transfer(msg.sender, LUSDGain);
        _sendCollGainToUser(collGainAssets, collGainAmounts);
    }
````

### Recommended Mitigation Steps
Apply necessary reentrancy prevention by utilizing the OpenZeppelin’s nonReentrant modifier to block possible re-entrancy.

# [L-05] `initialize()` function can be called by anybody

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73)
## Vulnerability details
### Proof of Concept
`initialize()` function can be called anybody when the contract is not initialized.    
Also, there is no 0 address check in the address arguments of the `initialize()` function, which must be defined.

Here is a definition of `initialize()` function.     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73)
````solidity
62:     function initialize(
63:         address _vault,
64:         address[] memory _strategists,
65:         address[] memory _multisigRoles,
66:         IAToken _gWant
67:     ) public initializer {
68:         gWant = _gWant;
69:         want = _gWant.UNDERLYING_ASSET_ADDRESS();
70:         __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
71:        rewardClaimingTokens = [address(_gWant)];
72:     }
````

### Recommended Mitigation Steps
Add a control that makes `initialize()` only call the Deployer Contract:
````solidity
if (msg.sender != DEPLOYER_ADDRESS) {
						revert NotDeployer();
				}
````
Alternatively, add the `onlyOwner` modifier like it has been done in other contract's `initialize()` function.

# [N-01] For modern and more readable code; update import usages

## Vulnerability details
````solidity
109 results - 12 files

/ActivePool.sol
import './Interfaces/IActivePool.sol';// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ICollateralConfig.sol";// @audit n : For modern and more readable code, update import usages
import './Interfaces/IDefaultPool.sol';// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ICollSurplusPool.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ILQTYStaking.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/IStabilityPool.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ITroveManager.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/SafeMath.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/Ownable.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/CheckContract.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/console.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/SafeERC20.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/IERC4626.sol";// @audit n : For modern and more readable code, update import usages

/BorrowerOperations.sol
import "./Interfaces/IBorrowerOperations.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ICollateralConfig.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ITroveManager.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ILUSDToken.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ICollSurplusPool.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ISortedTroves.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ILQTYStaking.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/LiquityBase.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/Ownable.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/CheckContract.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/console.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/SafeERC20.sol";// @audit n : For modern and more readable code, update import usages

/CollateralConfig.sol
import "./Dependencies/CheckContract.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/Ownable.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/SafeERC20.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ICollateralConfig.sol";// @audit n : For modern and more readable code, update import usages

/LUSDToken.sol
import "./Interfaces/ILUSDToken.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ITroveManager.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/SafeMath.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/CheckContract.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/console.sol";// @audit n : For modern and more readable code, update import usages

/StabilityPool.sol
import './Interfaces/IBorrowerOperations.sol';// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ICollateralConfig.sol";// @audit n : For modern and more readable code, update import usages
import './Interfaces/IStabilityPool.sol';// @audit n : For modern and more readable code, update import usages
import './Interfaces/IBorrowerOperations.sol';// @audit n : For modern and more readable code, update import usages
import './Interfaces/ITroveManager.sol';// @audit n : For modern and more readable code, update import usages
import './Interfaces/ILUSDToken.sol';// @audit n : For modern and more readable code, update import usages
import './Interfaces/ISortedTroves.sol';// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ICommunityIssuance.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/LiquityBase.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/SafeMath.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/LiquitySafeMath128.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/Ownable.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/CheckContract.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/console.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/SafeERC20.sol";// @audit n : For modern and more readable code, update import usages

/TroveManager.sol
import "./Interfaces/ICollateralConfig.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ITroveManager.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/IStabilityPool.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ICollSurplusPool.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ILUSDToken.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ISortedTroves.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/ILQTYStaking.sol";// @audit n : For modern and more readable code, update import usages
import "./Interfaces/IRedemptionHelper.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/LiquityBase.sol";// @audit n : For modern and more readable code, update import usages
// import "./Dependencies/Ownable.sol";
import "./Dependencies/CheckContract.sol";// @audit n : For modern and more readable code, update import usages
import "./Dependencies/IERC20.sol";// @audit n : For modern and more readable code, update import usages

/CommunityIssuance.sol
import "../Dependencies/IERC20.sol"; // @audit n : For modern and more readable code, update import usages
import "../Interfaces/ICommunityIssuance.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/BaseMath.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/LiquityMath.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/Ownable.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/CheckContract.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/SafeMath.sol";// @audit n : For modern and more readable code, update import usages

/LQTYStaking.sol
import "../Dependencies/BaseMath.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/SafeMath.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/Ownable.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/CheckContract.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/console.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/IERC20.sol";// @audit n : For modern and more readable code, update import usages
import "../Interfaces/ICollateralConfig.sol";// @audit n : For modern and more readable code, update import usages
import "../Interfaces/ILQTYStaking.sol";// @audit n : For modern and more readable code, update import usages
import "../Interfaces/ITroveManager.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/LiquityMath.sol";// @audit n : For modern and more readable code, update import usages
import "../Interfaces/ILUSDToken.sol";// @audit n : For modern and more readable code, update import usages
import "../Dependencies/SafeERC20.sol";// @audit n : For modern and more readable code, update import usages

/ReaperStrategyGranarySupplyOnly.sol
import "./abstract/ReaperBaseStrategyv4.sol";// @audit n : For modern and more readable code, update import usages
import "./interfaces/IAToken.sol";// @audit n : For modern and more readable code, update import usages
import "./interfaces/IAaveProtocolDataProvider.sol";// @audit n : For modern and more readable code, update import usages
import "./interfaces/ILendingPool.sol";// @audit n : For modern and more readable code, update import usages
import "./interfaces/ILendingPoolAddressesProvider.sol";// @audit n : For modern and more readable code, update import usages
import "./interfaces/IRewardsController.sol";// @audit n : For modern and more readable code, update import usages
import "./libraries/ReaperMathUtils.sol";// @audit n : For modern and more readable code, update import usages
import "./mixins/VeloSolidMixin.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts-upgradeable/utils/math/MathUpgradeable.sol";// @audit n : For modern and more readable code, update import usages

/ReaperVaultERC4626.sol
import "./ReaperVaultV2.sol";// @audit n : For modern and more readable code, update import usages
import "./interfaces/IERC4626Functions.sol";// @audit n : For modern and more readable code, update import usages

/ReaperVaultV2.sol
import "./interfaces/IERC4626Events.sol";// @audit n : For modern and more readable code, update import usages
import "./interfaces/IStrategy.sol";// @audit n : For modern and more readable code, update import usages
import "./libraries/ReaperMathUtils.sol";// @audit n : For modern and more readable code, update import usages
import "./mixins/ReaperAccessControl.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts/utils/math/Math.sol";// @audit n : For modern and more readable code, update import usages

/ReaperBaseStrategyv4.sol
import "../interfaces/IStrategy.sol"; // @audit n : For modern and more readable code, update import usages
import "../interfaces/IVault.sol";// @audit n : For modern and more readable code, update import usages
import "../libraries/ReaperMathUtils.sol";// @audit n : For modern and more readable code, update import usages
import "../mixins/ReaperAccessControl.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";// @audit n : For modern and more readable code, update import usages
import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";// @audit n : For modern and more readable code, update import usages

````

### Recommended Mitigation Steps
`import {contract1 , contract2} from "filename.sol";`
````solidity
import {BaseMath} from "../Dependencies/BaseMath.sol";
````

# [N-02] Add a timelock to critical functions

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71-L103](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71-L103)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L110-L153](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L110-L153)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L261-L291](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L261-L291)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L232-L278](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L232-L278)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L75](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L75)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66-L91](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66-L91)
## Vulnerability details
It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).    

Consider adding a timelock to:
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71-L103](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71-L103)
````solidity
    function setAddresses(
...        
        collateralConfigAddress = _collateralConfigAddress;
        borrowerOperationsAddress = _borrowerOperationsAddress;
        troveManagerAddress = _troveManagerAddress;
        stabilityPoolAddress = _stabilityPoolAddress;
        defaultPoolAddress = _defaultPoolAddress;
        collSurplusPoolAddress = _collSurplusPoolAddress;
        treasuryAddress = _treasuryAddress;
        lqtyStakingAddress = _lqtyStakingAddress;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L110-L153](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L110-L153)
````solidity
    function setAddresses(
...    
        collateralConfig = ICollateralConfig(_collateralConfigAddress);
        troveManager = ITroveManager(_troveManagerAddress);
        activePool = IActivePool(_activePoolAddress);
        defaultPool = IDefaultPool(_defaultPoolAddress);
        stabilityPoolAddress = _stabilityPoolAddress;
        gasPoolAddress = _gasPoolAddress;
        collSurplusPool = ICollSurplusPool(_collSurplusPoolAddress);
        priceFeed = IPriceFeed(_priceFeedAddress);
        sortedTroves = ISortedTroves(_sortedTrovesAddress);
        lusdToken = ILUSDToken(_lusdTokenAddress);
        lqtyStakingAddress = _lqtyStakingAddress;
        lqtyStaking = ILQTYStaking(_lqtyStakingAddress);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L261-L291](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L261-L291)
````solidity
    function setAddresses(
...     
        borrowerOperations = IBorrowerOperations(_borrowerOperationsAddress);
        collateralConfig = ICollateralConfig(_collateralConfigAddress);
        troveManager = ITroveManager(_troveManagerAddress);
        activePool = IActivePool(_activePoolAddress);
        lusdToken = ILUSDToken(_lusdTokenAddress);
        sortedTroves = ISortedTroves(_sortedTrovesAddress);
        priceFeed = IPriceFeed(_priceFeedAddress);
        communityIssuance = ICommunityIssuance(_communityIssuanceAddress);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L232-L278](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L232-L278)
````solidity
    function setAddresses(
...      
        borrowerOperationsAddress = _borrowerOperationsAddress;
        collateralConfig = ICollateralConfig(_collateralConfigAddress);
        activePool = IActivePool(_activePoolAddress);
        defaultPool = IDefaultPool(_defaultPoolAddress);
        stabilityPool = IStabilityPool(_stabilityPoolAddress);
        gasPoolAddress = _gasPoolAddress;
        collSurplusPool = ICollSurplusPool(_collSurplusPoolAddress);
        priceFeed = IPriceFeed(_priceFeedAddress);
        lusdToken = ILUSDToken(_lusdTokenAddress);
        sortedTroves = ISortedTroves(_sortedTrovesAddress);
        lqtyToken = IERC20(_lqtyTokenAddress);
        lqtyStaking = ILQTYStaking(_lqtyStakingAddress);
        redemptionHelper = IRedemptionHelper(_redemptionHelperAddress);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L75](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L75)
````solidity
    function setAddresses
    (
...
        OathToken = IERC20(_oathTokenAddress);
        stabilityPoolAddress = _stabilityPoolAddress;
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66-L91](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66-L91)
````solidity
    function setAddresses
    (
... 
        lqtyToken = IERC20(_lqtyTokenAddress);
        lusdToken = ILUSDToken(_lusdTokenAddress);
        troveManagerAddress = _troveManagerAddress;
        borrowerOperationsAddress = _borrowerOperationsAddress;
        activePoolAddress = _activePoolAddress;
        collateralConfig = ICollateralConfig(_collateralConfigAddress);
````

### Recommended Mitigation Steps
Add a timelock to the functions above.

# [N-03] Constants should be defined rather than using magic numbers

## Vulnerability details
It is bad practice to use numbers directly in code without explanation.
````solidity
9 results - 2 files

/ActivePool.sol
127:  require(_bps <= 10_000, "Invalid BPS value");

133:  require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

145:  require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");

258:  vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);

262:  vars.finalYieldingAmount = vars.finalBalance.mul(vars.yieldingPercentage).div(10_000);

291:  vars.treasurySplit = vars.profit.mul(yieldSplitTreasury).div(10_000);

296:  vars.stakingSplit = vars.profit.mul(yieldSplitStaking).div(10_000);

/ReaperVaultV2.sol
155:  require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");

181:  require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
````
### Tools Used
VS Code
### Recommended Mitigation Steps
Consider storing the value after declaring a `constant`.

# [N-04] Interchangeable usage of `uint` and `uint256`
## Lines of code
Present in most files.

### Recommended Mitigation Steps
Consider using only one approach throughout the codebase, e.g. only `uint` or only `uint256`.
````solidity
  struct LocalVariables_adjustTrove {
-          uint256 collCCR;
-          uint256 collMCR;
-          uint256 collDecimals;
+          uint collCCR;
+          uint collMCR;
+          uint collDecimals;
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
````
or
````solidity
struct LocalVariables_adjustTrove { 
        uint256 collCCR;
        uint256 collMCR;
        uint256 collDecimals;
-        uint price; 
-        uint collChange;
-        uint netDebtChange;
+        uint price; 
+        uint collChange;
+        uint netDebtChange;
        bool isCollIncrease;
-        uint debt;
-        uint coll;
-        uint oldICR;
-        uint newICR;
-        uint newTCR;
-        uint LUSDFee;
-        uint newDebt;
-        uint newColl;
-        uint stake;
+        uint256 debt;
+        uint256 coll;
+        uint256 oldICR;
+        uint256 newICR;
+        uint256 newTCR;
+        uint256 LUSDFee;
+        uint256 newDebt;
+        uint256 newColl;
+        uint256 stake;
    }
````

# [N-05] Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`)

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L131](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L131)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L494](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L494)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L496](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L496)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1132](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1132)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1147](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1147)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1251-L1252](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1251-L1252)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125)
## Vulnerability details
While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist.   
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L131](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L131)  
````solidity
131:  vars.newColl = vars.collAmount.sub(vars.maxRedeemableLUSD.mul(10**vars.collDecimals).div(_price));
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L494](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L494)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L496](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L496)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1132](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1132)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1147](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1147)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1251-L1252](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1251-L1252)   
````solidity
494:  cappedCollPortion = cappedCollPortion.div(10 ** (LiquityMath.CR_CALCULATION_DECIMALS - _collDecimals));

496:  cappedCollPortion = cappedCollPortion.mul(10 ** (_collDecimals - LiquityMath.CR_CALCULATION_DECIMALS));

1132: uint pendingCollateralReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);

1147: uint pendingLUSDDebtReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);

1251: uint collateralNumerator = _coll.mul(10**_collDecimals).add(lastCollateralError_Redistribution[_collateral]);
1252: uint LUSDDebtNumerator = _debt.mul(10**_collDecimals).add(lastLUSDDebtError_Redistribution[_collateral]);
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125)
````solidity
40: uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
125:  lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
````


# [N-06] Use of `bytes.concat()` instead of `abi.encodePacked()`

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L178](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L178)
## Vulnerability details
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L178](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L178)
````solidity
178:            latestRandomSeed = uint(keccak256(abi.encodePacked(latestRandomSeed)));
````
Rather than using `abi.encodePacked` for appending bytes, since version 0.8.4, `bytes.concat()` is enabled.     

Since version 0.8.4 for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked(,)`.


# [N-07] Signature Malleability of EVM’s `ecrecover()`

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287)
## Vulnerability details
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287)
````solidity
287:        address recoveredAddress = ecrecover(digest, v, r, s);
````
Description: The function calls the Solidity ecrecover() function directly to verify the given signatures. However, the ecrecover() EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.    

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin’s ECDSA library.


### Recommended Mitigation Steps
Use the ecrecover function from OpenZeppelin’s ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

# [N-08] `2**<n> - 1` should be re-written as `type(uint<n>).max`

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1329](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1329)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1346](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1346)
## Vulnerability details
Earlier versions of solidity can use `uint<n>(-1)` instead. Expressions not including the - 1 can often be re-written to accomodate the change (e.g. by using a > rather than a >=, which will also save some gas).          
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1329](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1329)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1346](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1346)
````solidity
1329: index = uint128(TroveOwners[_collateral].length.sub(1));
1346: uint idxLast = length.sub(1);
````

# [N-09] Not using the latest version of OpenZeppelin from dependencies

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/package.json#L34](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/package.json#L34)     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/package.json#L41-L42](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/package.json#L41-L42)
## Vulnerability details
The `package.json` configuration file says that the project is using 4.7.3 of OpenZeppelin which has a not last update version.     
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/package.json#L34](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/package.json#L34)
````solidity
34: "@openzeppelin/contracts": "^3.3.0",
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/package.json#L41-L42](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/package.json#L41-L42)
````solidity
41: "@openzeppelin/contracts": "^4.7.3",
42: "@openzeppelin/contracts-upgradeable": "^4.7.3",
````
### Recommended Mitigation Steps
Use patched versions (4.8.1).
  
# [N-10] `constant` redefined elsewhere

## Vulnerability details
Consider defining in only one contract so that values cannot become out of sync when only one location is updated. A cheap way to store constants in a single location is to create an `internal constant` in a `library`.     
If the variable is a local cache of another contract’s value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers don’t get out of sync.
````solidity
12 results - 8 files

Ethos-Core/contracts/ActivePool.sol
30: string constant public NAME = "ActivePool";

Ethos-Core/contracts/BorrowerOperations.sol
21: string constant public NAME = "BorrowerOperations";

Ethos-Core/contracts/CollateralConfig.sol
21: uint256 constant public MIN_ALLOWED_MCR = 1.1 ether;
25: uint256 constant public MIN_ALLOWED_CCR = 1.5 ether;

Ethos-Core/contracts/HintHelpers.sol
13: string constant public NAME = "HintHelpers";

Ethos-Core/contracts/StabilityPool.sol
150:  string constant public NAME = "StabilityPool";
197:  uint public constant SCALE_FACTOR = 1e9;

Ethos-Core/contracts/TroveManager.sol
48: uint constant public SECONDS_IN_ONE_MINUTE = 60;

53: uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;  
54: uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5% 
55: uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%

61: uint constant public BETA = 2;

Ethos-Core/contracts/LQTY/CommunityIssuance.sol
19: string constant public NAME = "CommunityIssuance";

Ethos-Core/contracts/LQTY/LQTYStaking.sol
23: string constant public NAME = "LQTYStaking";
````
````solidity
19 results - 3 files
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
24: address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;  
25: ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =
26: ILendingPoolAddressesProvider(0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6); 
27: IAaveProtocolDataProvider public constant DATA_PROVIDER =
28: IAaveProtocolDataProvider(0x9546F673eF71Ff666ae66d01Fd6E7C6Dae5a9995);  
29: IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);
  
Ethos-Vault/contracts/ReaperVaultV2.sol
40: uint256 public constant DEGRADATION_COEFFICIENT = 10**18;
41: uint256 public constant P
  
73: bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74: bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75: bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76: bytes32 public constant ADMIN = keccak256("ADMIN");
  
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
23: uint256 public constant PERCENT_DIVISOR = 10_000;  
24: uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF 
25: uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;

49: bytes32 public constant KEEPER = keccak256("KEEPER");
50: bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51: bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52: bytes32 public constant ADMIN = keccak256("ADMIN")
```` 

# [N-11] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

## Vulnerability details
````solidity
1 result - 1 file
  
Ethos-Core/contracts/LQTY/CommunityIssuance.sol
58: distributionPeriod = 14 days;
````
### Recommended Mitigation Steps
````solidity  
Ethos-Core/contracts/LQTY/CommunityIssuance.sol
- 58: distributionPeriod = 14 days; 
+ 58: distributionPeriod = 14 days; // 1_209_600 ( 14 * 24 * 60 * 60)
````
  
# [N-12] Inconsistent Solidity Versions

## Vulnerability details
Different Solidity compiler versions are used throughout the Ethos protocol.    
To be more specific, the two main mappings (Ethos-Core and Ethos-Vault) mix solidity versions:
````solidity 
Ethos-Core/contracts/ActivePool.sol
3:  pragma solidity 0.6.11;

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
3:  pragma solidity ^0.8.0;
````

### Recommended Mitigation Steps
````solidity  
Versions must be consistent with each other.
````
  
# [N-13] Inconsistent spacing in contract

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L28](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L28)
## Vulnerability details
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25)    
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L28](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L28)
````solidity 
25:     mapping( address => uint) public stakes;
28:     mapping (address => uint) public F_Collateral;  // Running sum of collateral fees per-LQTY-staked
````


# [N-14] Lack of event emission after critical `initialize()` function

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73)
## Vulnerability details
To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the `initialize()` function:
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73)
````solidity 
62:     function initialize(
63:         address _vault,
64:         address[] memory _strategists,
65:         address[] memory _multisigRoles,
66:         IAToken _gWant
67:     ) public initializer {
68:         gWant = _gWant;
69:         want = _gWant.UNDERLYING_ASSET_ADDRESS();
70:         __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
71:        rewardClaimingTokens = [address(_gWant)];
72:     }
````


# [N-15] Mark visibility of `initialize()` functions as `external`

## Lines of code
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73)
## Vulnerability details
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L73)
````solidity 
62:     function initialize(
63:         address _vault,
64:         address[] memory _strategists,
65:         address[] memory _multisigRoles,
66:         IAToken _gWant
67:     ) public initializer {
68:         gWant = _gWant;
69:         want = _gWant.UNDERLYING_ASSET_ADDRESS();
70:         __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
71:        rewardClaimingTokens = [address(_gWant)];
72:     }
````
If someone wants to extend via inheritance, it might make more sense that the overridden `initialize()` function calls the internal `{}_init` function, not the parent public `initialize()` function.    

External instead of public would give more sense of the `initialize()` functions to behave like a constructor (only called on deployment, so should only be called externally).    

From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract.     

It might cost a bit less gas to use external over public.     

It is possible to override a function from external to public (= “opening it up”), but it is not possible to override a function from public to external (= “narrow it down”).

For the above reasons, you can change `initialize()` to external:     
[https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750)


# [N-16] Constant values such as a call to `keccak256()`, should use `immutable` rather than `constant`

## Vulnerability details
There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.      

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.       

Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the `constructor`. 
````solidity
8 results - 2 files

Ethos-Vault/contracts/ReaperVaultV2.sol
73	bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74	bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75	bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76:	bytes32 public constant ADMIN = keccak256("ADMIN");

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
49	bytes32 public constant KEEPER = keccak256("KEEPER");
50	bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51	bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52:	bytes32 public constant ADMIN = keccak256("ADMIN");
````



# [N-17] Assembly Codes Specific – Should Have Comments

## Vulnerability details
Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.    
     
This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.    
     
Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.     
````solidity
299        assembly {
300            chainID := chainid()
301:        }
````
[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L299-L301](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L299-L301)



# [N-18] Use `require` instead of `assert`

## Vulnerability details
The Solidity `assert()` function is meant to assert invariants. Properly functioning code should never reach a failing assert statement.
````solidity
20 results - 4 files

Ethos-Core/contracts/BorrowerOperations.sol
128:  assert(MIN_NET_DEBT > 0);

197:  assert(vars.compositeDebt > 0);

301:  assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

331:  assert(_collWithdrawal <= vars.coll);

Ethos-Core/contracts/LUSDToken.sol
312:  assert(sender != address(0));
313:  assert(recipient != address(0));

321:  assert(account != address(0));

329:  assert(account != address(0));

337:  assert(owner != address(0));
338:  assert(spender != address(0));

Ethos-Core/contracts/StabilityPool.sol
526:  assert(_debtToOffset <= _totalLUSDDeposits);

551:  assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);

591:  assert(newP > 0);

Ethos-Core/contracts/TroveManager.sol
417:  assert(_LUSDInStabPool != 0);

1224: assert(totalStakesSnapshot[_collateral] > 0);

1279: assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

1342: assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

1348: assert(index <= idxLast);

1414: assert(newBaseRate > 0); // Base rate is always non-zero after redemption

1489: assert(decayedBaseRate <= DECIMAL_PRECISION);
````

### Recommended Mitigation Steps
Consider whether the condition checked in the `assert()` is actually an invariant. If not, replace the `assert()` statement with a `require()` statement.

# [I-01] Missing natspec comments for contract's constructor, variables or functions

## Lines of code
Most of the files.
## Vulnerability details
Some of the contract's constructor, variables and functions are missing natespec comments.

### Recommended Mitigation Steps
 Consider adding the missing natspec comments to the corresponding items.

# [I-02] `event` is not emitted when an important action happens on-chain

## Vulnerability details
### Proof of Concept
No `event` is emitted when an important action happens on-chain.
````solidity
17 results - 8 files
/ActivePool.sol
239:  function _rebalance(address _collateral, uint256 _amountLeavingPool) internal {

/BorrowerOperations.sol
476:  function _moveTokensAndCollateralfromAdjustment
    (
        IActivePool _activePool,
        ILUSDToken _lusdToken,
        address _borrower,
        address _collateral,
        uint _collChange,
        bool _isCollIncrease,
        uint _LUSDChange,
        bool _isDebtIncrease,
        uint _netDebtChange
    )
        internal
    {

511:  function _withdrawLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSDAmount, uint _netDebtIncrease) internal {

517:  function _repayLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSD) internal {

/LUSDToken.sol
133:  function pauseMinting() external {

141:  function unpauseMinting() external {

/StabilityPool.sol
794:  function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {

/TroveManager.sol
1189: function _removeStake(address _borrower, address _collateral) internal {

1213: function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {

1278: function _closeTrove(address _borrower, address _collateral, Status closedStatus) internal {

/CommunityIssuance.sol
61: function setAddresses 
    (
        address _oathTokenAddress, 
        address _stabilityPoolAddress
    ) 
        external 
        onlyOwner
        override 
    {

84: function issueOath() external override returns (uint issuance) {

/ReaperVaultERC4626.sol
110:  function deposit(uint256 assets, address receiver) external override returns (uint256 shares) {

154:  function mint(uint256 shares, address receiver) external override returns (uint256 assets) {

202:  function withdraw(
        uint256 assets,
        address receiver,
        address owner
    ) external override returns (uint256 shares) {

258:  function redeem(
        uint256 shares,
        address receiver,
        address owner
    ) external override returns (uint256 assets) {

/ReaperVaultV2.sol
627:  function updateTreasury(address newTreasury) external {
````

### Recommended Mitigation Steps
Add Event-Emit.

# [R-01] Some number values can be refactored with `_`

## Vulnerability details
Consider using underscores for number values to improve readability.
````solidity 
Ethos-Core/contracts/TroveManager.sol
54:	uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%

Ethos-Vault/contracts/ReaperVaultV2.sol
41:	uint256 public constant PERCENT_DIVISOR = 10000;
````
The instances above can be refactored to:
````solidity 
Ethos-Core/contracts/TroveManager.sol
54:	uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1_000 * 5; // 0.5%

Ethos-Vault/contracts/ReaperVaultV2.sol
41:	uint256 public constant PERCENT_DIVISOR = 10_000;
````

# [O-01] Floating `pragma`

## Lines of code
All files in `Ethos-Vault` mapping.
## Vulnerability details
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the `pragma` helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.    
[https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)
````solidity 
pragma solidity ^0.8.0;
````

### Recommended Mitigation Steps
Lock the `pragma` version and also consider known bugs [(https://github.com/ethereum/solidity/releases)](https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

# [O-02] Use a more recent version of Solidity

## Vulnerability details
For security, it is best practice to use the latest Solidity version.    

For the security fix list in the versions:   
[https://github.com/ethereum/solidity/blob/develop/Changelog.md](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

### Recommended Mitigation Steps
Old version of Solidity is used, newer version can be used (`0.8.17`) .

# [S-01] Project Upgrade and Stop Scenario should be

## Vulnerability details
At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.      
       
[https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol](https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol)

# [S-02] Use descriptive names for Contracts and Libraries

## Vulnerability details
This codebase will be difficult to navigate, as there are no descriptive naming conventions that specify which files should contain meaningful logic.       
       
Prefixes should be added like this by filing:      
      
Interface `I_`     
absctract contracts `Abs_`     
Libraries `Lib_`     
We recommend that you implement this or a similar agreement.