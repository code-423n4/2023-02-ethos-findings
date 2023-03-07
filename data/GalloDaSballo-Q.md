# Executive Summary

The following QA Report is an aggregate of smaller reports divided by contract / topic

The following criteria are used for suggested severity

L - Low Severity -> Some risk

R - Refactoring -> Suggested changes for improvements

NC - Non-Critical / Informational -> Minor suggestions


# List of all findings

* 1. [Vault](#Vault)
	* 1.1. [L - Strategy doesn't have `inCaseTokensGetStuck`](#L-StrategydoesnthaveinCaseTokensGetStuck)
	* 1.2. [L - Deposit / Withdraw use FOT compatible math, but withdrawing from strategy doesn't](#L-DepositWithdrawuseFOTcompatiblemathbutwithdrawingfromstrategydoesnt)
	* 1.3. [L - Roles for Vault and Strat are set separately](#L-RolesforVaultandStrataresetseparately)
	* 1.4. [L - MaxLoss hardcoded means the contract will revert if any loss bigger than BPS happens](#L-MaxLosshardcodedmeansthecontractwillrevertifanylossbiggerthanBPShappens)
	* 1.5. [R - Can refactor to simplify](#R-Canrefactortosimplify)
	* 1.6. [R - Track stats via Ninja](#R-TrackstatsviaNinja)
	* 1.7. [R - Track harvest health via Health Check (by Yearn)](#R-TrackharvesthealthviaHealthCheckbyYearn)
	* 1.8. [R - Consider forking Yearn.Watch](#R-ConsiderforkingYearn.Watch)
	* 1.9. [R -  PERCENT_DIVISOR is more a BPS_DIVISOR](#R-PERCENT_DIVISORismoreaBPS_DIVISOR)
	* 1.10. [R - Math for issuing shares can be turned into an internal function](#R-Mathforissuingsharescanbeturnedintoaninternalfunction)
	* 1.11. [R - License may be incompatible with source code](#R-Licensemaybeincompatiblewithsourcecode)
	* 1.12. [R - Performance Fee is Paid Out to Treasury Twice](#R-PerformanceFeeisPaidOuttoTreasuryTwice)
	* 1.13. [NC - `_addLiquidityVelo` is unused](#NC-_addLiquidityVeloisunused)
	* 1.14. [L - Steps not validated](#L-Stepsnotvalidated)
	* 1.15. [R - Equivalent to aToken.balanceOf(this)](#R-EquivalenttoaToken.balanceOfthis)
	* 1.16. [R - Amount non-zero is known](#R-Amountnon-zeroisknown)
* 2. [Redemption Helper](#RedemptionHelper)
	* 2.1. [L - Griefing redemptions for a specific collateral by redeeming the other one](#L-Griefingredemptionsforaspecificcollateralbyredeemingtheotherone)
		* 2.1.1. [Impact](#Impact)
		* 2.1.2. [POC](#POC)
		* 2.1.3. [Mitigation Steps](#MitigationSteps)
	* 2.2. [R - Could revert earlier if found is address(0)](#R-Couldrevertearlieriffoundisaddress0)
* 3. [DefaultPool](#DefaultPool)
	* 3.1. [R: cannot be front-run](#R:cannotbefront-run)
* 4. [CollateralConfig](#CollateralConfig)
	* 4.1. [L - No check for collateral existence](#L-Nocheckforcollateralexistence)
	* 4.2. [L - 150% CCR can be either too low or too high](#L-150CCRcanbeeithertoolowortoohigh)
	* 4.3. [R - No check unique](#R-Nocheckunique)
* 5. [BorrowerOperations](#BorrowerOperations)
	* 5.1. [R - Partially non-CEI conformant](#R-Partiallynon-CEIconformant)
	* 5.2. [R - Function is called withdraw, but it issues new tokens](#R-Functioniscalledwithdrawbutitissuesnewtokens)
	* 5.3. [R - Can XOR](#R-CanXOR)
* 6. [ActivePool](#ActivePool)
	* 6.1. [R - Using hardcoded value instead of a constant](#R-Usinghardcodedvalueinsteadofaconstant)
	* 6.2. [R - Approve is fine as it cannot be front-run](#R-Approveisfineasitcannotbefront-run)
	* 6.3. [NC - Old comment around borrowing fee](#NC-Oldcommentaroundborrowingfee)
* 7. [CommunityIssuance](#CommunityIssuance)
	* 7.1. [L - Right after deployment this check will be true](#L-Rightafterdeploymentthischeckwillbetrue)
	* 7.2. [R - CEI Confirmity](#R-CEIConfirmity)
	* 7.3. [NC - Comments looks out of place](#NC-Commentslooksoutofplace)
	* 7.4. [NC - Capitalization looks off](#NC-Capitalizationlooksoff)
* 8. [Price Feed](#PriceFeed)
	* 8.1. [L - The Tellor price may leak value](#L-TheTellorpricemayleakvalue)
	* 8.2. [L - Try Catch can still revert due to contract check](#L-TryCatchcanstillrevertduetocontractcheck)
	* 8.3. [L - Comment different from constants](#L-Commentdifferentfromconstants)
* 9. [NC - Gas cap at around 400 collaterals](#NC-Gascapataround400collaterals)


##  1. <a name='Vault'></a>Vault

###  1.1. <a name='L-StrategydoesnthaveinCaseTokensGetStuck'></a>L - Strategy doesn't have `inCaseTokensGetStuck`
Most Yield Farming Strategies interact with other protocols and for this reason they are subject to airdrops.

Without a `inCaseTokensGetStuck` these extra tokens would not be claimable and would be lost forever

###  1.2. <a name='L-DepositWithdrawuseFOTcompatiblemathbutwithdrawingfromstrategydoesnt'></a>L - Deposit / Withdraw use FOT compatible math, but withdrawing from strategy doesn't
While some parts of the code seem to be written to support FeeOnTransfer Tokens, other parts do not, which may cause accounting issues

###  1.3. <a name='L-RolesforVaultandStrataresetseparately'></a>L - Roles for Vault and Strat are set separately
While a vault can have multiple strategies

A strategy can only ever be used with one vault

For this reason it may be best to have a single set of Role Management Logic, in the Vault and have the strategy ask the vault rather than duplicating storage values, which may desynch


###  1.4. <a name='L-MaxLosshardcodedmeansthecontractwillrevertifanylossbiggerthanBPShappens'></a>L - MaxLoss hardcoded means the contract will revert if any loss bigger than BPS happens
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L55-L56

```solidity
    uint256 public withdrawMaxLoss = 1;

```

Due to the logic handling of losses, and because of the value being hardcoded, the Strategy may be unable to withdraw until the value is changed.

###  1.5. <a name='R-Canrefactortosimplify'></a>R - Can refactor to simplify

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/mixins/ReaperAccessControl.sol#L21-L45

```solidity
    function _atLeastRole(bytes32 _role) internal view {
        bytes32[] memory cascadingAccessRoles = _cascadingAccessRoles();
        uint256 numRoles = cascadingAccessRoles.length;
        bool specifiedRoleFound = false;
        bool senderHighestRoleFound = false;

        // {_role} must be found in the {cascadingAccessRoles} array.
        // Also, msg.sender's highest role index <= specified role index.
        for (uint256 i = 0; i < numRoles; i = i.uncheckedInc()) {
            // If the highest was not found and they have a role that is same or higher
            if (!senderHighestRoleFound && _hasRole(cascadingAccessRoles[i], msg.sender)) {
                senderHighestRoleFound = true;
            }
            // If we are at the this role part of the loop
            // E.g. we may or may have not found it here
            if (_role == cascadingAccessRoles[i]) {
                specifiedRoleFound = true;
                break;
            }
        }

        // require(senderHighestRoleFound)
        require(specifiedRoleFound && senderHighestRoleFound, "Unauthorized access");
    }

```

###  1.6. <a name='R-TrackstatsviaNinja'></a>R - Track stats via Ninja

By adding a return value via a struct of the form:
```solidity

struct TokenAmount {
    address token,
    uint256 amount
}

```

You can track the return value from harvest on a block by block basis, allowing for better monitoring

An example of the dashboard we built:
https://badger-ninja.vercel.app/vault/ethereum/0xBA485b556399123261a5F9c95d413B4f93107407

###  1.7. <a name='R-TrackharvesthealthviaHealthCheckbyYearn'></a>R - Track harvest health via Health Check (by Yearn)

Newer versions of Yearn Strategies use an [Health Check](https://github.com/yearn/yearn-vaults/blob/master/contracts/CommonHealthCheck.sol), this can help prevent an harvest when it could cause a loss.

It may be best to add this to enforce safe operations on-chain.

###  1.8. <a name='R-ConsiderforkingYearn.Watch'></a>R - Consider forking Yearn.Watch

Because the Vault is heavily inspired by YearnV2, you should be able to fork yearn.watch and have it provide automatic reporting

Site: https://yearn.watch/

###  1.9. <a name='R-PERCENT_DIVISORismoreaBPS_DIVISOR'></a>R -  PERCENT_DIVISOR is more a BPS_DIVISOR

The naming could be changed to reflect the unit used

###  1.10. <a name='R-Mathforissuingsharescanbeturnedintoaninternalfunction'></a>R - Math for issuing shares can be turned into an internal function

The logic is reused multiple times, the code can be simplified by using an internal pure function

###  1.11. <a name='R-Licensemaybeincompatiblewithsourcecode'></a>R - License may be incompatible with source code
// SPDX-License-Identifier: BUSL-1.1


Because the code is heavily inspired by YearnV2 and derivatives, a stricter license may not be valid


###  1.12. <a name='R-PerformanceFeeisPaidOuttoTreasuryTwice'></a>R - Performance Fee is Paid Out to Treasury Twice

-> On Harvest
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L463-L471

```solidity
    function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {
        uint256 performanceFee = (gain * strategies[strategy].feeBPS) / PERCENT_DIVISOR;
        if (performanceFee != 0) {
            uint256 supply = totalSupply();
            uint256 shares = supply == 0 ? performanceFee : (performanceFee * supply) / _freeFunds();
            _mint(treasury, shares);
        }
        return performanceFee;
    }
```

-> On Active Pool `_rebalance`
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L291-L294

```solidity
                vars.treasurySplit = vars.profit.mul(yieldSplitTreasury).div(10_000);
                if (vars.treasurySplit != 0) {
                    IERC20(_collateral).safeTransfer(treasuryAddress, vars.treasurySplit);
                }
```

Disclose this to end users, or consider reducing the split / adding a caller incentive for the keeper to pay for harvests

###  1.13. <a name='NC-_addLiquidityVeloisunused'></a>NC - `_addLiquidityVelo` is unused

Delete or comment as to why there's an unused function

# Strategy

###  1.14. <a name='L-Stepsnotvalidated'></a>L - Steps not validated

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L160-L171

```solidity
    function setHarvestSteps(address[2][] calldata _newSteps) external {
        _atLeastRole(ADMIN);
        delete steps;

        uint256 numSteps = _newSteps.length;
        for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
            address[2] memory step = _newSteps[i];
            require(step[0] != address(0));
            require(step[1] != address(0));
            steps.push(step);
        }
    }
```

The steps would allow to sell the want for some other token, which can result in loss of assets as well as loss of yield

A simple check would be to ensure that `step[0]` is never the `want`


###  1.15. <a name='R-EquivalenttoaToken.balanceOfthis'></a>R - Equivalent to aToken.balanceOf(this)

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L229-L236

```solidity

    function balanceOfPool() public view returns (uint256) {
        (uint256 supply, , , , , , , , ) = IAaveProtocolDataProvider(DATA_PROVIDER).getUserReserveData(
            address(want),
            address(this)
        );
        return supply;
    }
```

The main advantage to sticking to aToken is that you'll keep tracking the old implementation in case of changes

###  1.16. <a name='R-Amountnon-zeroisknown'></a>R - Amount non-zero is known

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L121-L124

```solidity
            if (amount == 0) {
                continue;
            }
            _swapVelo(step[0], step[1], amount, VELO_ROUTER);
```

Can be removed from within `_swapVelo`

##  2. <a name='RedemptionHelper'></a>Redemption Helper

###  2.1. <a name='L-Griefingredemptionsforaspecificcollateralbyredeemingtheotherone'></a>L - Griefing redemptions for a specific collateral by redeeming the other one

####  2.1.1. <a name='Impact'></a>Impact

An attacker can grief redemptions for a collateral by redeeming some other collateral, as long as they are willing to pay the fee

####  2.1.2. <a name='POC'></a>POC

Because the fee is the same for all collateral, redeeming on collateral will raise the redemption fee for others, this can be used maliciously to grief others

Because of this, it may be easier to end up getting below MCR, meaning that if the price of the collateral keeps dropping redemptions will be impossible.

Because of this:
- The protocol will receive less than them sum of X partial redemptions
- This can be used to grief the last redeemer


####  2.1.3. <a name='MitigationSteps'></a>Mitigation Steps

It may be best to have different fees per collateral

Alternatively, the redemption math could be changed to have the caller pay an increasing fee that scales with the amount being redeemed, instead of having the fee grow for the next caller

###  2.2. <a name='R-Couldrevertearlieriffoundisaddress0'></a>R - Could revert earlier if found is address(0)

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/RedemptionHelper.sol#L153

```solidity
totals.currentBorrower != address(0)
```

##  3. <a name='DefaultPool'></a>DefaultPool

###  3.1. <a name='R:cannotbefront-run'></a>R: cannot be front-run

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/DefaultPool.sol#L90-L91

```solidity
        IERC20(_collateral).safeIncreaseAllowance(activePool, _amount);

```

Increasing allowance is not different in this case

It could potentially not work with some collateral also, as you'd need to set approve(0) first


##  4. <a name='CollateralConfig'></a>CollateralConfig

###  4.1. <a name='L-Nocheckforcollateralexistence'></a>L - No check for collateral existence

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L90-L91

```solidity
        Config storage config = collateralConfig[_collateral];

```

Checking for collateral existence is equivalent to an address(0) check, and can avoid mistakes

###  4.2. <a name='L-150CCRcanbeeithertoolowortoohigh'></a>L - 150% CCR can be either too low or too high

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L25-L26

```solidity
    uint256 constant public MIN_ALLOWED_CCR = 1.5 ether; // 150%

```


Consider having custom values per collateral


###  4.3. <a name='R-Nocheckunique'></a>R - No check unique

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L53-L57

```solidity
        require(_MCRs.length == _collaterals.length, "Array lengths must match");
        require(_CCRs.length == _collaterals.length, "Array lenghts must match");
        
        for(uint256 i = 0; i < _collaterals.length; i++) {
            address collateral = _collaterals[i];
```

The lack of validation allows to input the same collateral twice, causing unintended behavior


##  5. <a name='BorrowerOperations'></a>BorrowerOperations

###  5.1. <a name='R-Partiallynon-CEIconformant'></a>R - Partially non-CEI conformant

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L231-L232

```solidity
        IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collAmount);

```

Since all other calls are from a verified contract, but collateral is not, the code could be refactored to transfer collateral directly to the active pool, as the last operation as to reduce any potential risk.

###  5.2. <a name='R-Functioniscalledwithdrawbutitissuesnewtokens'></a>R - Function is called withdraw, but it issues new tokens

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L235

```solidity
_withdrawLUSD
```

Consider renaming it

###  5.3. <a name='R-CanXOR'></a>R - Can XOR

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L296-L297

```solidity
        _requireSingularCollChange(_collTopUp, _collWithdrawal);
        _requireNonZeroAdjustment(_collTopUp, _collWithdrawal, _LUSDChange);
```

For these two functions, instead of chaning `||` and having two functions, you could use a `XOR` to combine both.

##  6. <a name='ActivePool'></a>ActivePool

###  6.1. <a name='R-Usinghardcodedvalueinsteadofaconstant'></a>R - Using hardcoded value instead of a constant

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L258

`10_000` could be turned into a `MAX_BPS` constant to improve refactoring and redability



###  6.2. <a name='R-Approveisfineasitcannotbefront-run'></a>R - Approve is fine as it cannot be front-run

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L279-L280

```solidity
            IERC20(_collateral).safeIncreaseAllowance(yieldGenerator[_collateral], uint256(vars.netAssetMovement));

```

###  6.3. <a name='NC-Oldcommentaroundborrowingfee'></a>NC - Old comment around borrowing fee

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L195-L196

```solidity
        // ICR is based on the composite debt, i.e. the requested LUSD amount + LUSD borrowing fee + LUSD gas comp.

```

There's no borrowing fee, only gas compensation, consider deleting the comment


##  7. <a name='CommunityIssuance'></a>CommunityIssuance

###  7.1. <a name='L-Rightafterdeploymentthischeckwillbetrue'></a>L - Right after deployment this check will be true

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L86-L87

```solidity
        if (lastIssuanceTimestamp < lastDistributionTime) {

```

Because `lastIssuanceTimestamp` will be 0



###  7.2. <a name='R-CEIConfirmity'></a>R - CEI Confirmity

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103-L104

```solidity
        OathToken.transferFrom(msg.sender, address(this), amount);

```

To conform to CEI, you could simply edit the function to perform the transfer at the end


###  7.3. <a name='NC-Commentslooksoutofplace'></a>NC - Comments looks out of place

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L23-L36

```solidity
   /* The issuance factor F determines the curvature of the issuance curve.
    *
    * Minutes in one year: 60*24*365 = 525600
    *
    * For 50% of remaining tokens issued each year, with minutes as time units, we have:
    * 
    * F ** 525600 = 0.5
    * 
    * Re-arranging:
    * 
    * 525600 * ln(F) = ln(0.5)
    * F = 0.5 ** (1/525600)
    * F = 0.999998681227695000 
    */
```

This comment seems to be from liquity and should be deleted

###  7.4. <a name='NC-Capitalizationlooksoff'></a>NC - Capitalization looks off

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L37-L38

```solidity
    IERC20 public OathToken;

```

Can change to: `oathToken`

##  8. <a name='PriceFeed'></a>Price Feed

### L - The Tellor fix requires constant monitoring

`TellorCaller` was modified to have a 20 minute delay on the value that is being recorded:

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/Dependencies/TellorCaller.sol#L44

```solidity
        (bytes memory data, uint256 timestamp) = getDataBefore(_queryId, block.timestamp - 20 minutes);
```

This should prevent the attack detailed in this [Liquity Disclosure](https://www.liquity.org/blog/tellor-issue-and-fix), however, do note that you'll have to constantly monitor the correctness of the feed and be able to react within 20 minutes to avoid the same attack from happening.

Because the cost of the attack is fairly inexpensive ($15 I believe), monitoring, as well as setting up infrastructure to dispute the incorrect price will be crucial.

###  8.1. <a name='L-TheTellorpricemayleakvalue'></a>L - The Tellor price may leak value

Due to the 20 minute delay, the Tellor Price may end up offering too good of a redemption price during time in which the price tanks

It may also prevent liquidations from happening correctly as the delay will make it so that some positions which should be liquidatable will be so only after the extra delay.

###  8.2. <a name='L-TryCatchcanstillrevertduetocontractcheck'></a>L - Try Catch can still revert due to contract check

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/PriceFeed.sol#L617-L636

```solidity
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
```

If the aggregator doesn't exist, Solidity will still revert in-spite of the try/catch

###  8.3. <a name='L-Commentdifferentfromconstants'></a>L - Comment different from constants

The code will check for 5% deviation, but the comment refers to 3%

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/PriceFeed.sol#L53-L54

```solidity
    uint constant public MAX_PRICE_DIFFERENCE_BETWEEN_ORACLES = 5e16; // 5%

```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/PriceFeed.sol#L496-L497

```solidity
        * Return true if the relative price difference is <= 3%: if so, we assume both oracles are probably reporting

```

# LQTY Staking

##  9. <a name='NC-Gascapataround400collaterals'></a>NC - Gas cap at around 400 collaterals
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L238-L248

```solidity
    function _sendCollGainToUser(address[] memory assets, uint[] memory amounts) internal {
        uint numCollaterals = assets.length;
        for (uint i = 0; i < numCollaterals; i++) {
            if (amounts[i] != 0) {
                address collateral = assets[i];
                emit CollateralSent(msg.sender, collateral, amounts[i]);
                IERC20(collateral).safeTransfer(msg.sender, amounts[i]);
            }
        }
    }

```

The Sponsor mentioned a DOS, but ultimately the math will break at 400+ collaterals