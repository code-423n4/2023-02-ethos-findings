# Ethos Reserve Low Risk and Non-Critical Issues
## Summary
| Risk      | Title | File | Instances
| ----------- | ----------- | ----------- | ----------- |
| L-01 | updateDistributionPeriod() should have a max limit | CommunityIssuance.sol | 1 |
| L-02 | Missing zero address check | CommunityIssuance.sol | 1 |
| L-03 | Initializer can be frontrunned | ReaperStrategyGranarySupplyOnly.sol | 1 |
| L-04 | Events are not emitted | All Vault Contracts | 4 |
| N-01 | Lock Pragma Version | All Vault Contracts  | 4 |
| N-02 | Outdated Pragma Version | All Contracts | 12 |
| N-03 | Use require instead of assert | - | 20 |
| N-04 | Mark Visilibility of Initializer as external | ReaperStrategyGranarySupplyOnly.sol | 1 |
| N-05 | For modern and more readable code; update import usages | All Contracts | 12 |
| N-06 | Insufficient test coverage | - | - |
| N-07 | Best practice is to prevent signature malleability | LUSDToken.sol | 1 |
| N-08 | Open TODOs | StabilityPool.sol | 1 |

## [L-01] updateDistributionPeriod() should have a max limit

#### Description:

The function updateDistributionPeriod() sets the length of the distributionPeriod. The function can only be called by the owner.

```
    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
        distributionPeriod = _newDistributionPeriod;
    }
```

The `_newDistributionPeriod` has no limit. If it is set to be an extremely large number like 100 years, then rewardsPerSecond will most probably truncate to zero because the distribution period is too large. Also, could be a cause of grief for protocol users, having to wait such a long time for their distribution.

```
        rewardPerSecond = amount.div(distributionPeriod);
```

Set a maximum possible limit, i.e. 3 years.

```
    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
+       require(_newDistributionPeriod <= 3 years, "Distribution Period is too long")
        distributionPeriod = _newDistributionPeriod;
    }
```

#### Contract: 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120-L122

## [L-02] Missing zero address check

#### Description:

The `_account` parameter in ComunityIssuance.sol has no zero address check. If the OATH token is sent to `_account ` with 0 address, OATH will be lost.

```
    function sendOath(address _account, uint _OathAmount) external override {
@--audit no 0 address check and no _OathAmount 0 check
        _requireCallerIsStabilityPool();


        OathToken.transfer(_account, _OathAmount);
    }
```

#### Contract:

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L124-L128

#### Recommendation:

```
    function sendOath(address _account, uint _OathAmount) external override {
+       checkContract(_account);
+       require(_OathAmount != 0, "Zero amount transfer")
        _requireCallerIsStabilityPool();


        OathToken.transfer(_account, _OathAmount);
    }
```

## [L-03] Initializer can be frontrunned 

#### Description: 

If the initializer is not executed in the same transaction as the constructor, a malicious user can front-run the initialize() call, forcing the contract to be redeployed.

```
    function initialize(
        address _vault,
        address[] memory _strategists,
        address[] memory _multisigRoles,
        IAToken _gWant
    ) public initializer {
        gWant = _gWant;
        want = _gWant.UNDERLYING_ASSET_ADDRESS();
        __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
        rewardClaimingTokens = [address(_gWant)];
    }
```

#### Recommendation:

Add a control that makes `initialize()` only call the Deployer Contract or EOA;

```
if (msg.sender != DEPLOYER_ADDRESS) {
            revert NotDeployer();
        }
```

## [L-04] Events are not Emitted

#### Description:
Events help non-contract tools to track changes, and events prevent users from being surprised by changes. The Vault contract does not seem to emit any sort of events. In case of a critical parameter change like deposit or withdraw, the frontend cannot display the event to the users.

For example, LQTYStaking.sol emits an event for staking and unstaking

```
    function stake(uint _LQTYamount) external override {
        _requireNonZeroAmount(_LQTYamount);


        uint currentStake = stakes[msg.sender];


        address[] memory collGainAssets;
        uint[] memory collGainAmounts;
        uint LUSDGain;
        // Grab any accumulated collateral and LUSD gains from the current stake
        if (currentStake != 0) {
            (collGainAssets, collGainAmounts) = _getPendingCollateralGain(msg.sender);
            LUSDGain = _getPendingLUSDGain(msg.sender);
        }
    
       _updateUserSnapshots(msg.sender);


        uint newStake = currentStake.add(_LQTYamount);


        // Increase user’s stake and total LQTY staked
        stakes[msg.sender] = newStake;
        totalLQTYStaked = totalLQTYStaked.add(_LQTYamount);
        emit TotalLQTYStakedUpdated(totalLQTYStaked);


        // Transfer LQTY from caller to this contract
        lqtyToken.safeTransferFrom(msg.sender, address(this), _LQTYamount);


        emit StakeChanged(msg.sender, newStake);
        emit StakingGainsWithdrawn(msg.sender, LUSDGain, collGainAssets, collGainAmounts);


         // Send accumulated LUSD and collateral gains to the caller
        if (currentStake != 0) {
            lusdToken.transfer(msg.sender, LUSDGain);
            _sendCollGainToUser(collGainAssets, collGainAmounts);
        }
    }
```

but ReaperBaseStrategyv4.sol does not emit any event for withdrawal.

```
    function withdraw(uint256 _amount) external override returns (uint256 loss) {
        require(msg.sender == vault, "Only vault can withdraw");
        require(_amount != 0, "Amount cannot be zero");
        require(_amount <= balanceOf(), "Ammount must be less than balance");


        uint256 amountFreed = 0;
        (amountFreed, loss) = _liquidatePosition(_amount);
        IERC20Upgradeable(want).safeTransfer(vault, amountFreed);
    }
```

#### Contracts:

All Vault Contracts are affected.

#### Recommendation:
Add Event-Emit

## [N-01] Unlocked pragma

#### Description:

Some of the files in scope uses `^0.8.0`, meaning that the file will compile with versions `>=0.8.0` and `<0.9.0`.  
It is considered best practice to use a locked Solidity version, thereby only allowing compilation with a specific version.  
So the instances of `^0.8.0` should be changed to `0.8.0`.

```
-    pragma solidity ^0.8.0;
+    pragma solidity 0.8.0;
```

#### Contracts:

There are 4 instances of this.  

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L3)  

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3)  

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3)  

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3)  

## [N-02] Outdated Pragma Version

#### Description:

Pragma versions are outdated. Some contracts uses 0.6.11, whereas other contracts uses 0.8.0. The latest version of solidity as of now is 0.8.19, but version 0.8.15-17 is considered to be more stable. Use a later version of solidity.

```
-     pragma solidity 0.6.11;
+     pragma solidity 0.8.16;
```

The later versions of solidity comes with additional benefits.

- Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining.
- Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.
- Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require()strings and get bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>.
- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.
- Use a solidity version of at least 0.8.12 to get string.concat() instead of abi.encodePacked(<str>,<str>)

#### Contracts:

There are 12 instances found in all the contracts in scope.

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L3)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L3)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L3)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L3)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L3)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L3)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L3)  

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3)  

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3)  

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3) 

## [N-03] Use `require` instead of `assert`

#### Description:

Assert should not be used except for tests, require should be used.

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

#### Contracts:

```
20 results - 4 files

main/Ethos-Core/contracts/BorrowerOperations.sol

128:        assert(MIN_NET_DEBT > 0);
197:        assert(vars.compositeDebt > 0);
301:        assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
331:        assert(_collWithdrawal <= vars.coll); 

main/Ethos-Core/contracts/LUSDToken.sol

312:        assert(sender != address(0));
313:        assert(recipient != address(0));
321:        assert(account != address(0));
329:        assert(account != address(0));
337:        assert(owner != address(0));
338:        assert(spender != address(0));

main/Ethos-Core/contracts/StabilityPool.sol

526:        assert(_debtToOffset <= _totalLUSDDeposits);
551:        assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
591:        assert(newP > 0);

main/Ethos-Core/contracts/TroveManager.sol

417:        assert(_LUSDInStabPool != 0);
1224:       assert(totalStakesSnapshot[_collateral] > 0);
1279:       assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
1342:       assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
1348:       assert(index <= idxLast);
1414:       assert(newBaseRate > 0); 
1489:       assert(decayedBaseRate <= DECIMAL_PRECISION);
```

### [N-04] Mark Visilibility of Initializer as `external`

#### Description:

External instead of public would give more the sense of the init(...) functions to behave like a constructor (only called on deployment, so should only be called externally)

Security point of view, it might be safer so that it cannot be called internally by accident in the child contract 

It might cost a bit less gas to use external over public 

It is possible to override a function from external to public (= "opening it up") ✅
but it is not possible to override a function from public to external (= "narrow it down"). ❌

For above reasons you can change init(...) to external

```
    function initialize(
        address _vault,
        address[] memory _strategists,
        address[] memory _multisigRoles,
        IAToken _gWant
@--audit public should be external instead
    ) public initializer {
        gWant = _gWant;
        want = _gWant.UNDERLYING_ASSET_ADDRESS();
        __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
        rewardClaimingTokens = [address(_gWant)];
    }
```

https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750

#### Contracts:

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L67](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L67)

### [N-05] For modern and more readable code; update import usages

#### Description:
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it. 
This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

In ReaperVaultV2.sol for example, the imports are written as such:

```
import "./interfaces/IERC4626Events.sol";
import "./interfaces/IStrategy.sol";
import "./libraries/ReaperMathUtils.sol";
import "./mixins/ReaperAccessControl.sol";
import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/utils/math/Math.sol";
```
#### Recommendation:
`import {contract1 , contract2} from "filename.sol";`

A good example from the ArtGobblers project;
```js
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```

#### Contracts:

All Contracts in scope.

## [N-06] Insufficient test coverage

#### Description:
The test coverage rate of the project is 93%. Given such a big codebase, testing all functions is best practice in terms of security criteria.

## [N-07] Best practice is to use signature malleability

#### Description: 

Use OpenZeppelin’s ECDSA contract rather than calling ecrecover() directly.

https://docs.openzeppelin.com/contracts/2.x/utilities

#### Contracts:

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L287

## [N-08] Open TODOs

#### Description: 

Open TODOs are usually code that is still in the brainstorming stage. In actual production, these TODOs should be deleted in code, even if the TODO is completed.

```
main/Ethos-Core/contracts/StabilityPool.sol

        /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
         * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
         * If needed could create a separate event just to report this.
         */
        uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit); // Needed only for event log
```

#### Contract: 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L335-L339

#### Recommendation: 

Use temporary TODOs as you work on a feature, but make sure to treat them before merging. Either add a link to a proper issue in your TODO, or remove it from the code.