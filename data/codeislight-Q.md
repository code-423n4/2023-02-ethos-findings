# Summary
## Quality Assurance Findings List:
| Number |Issues Details|Instances|
|:--:|:-------|:--:|
|[QA-1]| internal functions used only once | 4 |
|[QA-2]| Unnecessary return statement | 64 |
|[QA-3]| Using a consistent data type, instead of rounding up to uint256 | 1 |
|[QA-4]| The recovery mode is exclusive for CCR value | 1 |
|[QA-5]| Lack of sanity check for parameters being contracts | 45 |
|[QA-6]| Explicitly define the array size limit in the parameter | 1 |

### [QA-1] internal functions used only once 

One of the main benefits of using internal function is the reusability aspect of it, but in the case being used in 1 function only, it would be better to move the logic to the function that is using it, Unless the purpose is to breakdown a long/complex logic into multiple parts.

3 instances - 3 files

instances:
- LUSDToken._requireCallerIsStabilityPool()
- LUSDToken._requireCallerIsTroveMorSP()
- LUSDToken._requireMintingNotPaused()
- ActivePool._requireCallerIsBorrowerOperationsOrDefaultPool()
- BorrowerOperations._getCollChange()
- BorrowerOperations._updateTroveFromAdjustment()
- BorrowerOperations._moveTokensAndCollateralfromAdjustment()
- BorrowerOperations._requireValidCollateralAddress()
- StabilityPool._updateG()
...

### [QA-2] Unnecessary return statement

in the ITroveManager interface, we have updateTroveRewardSnapshots() function prototype doesn't return any value, but in the implementaion in TroveManager.updateTroveRewardSnapshots(), it does have a return statmenet, despite that _updateTroveRewardSnapshots() doesn't return any value.

```solidity
    function updateTroveRewardSnapshots(address _borrower, address _collateral)
        external
        override
    {
        _requireCallerIsBorrowerOperations();
        
        return _updateTroveRewardSnapshots(_borrower, _collateral);
    }

    function _updateTroveRewardSnapshots(address _borrower, address _collateral)
        internal
    {
        rewardSnapshots[_borrower][_collateral].collAmount = L_Collateral[
            _collateral
        ];
        rewardSnapshots[_borrower][_collateral].LUSDDebt = L_LUSDDebt[
            _collateral
        ];
        emit TroveSnapshotsUpdated(
            _collateral,
            L_Collateral[_collateral],
            L_LUSDDebt[_collateral]
        );
    }
```

### [QA-3] Using a consistent data type, instead of rounding up to uint256

keeping a consistent data type flow accross the system is important, to ensure an easier compatibility and more predictable behavior. We have a Status enum, which is being rounded to uint256, it is advised to be kept as either in the form of uint8 or remains in the form of a Status enum.

```solidity
    function getTroveStatus(address _borrower, address _collateral)
        external
        view
        override
        returns (uint256)
    {
        // tbd QA return uint8 instead of uint256, to maintain a consistency since enum are uint8 based
        return uint256(Troves[_borrower][_collateral].status);
    }
```

### [QA-4] The recovery mode is exclusive for CCR value

In the case of recovery mode, it is expected to be triggered as soon as CCR is reached, so thus, it's best that the _checkRecoveryMode() is inclusive for CCR value, by checking for TCR <= _CCR instead of TCR < _CCR.

```solidity
    function _checkRecoveryMode(
        address _collateral,
        uint256 _price,
        uint256 _CCR,
        uint256 _collateralDecimals
    ) internal view returns (bool) {
        uint256 TCR = _getTCR(_collateral, _price, _collateralDecimals);
        // tbd QA got to be inclusive for CCR
        return TCR < _CCR; // use TCR <= _CCR instead
    }
```

### [QA-5] Lack of sanity check for parameters being contracts

There are many one time set functions which need to be carefully set, it is advised that those functions add checks to ensure the parameters are actually contracts to reduce the possibility of human errors. 

A good way to perform a sanity check is to check the interface selector for the targeted contract. Overall, it would drastically reduce the possibility of error to be set to the wrong contract or an EOA.

File: ReaperVaultV2.sol

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
        checkContract(_token);
        checkContract(_treasury); // in the case of a multisig

        token = IERC20Metadata(_token);
        constructionTime = block.timestamp;
        lastReport = block.timestamp;
        tvlCap = _tvlCap;
        treasury = _treasury;
        // ...
    }
```

### [QA-6] Explicitly define the array size limit in the parameter

in ReaperVaultV2 constructor, the _multisigRoles is not limited by any size, so it is better that it is bounded by an array limit to clearly define the size of addresses that would be used by the constructor.

```solidity
    constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[3] memory _multisigRoles // specify an array size of 3
    ) ERC20(string(_name), string(_symbol)) {
```