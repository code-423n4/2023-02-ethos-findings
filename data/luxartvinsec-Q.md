# 1. Possible integer overflow in `CollateralConfig` contract

### Link : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L66

### Summary: 

The `CollateralConfig` contract contains a constant variable `MIN_ALLOWED_CCR` with the value 1.5 ether. The value of ether is assumed to be 10^18, but if ether were to be assigned a different value, there could be an integer overflow during the execution of the contract. An attacker could exploit this vulnerability to cause unexpected behavior in the contract.

### Impact: 

An integer overflow caused by a change in the value of ether could result in a breach of the security of the contract. Depending on the severity of the overflow, the attacker could be able to steal funds or cause a denial-of-service attack. In the worst case, the contract could be rendered completely unusable.

### Recommendation: 

It is recommended to replace the constant variables with the explicit values of 110% and 150%, respectively, to prevent potential integer overflow issues in the future. Additionally, it is recommended to use `SafeMath` to perform arithmetic operations in the contract, which would prevent integer overflow attacks.

### Example: 

The `MIN_ALLOWED_CCR` and `MIN_ALLOWED_MCR` variables could be replaced with the following code snippet:

```
uint256 constant public MIN_ALLOWED_MCR = 110e16; // 110%
uint256 constant public MIN_ALLOWED_CCR = 150e16; // 150%
```

The `SafeMath` library could be imported and used for arithmetic operations, as shown below:

```
import "./Dependencies/SafeMath.sol";

// ...

uint256 constant public MIN_ALLOWED_MCR = 110e16; // 110%
uint256 constant public MIN_ALLOWED_CCR = 150e16; // 150%

using SafeMath for uint256;

// ...

require(_MCRs[i].safeMul(1e18) >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
config.MCR = _MCRs[i];

require(_CCRs[i].safeMul(1e18) >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
config.CCR = _CCRs[i];
```

# 2. Potential Re-Entrancy Attack in `CollateralConfig` Contract

### Link: https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L85

### Summary: 

The `CollateralConfig` contract has a vulnerability that may allow an attacker to exploit the re-entrancy attack. The issue exists in the `updateCollateralRatios` function, which can change the `MCR` and `CCR` ratios of a specific collateral. An attacker can execute a re-entrancy attack on this function by calling an external contract with a fallback function that calls the `transferFrom` function of a vulnerable ERC20 token, such as a malicious token.

### Impact: 

An attacker can exploit this vulnerability to withdraw more funds than allowed as collateral, causing the contract to lose all of its assets. This vulnerability can lead to the loss of user funds, which can have a severe impact on the entire system.

### Recommendation: 

To mitigate this vulnerability, I recommend using the `nonReentrant` modifier in the `updateCollateralRatios` function to prevent re-entrancy attacks. Also, it is recommended to use the `SafeERC20` library in the `updateCollateralRatios` function and avoid transferring funds by calling external contracts.

### Example:

```
import "./Dependencies/ReentrancyGuard.sol";
contract CollateralConfig is ICollateralConfig, CheckContract, Ownable, ReentrancyGuard {
    using SafeERC20 for IERC20;
    ...

    function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
    ) external onlyOwner checkCollateral(_collateral) nonReentrant {
        Config storage config = collateralConfig[_collateral];
        require(_MCR <= config.MCR, "Can only walk down the MCR");
        require(_CCR <= config.CCR, "Can only walk down the CCR");

        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
        config.MCR = _MCR;

        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
        config.CCR = _CCR;
        emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
    }
}
```

Another recommendation you can use like this : 

```
bool private locked = false;

function updateCollateralRatios(
    address _collateral,
    uint256 _MCR,
    uint256 _CCR
) external onlyOwner checkCollateral(_collateral) {
    require(!locked, "Function is locked");
    locked = true;
    Config storage config = collateralConfig[_collateral];
    require(_MCR <= config.MCR, "Can only walk down the MCR");
    require(_CCR <= config.CCR, "Can only walk down the CCR");

    require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
    config.MCR = _MCR;

    require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
    config.CCR = _CCR;
    emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);

    locked = false;
}
```

# 3. Potential re-entrancy vulnerability in CommunityIssuance contract

### Link : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L84

### Summary: 

The `CommunityIssuance` contract may be vulnerable to re-entrancy attacks, which can allow an attacker to repeatedly execute a function to drain the contract's funds or cause other malicious behavior.

### Impact: 

An attacker can use a re-entrancy attack to repeatedly call the `issueOath()` or `sendOath()` functions, which can lead to unexpected results. For example, an attacker could drain the contract's funds by repeatedly calling the `issueOath()` function before the contract has a chance to update its state variables. Additionally, an attacker could execute malicious code repeatedly by exploiting the re-entrancy vulnerability.

### Recommendation: 

To mitigate the risk of re-entrancy attacks, the `CommunityIssuance` contract should use the "checks-effects-interactions" pattern. Specifically, any external call should be made after all state changes have been completed. In this case, the `OathToken.transfer()` function should be called after the `totalOATHIssued` variable has been updated, and the `lastIssuanceTimestamp` variable has been set.

### Example:

```
// @dev issues a set amount of Oath to the stability pool
function issueOath() external override returns (uint issuance) {
    _requireCallerIsStabilityPool();
    if (lastIssuanceTimestamp < lastDistributionTime) {
        uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;
        uint256 timePassed = endTimestamp.sub(lastIssuanceTimestamp);
        issuance = timePassed.mul(rewardPerSecond);
        totalOATHIssued = totalOATHIssued.add(issuance);
    }

    lastIssuanceTimestamp = block.timestamp;
    emit TotalOATHIssuedUpdated(totalOATHIssued);
    // Move the external call to the end of the function
    if (issuance > 0) {
        OathToken.transfer(stabilityPoolAddress, issuance);
    }
}
```

# 4. Potential integer underflow vulnerability in `CommunityIssuance` contract

### Link : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L84

### Summary: 

The `CommunityIssuance` contract has a potential integer underflow vulnerability when calculating the issuance of OATH tokens in the function `issueOath()`. An attacker could exploit this vulnerability to cause unexpected behavior, such as issuing more tokens than intended or causing the contract to become stuck.

### Impact: 

An attacker could exploit this vulnerability to cause the contract to behave unexpectedly, which could result in financial loss for the contract owner or users of the contract.

### Recommendation: 

I recommend adding a check to ensure that the `rewardPerSecond` variable is greater than or equal to zero before calculating the issuance amount in the `issueOath()` function. 

### Example : 

```
function issueOath() external override returns (uint issuance) {
    _requireCallerIsStabilityPool();
    if (lastIssuanceTimestamp < lastDistributionTime) {
        uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;
        uint256 timePassed = endTimestamp.sub(lastIssuanceTimestamp);
        if (rewardPerSecond > 0) {
            issuance = timePassed.mul(rewardPerSecond);
            totalOATHIssued = totalOATHIssued.add(issuance);
        }
    }

    lastIssuanceTimestamp = block.timestamp;
    emit TotalOATHIssuedUpdated(totalOATHIssued);
}
```

# 5. `LQTYStaking` contract allows users to stake LQTY tokens but does not check if the contract is paused

### Link : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L104

### Summary: 

The `LQTYStaking` contract allows users to stake LQTY tokens, but the contract does not check if it is paused. Therefore, if the contract is paused, users can still stake LQTY tokens, leading to possible losses.

### Impact: 

If the contract is paused, users can still stake LQTY tokens, leading to a potential loss of funds.

### Recommendation: 

It is recommended to add a `modifier` to check if the contract is not paused before allowing users to stake tokens. Additionally, the contract owner should have the ability to `pause` and `unpause` the contract.

### Example:

```
// --- Modifiers ---
modifier notPaused() {
    require(!paused, "Contract is paused");
    _;
}

// --- Functions ---
function stake(uint _LQTYamount) external override notPaused {
    // function body
}

function pause() external onlyOwner {
    paused = true;
}

function unpause() external onlyOwner {
    paused = false;
}
```

# 6. `stake` function does not update snapshots before sending accumulated gains

### Link: https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L118

### Summary: 

The `stake` function in the `LQTYStaking` contract allows users to stake LQTY tokens, and also withdraw any accumulated collateral and LUSD gains from their current stake. However, the function does not update the user's snapshots before sending the gains, which can lead to incorrect calculations of the collateral and LUSD gains.

### Impact: 

If the `stake` function is called by a user who has accumulated collateral or LUSD gains, but their snapshots have not been updated, the function will send incorrect gains. This could lead to incorrect accounting of the collateral and LUSD gains, and potentially cause financial losses for the staker.

### Recommendation: 

The `stake` function should update the user's snapshots before sending any accumulated gains. The following lines of code should be moved to the beginning of the function, before the check for existing stake:

```
_updateUserSnapshots(msg.sender);

address[] memory collGainAssets;
uint[] memory collGainAmounts;
uint LUSDGain;
// Grab any accumulated collateral and LUSD gains from the current stake
if (currentStake != 0) {
    (collGainAssets, collGainAmounts) = _getPendingCollateralGain(msg.sender);
    LUSDGain = _getPendingLUSDGain(msg.sender);
}
```

This will ensure that the user's snapshots are up-to-date before calculating the accumulated gains. As a best practice, the updated function should also include a `require` statement to ensure that the caller has approved the transfer of the LQTY tokens to the staking contract. For example:

```
function stake(uint _LQTYamount) external override {
    require(lqtyToken.allowance(msg.sender, address(this)) >= _LQTYamount, "LQTY allowance not set");
    _requireNonZeroAmount(_LQTYamount);

    _updateUserSnapshots(msg.sender);

    uint currentStake = stakes[msg.sender];

    address[] memory collGainAssets;
    uint[] memory collGainAmounts;
    uint LUSDGain;
    // Grab any accumulated collateral and LUSD gains from the current stake
    if (currentStake != 0) {
        (collGainAssets, collGainAmounts) = _getPendingCollateralGain(msg.sender);
        LUSDGain = _getPendingLUSDGain(msg.sender);
    }
    
    uint newStake = currentStake.add(_LQTYamount);

    // Increase user’s stake and total LQTY staked
    stakes[msg.sender] = newStake;
    totalLQTYStaked = totalLQTYStaked.add(_LQTYamount);
    emit TotalLQTYStakedUpdated(totalLQTYStaked);

    // Transfer LQTY from caller to this contract
    lqtyToken.safeTransferFrom(msg.sender, address(this), _LQTYamount);

    emit StakeChanged(msg.sender, newStake);
    emit StakingGainsWithdrawn(msg.sender, LUSDGain, collGainAssets, collGainAmounts);
}
```

If the Alice has a stake of 100 LQTY tokens, and has accumulated 10 LUSD in gains, but has not updated their snapshots, and then calls the `stake` function to add 50 LQTY tokens to their stake, the function will send 10 LUSD to Alice, but her snapshots will not be updated. As a result, when Alice tries to withdraw her accumulated collateral gains, the function will return an incorrect value, which can lead to financial losses for Alice.

# 7. Lack of input validation in `ReaperVaultERC4626` contract

### Link : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L110

### Summary: 

The `ReaperVaultERC4626` contract allows users to `deposit` and `withdraw` assets to/from the contract, but it lacks proper input validation. Specifically, the contract doesn't check if the receiver address is valid, which could lead to funds being sent to an unintended recipient or lost forever. 

### Impact: 

This vulnerability can have severe consequences as users could unintentionally lose their funds. If the receiver address is not a valid address, the funds will be lost forever. 

### Recommendation: 

To mitigate this vulnerability, input validation should be added to the `deposit` function. Specifically, the contract should check that the receiver address is a valid Ethereum address.
For example, in the `deposit` function, the following line of code can be added to check the validity of the receiver address:

```
require(receiver != address(0), "Invalid receiver address");
```

# 8. Integer Overflow Vulnerability in `ReaperVaultERC4626`

### Link : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol

### Summary: 

The contract `ReaperVaultERC4626` has an integer overflow vulnerability. The `convertToShares()` and `convertToAssets()` functions can potentially overflow if the values of the `totalSupply()` and `_freeFunds()` variables are very large. As a result, attackers can mint an unlimited number of shares, leading to a loss of funds for the contract.

### Impact: 

An attacker can exploit this vulnerability to mint an unlimited number of shares and potentially drain the contract of all its funds. If successful, this attack could cause significant financial losses to the contract and its users.

### Recommendation: 

To fix the vulnerability, the contract can be modified by using the `SafeMath` library to perform arithmetic operations on variables. The `SafeMath` library prevents integer overflow and underflow by reverting the transaction if an operation results in an overflow or underflow. 

### Example:

```
import "./ReaperVaultV2.sol";
import "./interfaces/IERC4626Functions.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

contract ReaperVaultERC4626 is ReaperVaultV2, IERC4626Functions {
    using SafeERC20 for IERC20Metadata;
    using SafeMath for uint256;

    // See comments on ReaperVaultV2's constructor
    constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}

    function convertToShares(uint256 assets) public view override returns (uint256 shares) {
        if (totalSupply() == 0 || _freeFunds() == 0) return assets;
        return assets.mul(totalSupply()).div(_freeFunds());
    }

    function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
        if (totalSupply() == 0) return shares;
        return shares.mul(_freeFunds()).div(totalSupply());
    }

    function previewMint(uint256 shares) public view override returns (uint256 assets) {
        if (totalSupply() == 0) return shares;
        assets = roundUpDiv(shares.mul(_freeFunds()), totalSupply());
    }

    function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {
        if (totalSupply() == 0 || _freeFunds() == 0) return 0;
        shares = roundUpDiv(assets.mul(totalSupply()), _freeFunds());
    }

    function roundUpDiv(uint256 x, uint256 y) internal pure returns (uint256) {
        require(y != 0, "Division by 0");

        uint256 z = x.add(y.sub(1));
        return z.div(y);
    }
}
```

# 9. Unsafe `withdrawal` function can be called by unauthorized parties

### Link : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95

### Summary: 

The `withdraw()` function of the `ReaperBaseStrategyv4` contract can be called by any address, not only the vault. This can lead to unauthorized withdrawals and theft of funds.

### Impact: 

An attacker can steal funds from the contract by calling the `withdraw()` function with an arbitrary amount. They can do this by simply creating a transaction from any address, even if it's not authorized to withdraw funds. This can result in the loss of user funds and reputation damage to the contract.

### Recommendation:

The contract should be updated to include access control on the `withdraw()` function. This can be achieved using OpenZeppelin's `AccessControl` library, which is already being used in the contract. Only the vault address should be allowed to call the function, as it's responsible for managing the contract's funds.

### Example:

```
    function withdraw(uint256 _amount) external override onlyRole(VAULT_ROLE) returns (uint256 loss) {
        require(_amount != 0, "Amount cannot be zero");
        require(_amount <= balanceOf(), "Amount must be less than balance");

        uint256 amountFreed = 0;
        (amountFreed, loss) = _liquidatePosition(_amount);
        IERC20Upgradeable(want).safeTransfer(msg.sender, amountFreed);
    }
```

In this example, the `onlyRole` modifier restricts access to the `VAULT_ROLE`. This ensures that only the vault can withdraw funds from the contract. Also, the `msg.sender` is used instead of the vault address to transfer funds back, which is safer as it ensures that the funds are transferred to the address that called the function.

# 10. Vulnerability in `ReaperStrategyGranarySupplyOnly` smart contract

### Link : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L114

### Summary: 

`ReaperStrategyGranarySupplyOnly` is a smart contract that deposits tokens on Granary to maximize yield. The vulnerability allows an attacker to steal funds from the contract by using a flash loan attack.

### Impact: 

An attacker can steal funds from the contract by using a flash loan attack. The vulnerability allows an attacker to perform multiple transactions in a single atomic transaction by taking advantage of the fact that some of the contract's logic is not atomic. The attacker can take out a flash loan, deposit the borrowed funds into the contract, and then withdraw the contract's funds to a different address before returning the borrowed funds. This would result in the contract losing its funds and the attacker gaining the stolen funds.

### Recommendation: 

The vulnerable function `_harvestCore` should be updated to ensure that it is atomic. One way to do this would be to use a reentrancy guard to prevent the function from being called multiple times within the same transaction. Additionally, it is recommended to use the `nonReentrant` modifier from the OpenZeppelin `ReentrancyGuard` library to prevent reentrancy attacks.

```
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract ReaperStrategyGranarySupplyOnly is ReaperBaseStrategyv4, VeloSolidMixin, ReentrancyGuard {
    ...

    function _harvestCore(uint256 _debt) internal override nonReentrant returns (int256 roi, uint256 repayment) {
        _claimRewards();
        uint256 numSteps = steps.length;
        for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
            address[2] storage step = steps[i];
            IERC20Upgradeable startToken = IERC20Upgradeable(step[0]);
            uint256 amount = startToken.balanceOf(address(this));
            if (amount == 0) {
                continue;
            }
            _swapVelo(step[0], step[1], amount, VELO_ROUTER);
        }

        uint256 allocated = IVault(vault).strategies(address(this)).allocated;
        ...
```	

# 11. Vulnerability in `ReaperStrategyGranarySupplyOnly` smart contract

### Link : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L74

### Summary: 

The `ReaperStrategyGranarySupplyOnly` smart contract is vulnerable to a potential reentrancy attack due to not updating the total debt when the position is adjusted.

### Impact: 

An attacker could potentially withdraw the entire want balance from the contract by exploiting the reentrancy vulnerability. The impact would depend on the amount of funds in the contract at the time of the attack.

### Recommendation: 

To prevent this reentrancy attack, it is recommended to update the total debt value before calling any external functions that could trigger further contract interactions. For example, in the `_adjustPosition` function, the `wantBalance` should be checked before updating the `totalDebt` value. The updated function should look like:

```
function _adjustPosition(uint256 _debt) internal override {
    if (emergencyExit) {
        return;
    }

    uint256 wantBalance = balanceOfWant();
    if (wantBalance > _debt) {
        uint256 toReinvest = wantBalance - _debt;
        _deposit(toReinvest);
    }

    totalDebt = ILendingPool(ADDRESSES_PROVIDER.getLendingPool()).getUserDebtCurrent(address(this), want);
}
```

This update will ensure that the `totalDebt` value is accurate before any external function calls are made.

# 12. Possible loss of funds due to unchecked arithmetic in `ReaperStrategyGranarySupplyOnly` contract

### Links : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L117
### https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L165

### Summary: 

The contract `ReaperStrategyGranarySupplyOnly` is vulnerable to an arithmetic overflow attack because it uses unchecked arithmetic. An attacker could exploit this vulnerability to steal funds from the contract.

### Impact: 

An attacker could exploit the vulnerability in the contract to steal funds from the contract. They could use a smart contract that exploits the vulnerability to drain the contract of its funds, leading to financial loss for the users of the contract.

### Recommendation: 

The use of unchecked arithmetic should be avoided as it can lead to unpredictable results. The Solidity language provides built-in functions for safe arithmetic operations such as `SafeMath`, which should be used instead. In the contract, the unchecked increment operator is used in the for loop at lines 117-165, which can be replaced with the safe `MathUpgradeable.add()` function.

### e.g:

```
for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
```
should be replaced with
```
for (uint256 i = 0; i < numSteps; i = MathUpgradeable.add(i, 1)) {
```

# 13. Wrong `import` in `TroveManager.sol`

### Link : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L14

Recommendation :  delete  `//` this wrong symbol for `import Ownable.sol`

# 14. Old solidity version
Also all contracts have old Solidity version. Please upgrade latest Solidity version : 0.8.19