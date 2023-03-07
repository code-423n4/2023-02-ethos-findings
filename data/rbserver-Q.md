## QA REPORT

| |Issue|
|-|:-|
| [01] | `ReaperBaseStrategyv4.harvest` TRANSACTION CAN BE SANDWICH ATTACKED |
| [02] | VAULT DOES NOT SUPPORT TOKENS THAT REVERT WHEN TRANSFERRING 0 TOKEN AMOUNT |
| [03] | LACK OF SLIPPAGE PROTECTIONS FOR EOA IN FUNCTIONS LIKE `ReaperVaultERC4626.deposit`, `ReaperVaultERC4626.mint`, `ReaperVaultERC4626.withdraw`, AND `ReaperVaultERC4626.redeem` |
| [04] | `ReaperVaultV2.getPricePerFullShare` FUNCTION CAN RETURN SHARE PRICE THAT IS NOT IN 18 DECIMALS |
| [05] | PRICES REPORTED BY CHAINLINK ORACLE CAN BE CARRIED OVER AND STALE |
| [06] | THERE IS NO WAY TO DIRECTLY ADD OR REMOVE ALLOWED COLLATERAL TOKENS |
| [07] | COLLATERAL TOKENS' MCR AND CCR CANNOT BE INCREASED |
| [08] | `ReaperVaultERC4626.convertToShares` AND `ReaperVaultERC4626.convertToAssets` FUNCTIONS ARE INCONSISTENT AND `ReaperVaultERC4626.previewDeposit` AND `ReaperVaultERC4626.previewMint` FUNCTIONS ARE ALSO INCONSISTENT WHEN `ReaperVaultV2._freeFunds()` IS 0 AND `totalSupply()` IS POSITIVE |
| [09] | `ReaperVaultERC4626.previewDeposit` AND `ReaperVaultERC4626.maxMint` ARE NOT EIP-4626 COMPLIANT WHEN `ReaperVaultV2._freeFunds()` IS 0 AND `totalSupply()` IS POSITIVE |
| [10] | `ReaperBaseStrategyv4.setEmergencyExit` FUNCTION ONLY ALLOWS ACTIVATION OF `emergencyExit` FOR CORRESPONDING STRATEGY |
| [11] | UNSAFE CAST BETWEEN `uint256` AND `int256` |
| [12] | UNSAFE `decimals()` CALLS FOR TOKEN THAT DOES NOT IMPLEMENT `decimals()` |
| [13] | 3RD-PARTY CONTRACT ADDRESSES ARE HARDCODED IN `ReaperStrategyGranarySupplyOnly` CONTRACT |
| [14] | REFERRAL CODE IS HARDCODED TO 0 IN `ReaperStrategyGranarySupplyOnly` CONTRACT |
| [15] | LENGTH OF `_multisigRoles` INPUT CAN BE CHECKED IN `ReaperVaultV2` CONTRACT'S CONSTRUCTOR |
| [16] | MISSING `address(0)` CHECKS FOR CRITICAL ADDRESS INPUTS |
| [17] | VERSION `0.6.12` CAN BE USED INSTEAD OF VERSION `0.6.11` OF SOLIDITY |
| [18] | VULNERABILITIES IN `@openzeppelin/contracts 3.3.0` |
| [19] | REDUNDANT NAMED RETURNS |
| [20] | `require` CAN BE USED INSTEAD OF `assert` |
| [21] | CONSTANTS CAN BE USED INSTEAD OF MAGIC NUMBERS |
| [22] | REDUNDANT CAST |
| [23] | MISSING REASON STRINGS IN `require` STATEMENTS |
| [24] | MISSING INDEXED EVENT FIELD |
| [25] | UNDERSCORES IN NUMBER LITERALS CAN BE USED |
| [26] | SCIENTIFIC NOTATIONS WITH EXPONENTS CAN BE USED |
| [27] | COMMENTED OUT CODE CAN BE REMOVED |
| [28] | ACCIDENTALLY TYPED SPACES CAN BE REMOVED |
| [29] | FLOATING PRAGMAS |
| [30] | `uint256` CAN BE USED INSTEAD OF `uint` |
| [31] | INCOMPLETE NATSPEC COMMENTS |
| [32] | MISSING NATSPEC COMMENTS |

## [01] `ReaperBaseStrategyv4.harvest` TRANSACTION CAN BE SANDWICH ATTACKED
When calling the following `ReaperBaseStrategyv4.harvest` function that calls the `ReaperStrategyGranarySupplyOnly._harvestCore` function below, the `VeloSolidMixin._swapVelo` function is further called to use `VELO_ROUTER` for swapping the claimed reward token amount to the underlying token amount. Calling the `VeloSolidMixin._swapVelo` function then calls `router.swapExactTokensForTokens(_amount, 0, routes, address(this), block.timestamp)` in which 0 is used as the `amountOutMin` input value. This means that there is no slippage protection when performing the swaps. A malicious actor can monitor the mempool and perform a sandwich attack on the `ReaperBaseStrategyv4.harvest` transaction. As a result, the swaps can be performed very suboptimally, and the strategy experiences a loss in which the received underlying token amount after the swaps can be much less than it should be.

As a mitigation, the `ReaperBaseStrategyv4.harvest`, `ReaperStrategyGranarySupplyOnly._harvestCore`, and `VeloSolidMixin._swapVelo` functions can be updated to include an input that would be used as the `amountOutMin` input for calling the `router.swapExactTokensForTokens` function.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L109-L138
```solidity
    function harvest() external override returns (int256 roi) {
        _atLeastRole(KEEPER);
        int256 availableCapital = IVault(vault).availableCapital();
        uint256 debt = 0;
        if (availableCapital < 0) {
            debt = uint256(-availableCapital);
        }

        uint256 repayment = 0;
        if (emergencyExit) {
            ...
        } else {
            (roi, repayment) = _harvestCore(debt);
        }

        ...
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L114-L142
```solidity
    function _harvestCore(uint256 _debt) internal override returns (int256 roi, uint256 repayment) {
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

        ...
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/mixins/VeloSolidMixin.sol#L18-L42
```solidity
    function _swapVelo(
        address _from,
        address _to,
        uint256 _amount,
        address _router
    ) internal {
        ...
        router.swapExactTokensForTokens(_amount, 0, routes, address(this), block.timestamp);
    }
```

## [02] VAULT DOES NOT SUPPORT TOKENS THAT REVERT WHEN TRANSFERRING 0 TOKEN AMOUNT
Calling the `ReaperVaultV2._withdraw` function below calls the following `ReaperBaseStrategyv4.withdraw` function. When the strategy incurs a significant loss, especially in a bear market, it is possible that `amountFreed` equals 0 and `loss` equals `_amount` after executing `_liquidatePosition(_amount)`. Then, executing `IERC20Upgradeable(want).safeTransfer(vault, amountFreed)` transfers 0 tokens to the vault. However, as mentioned in https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers, some tokens would revert when transferring 0 tokens. For such tokens, executing `IERC20Upgradeable(want).safeTransfer(vault, amountFreed)` reverts. This would cause users to fail to burn shares and withdraw tokens from the vault because calling the `ReaperVaultV2._withdraw` function reverts.

As a mitigation, the `ReaperBaseStrategyv4.withdraw` function can be updated to not execute `IERC20Upgradeable(want).safeTransfer(vault, amountFreed)` when `amountFreed` is 0.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95-L103
```solidity
    function withdraw(uint256 _amount) external override returns (uint256 loss) {
        require(msg.sender == vault, "Only vault can withdraw");
        require(_amount != 0, "Amount cannot be zero");
        require(_amount <= balanceOf(), "Ammount must be less than balance");

        uint256 amountFreed = 0;
        (amountFreed, loss) = _liquidatePosition(_amount);
        IERC20Upgradeable(want).safeTransfer(vault, amountFreed);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L359-L412
```solidity
    function _withdraw(
        uint256 _shares,
        address _receiver,
        address _owner
    ) internal nonReentrant returns (uint256 value) {
        ...

        if (value > token.balanceOf(address(this))) {
            ...
            uint256 vaultBalance = 0;
            for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
                vaultBalance = token.balanceOf(address(this));
                if (value <= vaultBalance) {
                    break;
                }

                address stratAddr = withdrawalQueue[i];
                uint256 strategyBal = strategies[stratAddr].allocated;
                if (strategyBal == 0) {
                    continue;
                }

                uint256 remaining = value - vaultBalance;
                uint256 loss = IStrategy(stratAddr).withdraw(Math.min(remaining, strategyBal));
                ...
            }

            ...
        }

        ...
    }
```

## [03] LACK OF SLIPPAGE PROTECTIONS FOR EOA IN FUNCTIONS LIKE `ReaperVaultERC4626.deposit`, `ReaperVaultERC4626.mint`, `ReaperVaultERC4626.withdraw`, AND `ReaperVaultERC4626.redeem`
https://eips.ethereum.org/EIPS/eip-4626#security-considerations mentions that "if implementors intend to support EOA account access directly, they should consider adding an additional function call for `deposit`/`mint`/`withdraw`/`redeem` with the means to accommodate slippage loss or unexpected deposit/withdrawal limits, since they have no other means to revert the transaction if the exact output amount is not achieved."

As mentioned by the comment below for `DEPOSITOR`, an EOA can be granted the `DEPOSITOR` role and can directly access the vault's functions like `ReaperVaultERC4626.deposit`, `ReaperVaultERC4626.mint`, `ReaperVaultERC4626.withdraw`, and `ReaperVaultERC4626.redeem`. However, such EOA cannot specify any slippage controls when calling these functions. Because it cannot specify an acceptable minimum number of minted shares when depositing or minting, the actual number of minted shares can be less than expected. Because it cannot specify an acceptable minimum token amount when withdrawing or redeeming, the actual withdrawn token amount can be less than expected. Such EOA experiences losses in both cases.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L59-L73
```solidity
    /**
     * Reaper Roles in increasing order of privilege.
     * {DEPOSITOR} - Role conferred to EOAs/contracts that are allowed to deposit in the vault.
     ...
     */
    bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L110-L112
```solidity
    function deposit(uint256 assets, address receiver) external override returns (uint256 shares) {
        shares = _deposit(assets, receiver);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L154-L157
```solidity
    function mint(uint256 shares, address receiver) external override returns (uint256 assets) {
        assets = previewMint(shares); // previewMint rounds up so exactly "shares" should be minted and not 1 wei less
        _deposit(assets, receiver);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L202-L210
```solidity
    function withdraw(
        uint256 assets,
        address receiver,
        address owner
    ) external override returns (uint256 shares) {
        shares = previewWithdraw(assets); // previewWithdraw() rounds up so exactly "assets" are withdrawn and not 1 wei less
        if (msg.sender != owner) _spendAllowance(owner, msg.sender, shares);
        _withdraw(shares, receiver, owner);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L258-L265
```solidity
    function redeem(
        uint256 shares,
        address receiver,
        address owner
    ) external override returns (uint256 assets) {
        if (msg.sender != owner) _spendAllowance(owner, msg.sender, shares);
        assets = _withdraw(shares, receiver, owner);
    }
```

As a mitigation, since the `ReaperVaultV2._deposit` does return the actual number of minted shares, functions like `ReaperVaultERC4626.deposit` and `ReaperVaultERC4626.mint` can include an input that is the acceptable minimum number of minted shares, which can be used to check against the actual number of minted shares. Similarly, because the `ReaperVaultV2._withdraw` function does return the actual withdrawn token amount, functions like `ReaperVaultERC4626.withdraw` and `ReaperVaultERC4626.redeem` can include an input that is the acceptable minimum withdrawn token amount, which can be used to check against the actual withdrawn token amount.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L319-L338
```solidity
    function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {
        _atLeastRole(DEPOSITOR);
        ...
        if (totalSupply() == 0) {
            shares = _amount;
        } else {
            shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"
        }
        _mint(_receiver, shares);
        ...
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L359-L412
```solidity
    function _withdraw(
        uint256 _shares,
        address _receiver,
        address _owner
    ) internal nonReentrant returns (uint256 value) {
        require(_shares != 0, "Invalid amount");
        value = (_freeFunds() * _shares) / totalSupply();
        _burn(_owner, _shares);

        ...

        token.safeTransfer(_receiver, value);
        ...
    }
```

## [04] `ReaperVaultV2.getPricePerFullShare` FUNCTION CAN RETURN SHARE PRICE THAT IS NOT IN 18 DECIMALS
Comments of the following `ReaperVaultV2.getPricePerFullShare` function indicates that it should return an `uint256` with 18 decimals of how much underlying asset one vault share represents for the UIs to display. Yet, when the token's decimals is not 18, the `ReaperVaultV2.decimals` function returns a number that is not 18. In this situation, the `ReaperVaultV2.getPricePerFullShare` function does not return an `uint256` with 18 decimals while the UIs still expects the return value to have 18 decimals. This can mislead users to unfavorable user actions. For example, for a token with the decimals being less than 18, such as USDC, the current share price displayed by the UI would be smaller than it should be after scaling down with the 18 decimals; in this case, users might think that the share price is low and decide to mint some shares. Yet, the token amount deposited to the vault would be more than expected, and the user is exposed to extra risk for the extra token amount that is deposited unexpectedly because the vault's strategies can possibly incur losses before withdrawing.

As a mitigation, the `ReaperVaultV2.getPricePerFullShare` function can be updated to normalize the return value to actually have 18 decimals through multiplying by 1e18 instead of the token's decimals.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L291-L297
```solidity
    /**
     * @dev Function for various UIs to display the current value of one of our yield tokens.
     * Returns an uint256 with 18 decimals of how much underlying asset one vault share represents.
     */
    function getPricePerFullShare() public view returns (uint256) {
        return totalSupply() == 0 ? 10**decimals() : (_freeFunds() * 10**decimals()) / totalSupply();
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L650-L652
```solidity
    function decimals() public view override returns (uint8) {
        return token.decimals();
    }
```

## [05] PRICES REPORTED BY CHAINLINK ORACLE CAN BE CARRIED OVER AND STALE
According to [Chainlink's documentation](https://docs.chain.link/docs/historical-price-data), to avoid using a stale price that is being carried over, which is reported by the Chainlink oracle, "you can check `answeredInRound` against the current `roundId`. If `answeredInRound` is less than `roundId`, the answer is being carried over. If `answeredInRound` is equal to `roundId`, then the answer is fresh." However, calling the `PriceFeed._getCurrentChainlinkResponse` and `PriceFeed._getPrevChainlinkResponse` functions below do not store `answeredInRound` returned by `priceAggregator[_collateral].latestRoundData()` and `priceAggregator[_collateral].getRoundData(_currentRoundId - 1)` in the respective `ChainlinkResponse` so `answeredInRound` cannot be used for verifying if the reported prices are being carried over and stale or not.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L578-L608
```solidity
    function _getCurrentChainlinkResponse(address _collateral) internal view returns (ChainlinkResponse memory chainlinkResponse) {
        ...

        // Secondly, try to get latest price data:
        try priceAggregator[_collateral].latestRoundData() returns
        (
            uint80 roundId,
            int256 answer,
            uint256 /* startedAt */,
            uint256 timestamp,
            uint80 /* answeredInRound */
        )
        {
            // If call to Chainlink succeeds, return the response and success = true
            chainlinkResponse.roundId = roundId;
            chainlinkResponse.answer = answer;
            chainlinkResponse.timestamp = timestamp;
            chainlinkResponse.success = true;
            return chainlinkResponse;
        } catch {
            // If call to Chainlink aggregator reverts, return a zero response with success = false
            return chainlinkResponse;
        }
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L610-L637
```solidity
    function _getPrevChainlinkResponse(address _collateral, uint80 _currentRoundId, uint8 _currentDecimals) internal view returns (ChainlinkResponse memory prevChainlinkResponse) {
        ...

        // Try to get the price data from the previous round:
        try priceAggregator[_collateral].getRoundData(_currentRoundId - 1) returns
        (
            uint80 roundId,
            int256 answer,
            uint256 /* startedAt */,
            uint256 timestamp,
            uint80 /* answeredInRound */
        )
        {
            // If call to Chainlink succeeds, return the response and success = true
            prevChainlinkResponse.roundId = roundId;
            prevChainlinkResponse.answer = answer;
            prevChainlinkResponse.timestamp = timestamp;
            prevChainlinkResponse.decimals = _currentDecimals;
            prevChainlinkResponse.success = true;
            return prevChainlinkResponse;
        } catch {
            // If call to Chainlink aggregator reverts, return a zero response with success = false
            return prevChainlinkResponse;
        }
    }
```

Although `_chainlinkIsFrozen(chainlinkResponse)` is called to check if the latest reported price is not more than 4 hours old when calling the `PriceFeed.fetchPrice` function, such check does not guarantee that the reported price is not being carried over or not stale. For a reported price that is less than 4 hours old, it still can be carried over and stale if its `answeredInRound` is less than its `roundId`. Using a reported price that is being carried over and stale can have unexpected consequences, such as mistakenly considering a Trove unhealthy that allows liquidation.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L424-L426
```solidity
    function _chainlinkIsFrozen(ChainlinkResponse memory _response) internal view returns (bool) {
        return block.timestamp.sub(_response.timestamp) > TIMEOUT;
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L44
```solidity
    uint constant public TIMEOUT = 14400;
```

As a mitigation, `answeredInRound` returned by `priceAggregator[_collateral].latestRoundData()` and `priceAggregator[_collateral].getRoundData(_currentRoundId - 1)` need to be stored in the correponding `ChainlinkResponse`. Moreover, the following code need to be added in the `PriceFeed._badChainlinkResponse` function.

```solidity
        if (answeredInRound < roundId) {return true;}
```

## [06] THERE IS NO WAY TO DIRECTLY ADD OR REMOVE ALLOWED COLLATERAL TOKENS
The following `CollateralConfig.initialize` function can only be called once to directly add the allowed collateral tokens. After it is called, there is no way to add or remove the allowed collateral tokens, which is unlike the `CollateralConfig.updateCollateralRatios` function below that can still be called for multiple times by the admin to adjust the collateral token's MCR and CCR. If users want the protocol to support a new collateral token, the protocol cannot add such token, which can degrade the protocol's usability and user experience. If one of the collateral tokens becomes hacked or untrusted, the protocol cannot remove such token promptly, which can allow malicious actors to continue deposit worthless collaterals of such token to gain funds from the protocol. Although a new `CollateralConfig` contract can be deployed with a new set of collateral tokens, it can be inconvenient in which such deployment causes the protocol team extra gas cost and not timely enough for preventing a loss of the protocol's funds because the worthless collaterals of the token that becomes hacked or untrusted can still be deposited before the new `CollateralConfig` contract is used.

As a mitigation, please consider adding a function that is only callable by the admin for adding and removing the allowed collateral tokens in the `CollateralConfig` contract.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L46-L76
```solidity
    function initialize(
        address[] calldata _collaterals,
        uint256[] calldata _MCRs,
        uint256[] calldata _CCRs
    ) external override onlyOwner {
        require(!initialized, "Can only initialize once");
        ...
        
        for(uint256 i = 0; i < _collaterals.length; i++) {
            address collateral = _collaterals[i];
            ...
            collaterals.push(collateral);

            Config storage config = collateralConfig[collateral];
            config.allowed = true;
            uint256 decimals = IERC20(collateral).decimals();
            config.decimals = decimals;
            ...
            config.MCR = _MCRs[i];
            ...
            config.CCR = _CCRs[i];
            ...
        }

        initialized = true;
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L85-L100
```solidity
    function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
    ) external onlyOwner checkCollateral(_collateral) {
        Config storage config = collateralConfig[_collateral];
        ...
        config.MCR = _MCR;
        ...
        config.CCR = _CCR;
        ...
    }
```

## [07] COLLATERAL TOKENS' MCR AND CCR CANNOT BE INCREASED
The following `CollateralConfig.updateCollateralRatios` function allows the admin to decrease the collateral token's MCR and CCR but does not allow such MCR and CCR to be increased. If the collateral token has proved itself through a time when the market is turbulent, the admin can call this function to lower the MCR and CCR of this collateral token, which, for example, can make the associated Troves less likely to be liquidated. However, the collateral token's current performance cannot guarantee its future performance. It is possible that such token performs much worse when the market becomes turbulent again or such token becomes under attack after a security vulnerability is discovered in the future; in this case, such token would not be considered as reliable as before. Yet, because the admin cannot increase such token's MCR and CCR anymore, user actions that should be allowed can be disallowed or vice versa; for example, since such collateral token's MCR remains at the previous low level, the associated Troves would not be considered as liquidatable though they could be if such token's MCR could be raised.

As a mitigation, please consider updating `CollateralConfig.updateCollateralRatios` function to also allow the admin to increase the collateral token's MCR and CCR instead of only allowing the decrease of such token's MCR and CCR.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L85-L100
```solidity
    function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
    ) external onlyOwner checkCollateral(_collateral) {
        Config storage config = collateralConfig[_collateral];
        require(_MCR <= config.MCR, "Can only walk down the MCR");
        require(_CCR <= config.CCR, "Can only walk down the CCR");

        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
        config.MCR = _MCR;

        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
        config.CCR = _CCR;
        emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
    }
```

## [08] `ReaperVaultERC4626.convertToShares` AND `ReaperVaultERC4626.convertToAssets` FUNCTIONS ARE INCONSISTENT AND `ReaperVaultERC4626.previewDeposit` AND `ReaperVaultERC4626.previewMint` FUNCTIONS ARE ALSO INCONSISTENT WHEN `ReaperVaultV2._freeFunds()` IS 0 AND `totalSupply()` IS POSITIVE
When `ReaperVaultV2._freeFunds()` is 0 and `totalSupply()` is positive, calling the following `ReaperVaultERC4626.convertToShares` function with N `assets` can return N `shares`, where N is a positive number. However, calling the `ReaperVaultERC4626.convertToAssets` function with the same N `shares` will return 0 `assets`. This inconsistency between the `ReaperVaultERC4626.convertToShares` and `ReaperVaultERC4626.convertToAssets` functions can confuse the users and may encourage them to perform user actions, such as calling the `ReaperVaultV2._deposit` function, that would eventually revert and waste their gas. A similar inconsistency can also be found for the `ReaperVaultERC4626.previewDeposit` and `ReaperVaultERC4626.previewMint` functions in the same situation. As a result, the user experience is degraded.

As a mitigation, because calling the `ReaperVaultV2._deposit` function would revert when `ReaperVaultV2._freeFunds()` is 0 and `totalSupply()` is positive, please consider updating the `ReaperVaultERC4626.convertToShares` function to return 0 if `_freeFunds() == 0` is true for mirroring the `ReaperVaultERC4626.convertToAssets` function. This change can also help the `ReaperVaultERC4626.previewDeposit` function to mirror the `ReaperVaultERC4626.previewMint` function.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51-L54
```solidity
    function convertToShares(uint256 assets) public view override returns (uint256 shares) {
        if (totalSupply() == 0 || _freeFunds() == 0) return assets;
        return (assets * totalSupply()) / _freeFunds();
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66-L69
```solidity
    function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
        if (totalSupply() == 0) return shares;
        return (shares * _freeFunds()) / totalSupply();
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L96-L98
```solidity
    function previewDeposit(uint256 assets) external view override returns (uint256 shares) {
        return convertToShares(assets);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L139-L142
```solidity
    function previewMint(uint256 shares) public view override returns (uint256 assets) {
        if (totalSupply() == 0) return shares;
        assets = roundUpDiv(shares * _freeFunds(), totalSupply());
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L319-L338
```solidity
    function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {
        ...

        uint256 freeFunds = _freeFunds();
        ...
        if (totalSupply() == 0) {
            shares = _amount;
        } else {
            shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"
        }
        ...
    }
```

## [09] `ReaperVaultERC4626.previewDeposit` AND `ReaperVaultERC4626.maxMint` ARE NOT EIP-4626 COMPLIANT WHEN `ReaperVaultV2._freeFunds()` IS 0 AND `totalSupply()` IS POSITIVE
https://eips.ethereum.org/EIPS/eip-4626#previewdeposit states that the `previewDeposit` function "MUST return as close to and no more than the exact amount of Vault shares that would be minted in a deposit call in the same transaction", and https://eips.ethereum.org/EIPS/eip-4626#maxmint states that the `maxMint` function "MUST factor in both global and user-specific limits". When `ReaperVaultV2._freeFunds()` is 0 and `totalSupply()` is positive, calling the following `ReaperVaultERC4626.previewDeposit` function with N `assets` will return N `shares`, where N is a positive number, and calling the `ReaperVaultERC4626.maxMint` function below will return `tvlCap - balance()` `maxShares` in which `tvlCap - balance()` can also be positive.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L96-L98
```solidity
    function previewDeposit(uint256 assets) external view override returns (uint256 shares) {
        return convertToShares(assets);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122-L126
```solidity
    function maxMint(address) external view override returns (uint256 maxShares) {
        if (emergencyShutdown || balance() >= tvlCap) return 0;
        if (tvlCap == type(uint256).max) return type(uint256).max;
        return convertToShares(tvlCap - balance());
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51-L54
```solidity
    function convertToShares(uint256 assets) public view override returns (uint256 shares) {
        if (totalSupply() == 0 || _freeFunds() == 0) return assets;
        return (assets * totalSupply()) / _freeFunds();
    }
```

Yet, calling the `ReaperVaultERC4626.deposit` function, with a positive `assets` input, that further calls the `ReaperVaultV2._deposit` function reverts in this situation. Similarly, calling the `ReaperVaultERC4626.mint` function, with a positive `shares` input, that uses the 0 `assets` returned from the `ReaperVaultERC4626.previewMint` function call to call the `ReaperVaultV2._deposit` function also reverts in this situation.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L110-L112
```solidity
    function deposit(uint256 assets, address receiver) external override returns (uint256 shares) {
        shares = _deposit(assets, receiver);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L154-L157
```solidity
    function mint(uint256 shares, address receiver) external override returns (uint256 assets) {
        assets = previewMint(shares); // previewMint rounds up so exactly "shares" should be minted and not 1 wei less
        _deposit(assets, receiver);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L139-L142
```solidity
    function previewMint(uint256 shares) public view override returns (uint256 assets) {
        if (totalSupply() == 0) return shares;
        assets = roundUpDiv(shares * _freeFunds(), totalSupply());
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L319-L338
```solidity
    function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {
        ...
        require(_amount != 0, "Invalid amount");
        ...

        uint256 freeFunds = _freeFunds();
        ...
        if (totalSupply() == 0) {
            shares = _amount;
        } else {
            shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"
        }
        ...
    }
```

Because both of the `ReaperVaultERC4626.previewDeposit` and `ReaperVaultERC4626.maxMint` functions can return a positive number of shares while calling the `ReaperVaultERC4626.deposit` and `ReaperVaultERC4626.mint` functions revert when `ReaperVaultV2._freeFunds()` is 0 and `totalSupply()` is positive, the `ReaperVaultERC4626.previewDeposit` and `ReaperVaultERC4626.maxMint` functions are not compliant to EIP-4626. This can mislead users to perform user actions that eventually revert and also cause issues for other protocols when integrating with this protocol, which can result in disputes.

As a mitigation, the `ReaperVaultERC4626.convertToShares` function can be updated to return 0 if `_freeFunds() == 0` is true.

## [10] `ReaperBaseStrategyv4.setEmergencyExit` FUNCTION ONLY ALLOWS ACTIVATION OF `emergencyExit` FOR CORRESPONDING STRATEGY
Unlike the `ReaperVaultV2.setEmergencyShutdown` function, calling the `ReaperBaseStrategyv4.setEmergencyExit` function can only activate and cannot inactivate `emergencyExit` for the corresponding strategy. If users still value the strategy, especially after recovering from a turbulent market condition, the `GUARDIAN` cannot bring the strategy back to the non-emergency state. The same strategy has to be deployed at a different address for restoring its normal usage, which is inconvenient and costs extra gas.

As a mitigation, please consider updating the `ReaperBaseStrategyv4.setEmergencyExit` function to allow the `GUARDIAN` to also inactivate `emergencyExit` for the strategy instead of only allowing the activation of `emergencyExit`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L156-L160
```solidity
    function setEmergencyExit() external {
        _atLeastRole(GUARDIAN);
        emergencyExit = true;
        IVault(vault).revokeStrategy(address(this));
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L603-L611
```solidity
    function setEmergencyShutdown(bool _active) external {
        if (_active) {
            _atLeastRole(GUARDIAN);
        } else {
            _atLeastRole(ADMIN);
        }
        emergencyShutdown = _active;
        emit EmergencyShutdown(_active);
    }
```

## [11] UNSAFE CAST BETWEEN `uint256` AND `int256`
Functions like `ReaperVaultV2.availableCapital`, `ReaperStrategyGranarySupplyOnly._harvestCore`, and `ReaperBaseStrategyv4.harvest` below cast between `uint256` and `int256` unsafely. Such unsafe cast can lead to unexpected and incorrect accounting. For example, if `strategies[stratAddr].allocated` is increased to `uint256(type(int256).max) + 1`, which is possible since the `StrategyParams` struct's `allocated` is `uint256`, then `int256(strategies[stratAddr].allocated)` equals `type(int256).min`, which is negative; if `emergencyShutdown` is true, calling the `ReaperVaultV2.availableCapital` function will revert because executing `-int256(strategies[stratAddr].allocated)` reverts, which causes calling the `ReaperBaseStrategyv4.harvest` function to revert. In this situation, funds of the relevant strategy fail to be transferred to the vault.

As a mitigation, logics can be added to ensure that the allocated amount cannot exceed `uint256(type(int256).max)`. In addition, OpenZeppelin's [SafeCast](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol) library can be used for casting between `uint256` and `int256`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L225-L252
```solidity
    function availableCapital() public view returns (int256) {
        address stratAddr = msg.sender;
        if (totalAllocBPS == 0 || emergencyShutdown) {
            return -int256(strategies[stratAddr].allocated);
        }

        uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;
        uint256 stratCurrentAllocation = strategies[stratAddr].allocated;

        if (stratCurrentAllocation > stratMaxAllocation) {
            return -int256(stratCurrentAllocation - stratMaxAllocation);
        } else if (stratCurrentAllocation < stratMaxAllocation) {
            uint256 vaultMaxAllocation = (totalAllocBPS * balance()) / PERCENT_DIVISOR;
            uint256 vaultCurrentAllocation = totalAllocated;

            if (vaultCurrentAllocation >= vaultMaxAllocation) {
                return 0;
            }

            uint256 available = stratMaxAllocation - stratCurrentAllocation;
            available = Math.min(available, vaultMaxAllocation - vaultCurrentAllocation);
            available = Math.min(available, token.balanceOf(address(this)));

            return int256(available);
        } else {
            return 0;
        }
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L114-L142
```solidity
    function _harvestCore(uint256 _debt) internal override returns (int256 roi, uint256 repayment) {
        ...

        uint256 allocated = IVault(vault).strategies(address(this)).allocated;
        uint256 totalAssets = balanceOf();
        uint256 toFree = _debt;

        if (totalAssets > allocated) {
            uint256 profit = totalAssets - allocated;
            toFree += profit;
            roi = int256(profit);
        } else if (totalAssets < allocated) {
            roi = -int256(allocated - totalAssets);
        }

        (uint256 amountFreed, uint256 loss) = _liquidatePosition(toFree);
        repayment = MathUpgradeable.min(_debt, amountFreed);
        roi -= int256(loss);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L109-L138
```solidity
    function harvest() external override returns (int256 roi) {
        ...
        int256 availableCapital = IVault(vault).availableCapital();
        uint256 debt = 0;
        if (availableCapital < 0) {
            debt = uint256(-availableCapital);
        }

        uint256 repayment = 0;
        if (emergencyExit) {
            uint256 amountFreed = _liquidateAllPositions();
            if (amountFreed < debt) {
                roi = -int256(debt - amountFreed);
            } else if (amountFreed > debt) {
                roi = int256(amountFreed - debt);
            }

            repayment = debt;
            if (roi < 0) {
                repayment -= uint256(-roi);
            }
        } else {
            (roi, repayment) = _harvestCore(debt);
        }

        ...
    }
```

## [12] UNSAFE `decimals()` CALLS FOR TOKEN THAT DOES NOT IMPLEMENT `decimals()`
The following `CollateralConfig.initialize` and `ReaperVaultV2.decimals` functions call the respective token's `decimals()`. According to https://eips.ethereum.org/EIPS/eip-20,  `decimals()` is optional so it is possible that the relevant token does not implement `decimals()`. When this occurs, calling such token's `decimals()` reverts.

To support such token, helper functions like BoringCrypto's [`safeDecimals`](https://github.com/boringcrypto/BoringSolidity/blob/ccb743d4c3363ca37491b87c6c9b24b1f5fa25dc/contracts/libraries/BoringERC20.sol#L52-L55) can be used instead of directly calling `decimals()`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L46-L76
```solidity
    function initialize(
        address[] calldata _collaterals,
        uint256[] calldata _MCRs,
        uint256[] calldata _CCRs
    ) external override onlyOwner {
        ...
        
        for(uint256 i = 0; i < _collaterals.length; i++) {
            ...
            uint256 decimals = IERC20(collateral).decimals();
            ...
        }

        ...
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L650-L652
```solidity
    function decimals() public view override returns (uint8) {
        return token.decimals();
    }
```

## [13] 3RD-PARTY CONTRACT ADDRESSES ARE HARDCODED IN `ReaperStrategyGranarySupplyOnly` CONTRACT
As shown in the following code, the 3rd-party contract addresses are hardcoded in the `ReaperStrategyGranarySupplyOnly` contract. If the 3rd-party protocols upgrade these contracts to new addresses or if Ethos protocol wants to launch in a new chain where the hardcoded addresses do not correspond to the needed contracts (for instance, 0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6 is not Granary's lending pool address provider on Ethereum mainnet as shown by https://etherscan.io/address/0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6 and on Polygon as shown by https://polygonscan.com/address/0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6), calling functions that rely on these hardcoded addresses can revert or return values that are no longer reliable. For example, if Granary updates its lending pool address provider to another address, the hardcoded `ADDRESSES_PROVIDER` would still correspond to the old contract that can return the old lending pool address, and calling functions like `ReaperStrategyGranarySupplyOnly._deposit` below can deposit the `toReinvest` amount into the old lending pool that no longer functions properly, which can cause the deposited amount to be depreciated or lost.

As a mitigation, instead of hardcoding these 3rd-party contract addresses in the `ReaperStrategyGranarySupplyOnly` contract, functions that are only callable by parties with the reasonable privileges, such as an admin, can be added for setting and updating these 3rd-party contract addresses.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L24-L29
```solidity
    address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;
    ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =
        ILendingPoolAddressesProvider(0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6);
    IAaveProtocolDataProvider public constant DATA_PROVIDER =
        IAaveProtocolDataProvider(0x9546F673eF71Ff666ae66d01Fd6E7C6Dae5a9995);
    IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176-L182
```solidity
    function _deposit(uint256 toReinvest) internal {
        if (toReinvest != 0) {
            address lendingPoolAddress = ADDRESSES_PROVIDER.getLendingPool();
            IERC20Upgradeable(want).safeIncreaseAllowance(lendingPoolAddress, toReinvest);
            ILendingPool(lendingPoolAddress).deposit(want, toReinvest, address(this), LENDER_REFERRAL_CODE_NONE);
        }
    }
```

## [14] REFERRAL CODE IS HARDCODED TO 0 IN `ReaperStrategyGranarySupplyOnly` CONTRACT
As shown by the code below, `LENDER_REFERRAL_CODE_NONE` is hardcoded to 0 in the `ReaperStrategyGranarySupplyOnly` contract and is used when calling the `ILendingPool(lendingPoolAddress).deposit` function. If Granary has an active referral program in the future, the strategy represented by the `ReaperStrategyGranarySupplyOnly` contract cannot participate in such program and will miss the associated referral benefits because `LENDER_REFERRAL_CODE_NONE` is hardcoded to 0 and is used as the relevant referral code.

As a mitigation, instead of using the hardcoded `LENDER_REFERRAL_CODE_NONE` that is 0, a function that is only callable by parties with the reasonable privileges, such as a strategist, can be added for setting and updating the referral code for Granary's referral program in the `ReaperStrategyGranarySupplyOnly` contract.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35
```solidity
    uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176-L182
```solidity
    function _deposit(uint256 toReinvest) internal {
        if (toReinvest != 0) {
            address lendingPoolAddress = ADDRESSES_PROVIDER.getLendingPool();
            IERC20Upgradeable(want).safeIncreaseAllowance(lendingPoolAddress, toReinvest);
            ILendingPool(lendingPoolAddress).deposit(want, toReinvest, address(this), LENDER_REFERRAL_CODE_NONE);
        }
    }
```

## [15] LENGTH OF `_multisigRoles` INPUT CAN BE CHECKED IN `ReaperVaultV2` CONTRACT'S CONSTRUCTOR
In the `ReaperVaultV2` contract's constructor, the `_multisigRoles` input's length is not checked, which is unlike the `ReaperBaseStrategyv4.__ReaperBaseStrategy_init` function. Because only the first three addresses of the `_multisigRoles` input would be granted a role in the `ReaperVaultV2` contract's constructor, the length of this `_multisigRoles` input should be checked to ensure it equals 3. This can help avoid mistakes when inputting `_multisigRoles` for constructing the `ReaperVaultV2` contract.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136
```solidity
    constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) ERC20(string(_name), string(_symbol)) {
        ...
        _grantRole(DEFAULT_ADMIN_ROLE, _multisigRoles[0]);
        _grantRole(ADMIN, _multisigRoles[1]);
        _grantRole(GUARDIAN, _multisigRoles[2]);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L63-L88
```solidity
    function __ReaperBaseStrategy_init(
        address _vault,
        address _want,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) internal onlyInitializing {
        ...

        require(_multisigRoles.length == 3, "Invalid number of multisig roles");
        ...
        _grantRole(DEFAULT_ADMIN_ROLE, _multisigRoles[0]);
        _grantRole(ADMIN, _multisigRoles[1]);
        _grantRole(GUARDIAN, _multisigRoles[2]);

        ...
    }
```

## [16] MISSING `address(0)` CHECKS FOR CRITICAL ADDRESS INPUTS
( Please note that the following instances are not found in https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae#nc-1-missing-checks-for-address0-when-assigning-values-to-address-state-variables. )

To prevent unintended behaviors, critical address inputs should be checked against `address(0)`.

`address(0)` checks are missing for `_token`, `_treasury`, `_strategists[i]`, and `_multisigRoles[i]` in the following constructor. Please consider checking them.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136
```solidity
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
        ...
        treasury = _treasury;
        ...

        uint256 numStrategists = _strategists.length;
        for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
            _grantRole(STRATEGIST, _strategists[i]);
        }

        ...
        _grantRole(DEFAULT_ADMIN_ROLE, _multisigRoles[0]);
        _grantRole(ADMIN, _multisigRoles[1]);
        _grantRole(GUARDIAN, _multisigRoles[2]);
    }
```

`address(0)` checks are missing for `_vault`, `_want`, `_strategists[i]`, and `_multisigRoles[i]` in the following `__ReaperBaseStrategy_init` function. Please consider checking them.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L63-L88
```solidity
    function __ReaperBaseStrategy_init(
        address _vault,
        address _want,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) internal onlyInitializing {
        ...

        vault = _vault;
        want = _want;
        ...

        uint256 numStrategists = _strategists.length;
        for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
            _grantRole(STRATEGIST, _strategists[i]);
        }

        ...
        _grantRole(DEFAULT_ADMIN_ROLE, _multisigRoles[0]);
        _grantRole(ADMIN, _multisigRoles[1]);
        _grantRole(GUARDIAN, _multisigRoles[2]);

        ...
    }
```

## [17] VERSION `0.6.12` CAN BE USED INSTEAD OF VERSION `0.6.11` OF SOLIDITY
Comparing to version `0.6.11`, version `0.6.12` of Solidity, which is also a stable release, fixes more bugs and makes more improvements, including a fix for an error in events with indices of type static array as mentioned in https://blog.soliditylang.org/2020/07/22/Solidity-0612-release-announcement/. Please consider using version `0.6.12` instead of version `0.6.11` in the following files.

```solidity
Ethos-Core\contracts\ActivePool.sol
  3: pragma solidity 0.6.11; 

Ethos-Core\contracts\BorrowerOperations.sol
  3: pragma solidity 0.6.11; 

Ethos-Core\contracts\CollateralConfig.sol
  3: pragma solidity 0.6.11; 

Ethos-Core\contracts\LUSDToken.sol
  3: pragma solidity 0.6.11; 

Ethos-Core\contracts\StabilityPool.sol
  3: pragma solidity 0.6.11; 

Ethos-Core\contracts\TroveManager.sol
  3: pragma solidity 0.6.11; 

Ethos-Core\contracts\LQTY\CommunityIssuance.sol
  3: pragma solidity 0.6.11; 

Ethos-Core\contracts\LQTY\LQTYStaking.sol
  3: pragma solidity 0.6.11; 
```

## [18] VULNERABILITIES IN `@openzeppelin/contracts 3.3.0`
As shown in the following code in `package.json`, version 3.3.0 of `@openzeppelin/contracts` can be used. As described in https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/3.3.0, this version has vulnerabilities related to `ECDSA.recover`, `initializer`, etc. for which the protocol team should be aware of.

To reduce the potential attack surface and be more future-proofed, please consider upgrading this package to at least version 4.7.3.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/package.json#L34
```solidity
    "@openzeppelin/contracts": "^3.3.0",
```

## [19] REDUNDANT NAMED RETURNS
When a function has unused named returns and used return statement, these named returns become redundant. To improve readability and maintainability, these variables for the named returns can be removed while keeping the return statements for the following functions.

```solidity
Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol
  104: function _liquidateAllPositions() internal override returns (uint256 amountFreed) {

Ethos-Vault\contracts\ReaperVaultERC4626.sol
  29: function asset() external view override returns (address assetTokenAddress) { 
  37: function totalAssets() external view override returns (uint256 totalManagedAssets) { 
  51: function convertToShares(uint256 assets) public view override returns (uint256 shares) { 
  66: function convertToAssets(uint256 shares) public view override returns (uint256 assets) { 
  79: function maxDeposit(address) external view override returns (uint256 maxAssets) { 
  96: function previewDeposit(uint256 assets) external view override returns (uint256 shares) { 
  110: function deposit(uint256 assets, address receiver) external override returns (uint256 shares) { 
  122: function maxMint(address) external view override returns (uint256 maxShares) { 
  165: function maxWithdraw(address owner) external view override returns (uint256 maxAssets) { 
  220: function maxRedeem(address owner) external view override returns (uint256 maxShares) { 
  240: function previewRedeem(uint256 shares) external view override returns (uint256 assets) { 
```

## [20] `require` CAN BE USED INSTEAD OF `assert`
`assert` should be used for testing invariants and internal bugs while `require` should be used for checking function call returns, function inputs, valid conditions, etc. Hence, the following `assert` can be changed to `require`.

```solidity
Ethos-Core\contracts\BorrowerOperations.sol
  197: assert(vars.compositeDebt > 0); 

Ethos-Core\contracts\LUSDToken.sol
  321: assert(account != address(0));  
  329: assert(account != address(0));  
  337: assert(owner != address(0));  
  338: assert(spender != address(0));  

Ethos-Core\contracts\TroveManager.sol
  1489: assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0   
```

## [21] CONSTANTS CAN BE USED INSTEAD OF MAGIC NUMBERS
To improve readability and maintainability, a constant can be used instead of the magic number. Please consider replacing the magic numbers, such as `46`, used in the following code with constants.

```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol
  125: lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
  155: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
  181: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
```

## [22] REDUNDANT CAST
`_name` and `_symbol` are already `string` so they do not need to be casted to `string` again in the following constructor.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136
```solidity
    constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) ERC20(string(_name), string(_symbol)) {
        ...
    }
```

## [23] MISSING REASON STRINGS IN `require` STATEMENTS
( Please note that the following instances are not found in https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae#nc-2--requirerevertstatements-should-have-descriptive-reason-strings. )

When the reason strings are missing in the `require` statements, it is unclear about why certain conditions revert. Please add descriptive reason strings for the following `require` statements.

```solidity
Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol
  167: require(step[0] != address(0)); 
  168: require(step[1] != address(0)); 
```

## [24] MISSING INDEXED EVENT FIELD
( Please note that the following instance is not found in https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae#nc-5-event-is-missing-indexed-fields. )

Querying event can be optimized with index. Please consider adding `indexed` to the relevant field of the following event.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L87
```solidity
    event InCaseTokensGetStuckCalled(address token, uint256 amount);
```

## [25] UNDERSCORES IN NUMBER LITERALS CAN BE USED
Underscores in number literals can be used for improving readability and maintainability. Please consider using underscores for numbers like `999037758833783000` in the following code.

```solidity
Ethos-Core\contracts\TroveManager.sol
  53: uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;

Ethos-Vault\contracts\ReaperVaultV2.sol
  41: uint256 public constant PERCENT_DIVISOR = 10000;  
```

## [26] SCIENTIFIC NOTATIONS WITH EXPONENTS CAN BE USED
For better readability and maintainability, scientific notations with exponents can be used. Please consider using scientific notations with exponents for operations like `10**18` in the following code.

```solidity
Ethos-Vault\contracts\ReaperVaultV2.sol
  40: uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
  125: lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```

## [27] COMMENTED OUT CODE CAN BE REMOVED
Some or all of the following code are commented out. To improve readability and maintainability, please consider removing the commented out code.

```solidity
Ethos-Core\contracts\TroveManager.sol
  14: // import "./Dependencies/Ownable.sol"; 
  18: contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager { 
  19: // string constant public NAME = "TroveManager"; 
```

## [28] ACCIDENTALLY TYPED SPACES CAN BE REMOVED
For better code quality, the spaces that are accidentally typed can be removed.

A space is accidentally typed before `struct` in the following code.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L47-L64
```solidity
     struct LocalVariables_adjustTrove {
        uint256 collCCR;
        ...
        uint stake;
    }
```

A space is accidentally typed between `require` and `(_netDebt >= MIN_NET_DEBT` in the following code.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L632-L634
```solidity
    function _requireAtLeastMinNetDebt(uint _netDebt) internal pure {
        require (_netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");
    }
```

A space is accidentally typed before `function` in the following code.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L644-L646
```solidity
     function _requireSufficientLUSDBalance(ILUSDToken _lusdToken, address _borrower, uint _debtRepayment) internal view {
        require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");
    }
```

## [29] FLOATING PRAGMAS
It is a best practice to lock pragmas instead of using floating pragmas to ensure that contracts are tested and deployed with the intended compiler version. Accidentally deploying contracts with different compiler versions can lead to unexpected risks and undiscovered bugs. Please consider locking pragmas for the following files.

```solidity
Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol
  3: pragma solidity ^0.8.0;

Ethos-Vault\contracts\ReaperVaultERC4626.sol
  3: pragma solidity ^0.8.0;

Ethos-Vault\contracts\ReaperVaultV2.sol
  3: pragma solidity ^0.8.0;

Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol
  3: pragma solidity ^0.8.0;
```

## [30] `uint256` CAN BE USED INSTEAD OF `uint`
Both `uint` and `uint256` are used throughout the protocol's code. In favor of explicitness, please consider using `uint256` instead of `uint`. The following code are some examples.

```solidity
Ethos-Core\contracts\ActivePool.sol
  62: event ActivePoolLUSDDebtUpdated(address _collateral, uint _LUSDDebt);
  63: event ActivePoolCollateralBalanceUpdated(address _collateral, uint _amount);
  159: function getCollateral(address _collateral) external view override returns (uint) {
  164: function getLUSDDebt(address _collateral) external view override returns (uint) {
  171: function sendCollateral(address _collateral, address _account, uint _amount) external override {
  190: function increaseLUSDDebt(address _collateral, uint _amount) external override {
  197: function decreaseLUSDDebt(address _collateral, uint _amount) external override {
  204: function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override {

Ethos-Core\contracts\LUSDToken.sol
  75: uint internal immutable deploymentStartTime;
  266: uint amount, 
  267: uint deadline,

Ethos-Core\contracts\BorrowerOperations.sol
  51: uint price;
  52: uint collChange;
  53: uint netDebtChange;
  55: uint debt;
  56: uint coll;
  57: uint oldICR;
  58: uint newICR;
  59: uint newTCR;
  60: uint LUSDFee;
  61: uint newDebt;
  62: uint newColl;
  63: uint stake;
  70: uint price;
  71: uint LUSDFee;
  72: uint netDebt;
  73: uint compositeDebt;
  74: uint ICR;
  75: uint NICR;
  76: uint stake;
  77: uint arrayIndex;
  104: event TroveCreated(address indexed _borrower, address _collateral, uint arrayIndex);
  105: event TroveUpdated(address indexed _borrower, address _collateral, uint _debt, uint _coll, uint stake, BorrowerOperation operation);
  106: event LUSDBorrowingFeePaid(address indexed _borrower, address _collateral, uint _LUSDFee);
```

## [31] INCOMPLETE NATSPEC COMMENTS
NatSpec comments provide rich code documentation. The following functions miss the `@param` and/or `@return` comments. Please consider completing the NatSpec comments for these functions.

```solidity
Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol
  114: function _harvestCore(uint256 _debt) internal override returns (int256 roi, uint256 repayment) {    
  176: function _deposit(uint256 toReinvest) internal {    
  187: function _withdraw(uint256 _amount) internal {  
  197: function _withdrawUnderlying(uint256 _withdrawAmount) internal {  
  213: function authorizedWithdrawUnderlying(uint256 _amount) external {   
  222: function balanceOf() public view override returns (uint256) {   

Ethos-Vault\contracts\ReaperVaultV2.sol
  225: function availableCapital() public view returns (int256) {  
  278: function balance() public view returns (uint256) {  
  295: function getPricePerFullShare() public view returns (uint256) { 
  493: function report(int256 _roi, uint256 _repayment) external returns (uint256) {
  627: function updateTreasury(address newTreasury) external { 
  650: function decimals() public view override returns (uint8) {  
  659: function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {    
  673: function _hasRole(bytes32 _role, address _account) internal view override returns (bool) {  

Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol
  95: function withdraw(uint256 _amount) external override returns (uint256 loss) {   
  109: function harvest() external override returns (int256 roi) {   
  179: function clearUpgradeCooldown() public {    
  203: function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {    
  217: function _hasRole(bytes32 _role, address _account) internal view override returns (bool) {  
```

## [32] MISSING NATSPEC COMMENTS
NatSpec comments provide rich code documentation. The following functions miss NatSpec comments. Please consider adding NatSpec comments for these functions.

```solidity
Ethos-Core\contracts\BorrowerOperations.sol
  172: function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override { 
  419: function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint) {   
  524: function _requireValidCollateralAddress(address _collateral) internal view {    
  528: function _requireSufficientCollateralBalanceAndAllowance(address _user, address _collateral, uint _collAmount) internal view {    
  546: function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {  
  616: function _requireICRisAboveMCR(uint _newICR, uint256 _MCR) internal pure {   
  620: function _requireICRisAboveCCR(uint _newICR, uint256 _CCR) internal pure {   
  632: function _requireAtLeastMinNetDebt(uint _netDebt) internal pure {   
  648: function _requireValidMaxFeePercentage(uint _maxFeePercentage, bool _isRecoveryMode) internal pure {    

Ethos-Core\contracts\CollateralConfig.sol
  106: function isCollateralAllowed(address _collateral) external override view returns (bool) {   
  110: function getCollateralDecimals(   
  116: function getCollateralMCR(   
  122: function getCollateralCCR(   

Ethos-Core\contracts\LUSDToken.sol
  185: function mint(address _account, uint256 _amount) external override {  
  320: function _mint(address account, uint256 amount) internal {  
  360: function _requireCallerIsBorrowerOperations() internal view {   
  393: function _requireMintingNotPaused() internal view { 

Ethos-Core\contracts\TroveManager.sol
  1456: function getBorrowingRate() public view override returns (uint) {   
  1464: function _calcBorrowingRate(uint _baseRate) internal pure returns (uint) {   
  1471: function getBorrowingFee(uint _LUSDDebt) external view override returns (uint) {   
  1479: function _calcBorrowingFee(uint _borrowingRate, uint _LUSDDebt) internal pure returns (uint) {   
  1485: function decayBaseRateFromBorrowing() external override {   
  1500: function _updateLastFeeOpTime() internal {   
  1509: function _calcDecayedBaseRate() internal view returns (uint) {   
  1516: function _minutesPassedSinceLastFeeOp() internal view returns (uint) {   
  1522: function _requireCallerIsBorrowerOperations() internal view {   
  1544: function getTroveStatus(address _borrower, address _collateral) external view override returns (uint) { 

Ethos-Core\contracts\LQTY\LQTYStaking.sol
  187: function increaseF_LUSD(uint _LUSDFee) external override {  
  261: function _requireCallerIsBorrowerOperations() internal view {   

Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol
  74: function _adjustPosition(uint256 _debt) internal override { 
  86: function _liquidatePosition(uint256 _amountNeeded)  
  104: function _liquidateAllPositions() internal override returns (uint256 amountFreed) {
  147: function updateVeloSwapPath(    
  160: function setHarvestSteps(address[2][] calldata _newSteps) external {    
  226: function balanceOfWant() public view returns (uint256) {    
  230: function balanceOfPool() public view returns (uint256) {    

Ethos-Vault\contracts\ReaperVaultERC4626.sol
  29: function asset() external view override returns (address assetTokenAddress) {
  37: function totalAssets() external view override returns (uint256 totalManagedAssets) {
  51: function convertToShares(uint256 assets) public view override returns (uint256 shares) {
  66: function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
  79: function maxDeposit(address) external view override returns (uint256 maxAssets) {
  96: function previewDeposit(uint256 assets) external view override returns (uint256 shares) {
  110: function deposit(uint256 assets, address receiver) external override returns (uint256 shares) {
  122: function maxMint(address) external view override returns (uint256 maxShares) {
  139: function previewMint(uint256 shares) public view override returns (uint256 assets) {   
  154: function mint(uint256 shares, address receiver) external override returns (uint256 assets) {   
  165: function maxWithdraw(address owner) external view override returns (uint256 maxAssets) {
  183: function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {   
  202: function withdraw(   
  220: function maxRedeem(address owner) external view override returns (uint256 maxShares) {
  240: function previewRedeem(uint256 shares) external view override returns (uint256 assets) {
  258: function redeem(   
  269: function roundUpDiv(uint256 x, uint256 y) internal pure returns (uint256) { 

Ethos-Vault\contracts\ReaperVaultV2.sol
  319: function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {  
  359: function _withdraw( 
  603: function setEmergencyShutdown(bool _active) external {  
```