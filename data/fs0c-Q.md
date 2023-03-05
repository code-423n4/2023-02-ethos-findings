| Letter | Name | Description |
| --- | --- | --- |
| L | Low | Low severity/Potentially risky |
| N | Non-critical | Non-risky findings |

## [L-01] No restrictions on minting to invalid recipients.

### Impact and POC

Certain transfer restrictions imposed in the LUSDToken are not properly handled during the minting.
The LUSDToken forbids any transfer to a special set of addresses specified in the `_requireValidRecipient` function:

```solidity
function _requireValidRecipient(address _recipient) internal view {
        require(
            _recipient != address(0) && 
            _recipient != address(this),
            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
        );
        require(
            !stabilityPools[_recipient] &&
            !troveManagers[_recipient] &&
            !borrowerOperations[_recipient],
            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
        );
    }
```

But these restrictions are not followed in minting. This could lead to the minting of tokens to invalid recipient addresses.

### Recommendation

Add _requireValidRecipient(account) before any state changing operations in `_mint`

## [L-02] **Incorrect TCR calculation in batchLiquidateTroves() during Recovery Mode.**

### Impact and POC

TCR is temporarily miscalculated in **batchLiquidateTroves.** 

When calculating system's entire collateral, we should also exclude the liquidated trove's surplus collateral, since liquidation closes the trove and makes the surplus collateral claimable by the trove owner. This means, this line of code should look like this:

```solidity
vars.entireSystemColl = vars.entireSystemColl.
                    sub(singleLiquidation.collToSendToSP).
                    sub(singleLiquidation.collSurplus);
```

The miscalculated entire collateral is used only to calculate the TCR and check if the system has been able to exit Recovery Mode. The miscalculation only persists temporarily, and within the`batchLiquidateTroves` transaction. Once the transaction completes the TCR and Recovery Mode will be calculated properly again. However, the bug could negatively impact the liquidation throughput and the gas efficiency gains from batching multiple liquidations in a single transaction.

The potential impact can be:

- The next trove in the sequence is liquidated while its ICR is over the real TCR: the function calculates the TCR as being slightly too high, and thus can liquidate a trove that has ICR less than the calculated TCR, but greater than the true TCR.

### Recommendation

As stated above the calculation should look like this:

```solidity
vars.entireSystemColl = vars.entireSystemColl.
                    sub(singleLiquidation.collToSendToSP).
                    sub(singleLiquidation.collSurplus);
```

## [L-03] No storage gap for upgradeable contracts.

### Impact

The contract `ReaperStrategyGranarySupplyOnly` is an UUPS upgradeable contract but doesn’t have storage gap for any future variables that might be introduced. In order to be able to add new variables to the upgradeable abstract contract without causing storage collisions, a storage gap should be added to the upgradeable abstract contract.

If no storage gap is added, when the upgradable abstract contract introduces new variables, it may override the variables in the inheriting contract.

### POC

[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L19)

### Recommendation

Consider adding storage gaps at the end of the contracts

```solidity
uint256[50] private __gap;
```

Reference: [https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps)

## [N-01] `Permit` function can be front-run in LUSDToken.

### Impact and POC

The `permit` function can be front-run in LUSDToken. In case where a user has written a contract for a series of transfer with a permit function call, an attacker can front-run permit to revert the entire transaction.

### Recommendation

Add this case in the documentation stating this case.

## [N-02] Missing require strings

Require statements should have descriptive strings to describe why the revert occurs.

```solidity
./TroveManager.sol:250:        require(msg.sender == owner);
./TroveManager.sol:551:        require(totals.totalDebtInSequence > 0);
./TroveManager.sol:716:        require(_troveArray.length != 0);
./TroveManager.sol:754:        require(totals.totalDebtInSequence > 0);
./TroveManager.sol:1450:        require(redemptionFee < _collateralDrawn);
./TroveManager.sol:1523:        require(msg.sender == borrowerOperationsAddress);
./TroveManager.sol:1527:        require(msg.sender == address(redemptionHelper));
./TroveManager.sol:1531:        require(msg.sender == borrowerOperationsAddress || msg.sender == address(redemptionHelper));
./TroveManager.sol:1535:        require(Troves[_borrower][_collateral].status == Status.active);
./ReaperStrategyGranarySupplyOnly.sol:167:            require(step[0] != address(0));
./ReaperStrategyGranarySupplyOnly.sol:168:            require(step[1] != address(0));

```

## [N-03] Use delete instead of zero assignment

Instances:

```solidity
./TroveManager.sol:1192:        Troves[_borrower][_collateral].stake = 0;
./TroveManager.sol:1285:        Troves[_borrower][_collateral].coll = 0;
./TroveManager.sol:1286:        Troves[_borrower][_collateral].debt = 0;
./TroveManager.sol:1288:        rewardSnapshots[_borrower][_collateral].collAmount = 0;
./TroveManager.sol:1289:        rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;
./TroveManager.sol:505:        singleLiquidation.debtToRedistribute = 0;
./TroveManager.sol:506:        singleLiquidation.collToRedistribute = 0;
./TroveManager.sol:387:            singleLiquidation.debtToOffset = 0;
./TroveManager.sol:388:            singleLiquidation.collToSendToSP = 0;
```

## [N-04] Unused variable defined.

In contract `ReaperBaseStrategyv4.sol` the variable `PERCENT_DIVISOR` is defined but not used anywhere. Consider removing it.

```solidity
uint256 public constant PERCENT_DIVISOR = 10_000;
```

## [N-05] Centralization Related risks on Upgrade.

In the contract `ReaperBaseStrategyv4.sol` , during the authorization process of the UUPS upgrade, the permissions are limited to the following address having the `DEFAULT_ADMIN_ROLE` set:

- `msg.sender`
- An address set as `_multisigRoles[0]`

However, no multi-signature validations are found in the codebase that is verified upon a new upgrade request. 

In Upgradeable contracts like this, the owner can upgrade the contract without the community's commitment. If an attacker compromises the account, the owner(s) can change the implementation of the contract and drain tokens from the contract.

## [N-06] Missing input validation during initialization.

During contract initialization call to `__ReaperBaseStrategy_init()` the following multisig account are given some permissions:

```solidity
_grantRole(DEFAULT_ADMIN_ROLE, _multisigRoles[0]);
_grantRole(ADMIN, _multisigRoles[1]);
_grantRole(GUARDIAN, _multisigRoles[2]);
```

However, the call to `__ReaperBaseStrategy_init()` may be reverted if the length of `_multisigRoles` is less than 3.

Consider adding a require statement to check if the length of `_multisigRoles` is less then 3, and revert if it is.