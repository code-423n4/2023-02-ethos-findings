### No revert messages in TroveManager.sol

In TroveManager there are no revert messages within the require statements.

1530:   function _requireCallerIsBorrowerOperationsOrRedemptionHelper() internal view {
        require(msg.sender == borrowerOperationsAddress || msg.sender == address(redemptionHelper));
    }

Revert messages help us understand why a transaction has failed if called by parties other than,
for example borrowerOperationsAddress or redemptionHelper.

Refactored code:

```
function _requireCallerIsBorrowerOperationsOrRedemptionHelper() internal view {
    require(msg.sender == borrowerOperationsAddress || msg.sender == address(redemptionHelper), "Only borrowerOperationsAddress or redemptionHelper can call this function");
}
```

Also on lines:

1538 function _requireMoreThanOneTroveInSystem
1535 function _requireTroveIsActive
1530 function _requireCallerIsBorrowerOperationsOrRedemptionHelper
1526 function _requireCallerIsRedemptionHelper
1523 function _requireCallerIsBorrowerOperations
1450 function _calcRedemptionFee
754 function batchLiquidateTroves
716 function batchLiquidateTroves
551 function liquidateTroves
250 function setAddresses

### Remove comments from zombie code

In ReaperBaseStrategyv4, there is a comment that the DEFAULT_ADMIN_ROLE will be granted to a multisig requiring 4 signatures. 
However, the code only grants 3 multisig roles.


     * The DEFAULT_ADMIN_ROLE (in-built access control role) will be granted to a multisig requiring 4
     * signatures. This role would have upgrading capability, as well as the ability to grant any other
     * roles.
    
    ```
    function __ReaperBaseStrategy_init(
        address _vault,
        address _want,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) internal onlyInitializing {
        __UUPSUpgradeable_init();
        __AccessControlEnumerable_init();

        vault = _vault;
        want = _want;
        IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);

        uint256 numStrategists = _strategists.length;
        for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
            _grantRole(STRATEGIST, _strategists[i]);
        }

        require(_multisigRoles.length == 3, "Invalid number of multisig roles");
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(DEFAULT_ADMIN_ROLE, _multisigRoles[0]);
        _grantRole(ADMIN, _multisigRoles[1]);
        _grantRole(GUARDIAN, _multisigRoles[2]);

        clearUpgradeCooldown();
    }
```

### No Access Control permissions for revoking key roles

There are no functions surrounding access control to revoke the permissions of any of the multisig roles.
Although ReaperBaseStrategyv4 and ReaperVaultV2 inherit AccessControlEnumerable, there are no functions to revoke the permissions of the multisig roles. 
E.g introducing a revokeRole function to revoke the permissions of the multisig roles, with checks.
This can be introduced to mitigate bad actors internally or for other reasons such as unintentionally granting the wrong address the permissions.
Or the multisig roles can be removed from the contract and the permissions can be granted to a new address with the revokeRole function.


   #### This function exists in AccessControlEnumerableUpgradeable.sol

    ```
function _revokeRole(bytes32 role, address account) internal virtual override {
        super._revokeRole(role, account);
        _roleMembers[role].remove(account);
    }
```

Use this as an example in the contracts where access control/roles are introduced:

```
function revokeRole(bytes32 role, address account) external {
    require(hasRole(REVOKE_ROLE, msg.sender), "Sender must have REVOKE_ROLE");
    _revokeRole(role, account);
}
```

This can be used to revoke the permissions of the multisig roles.
You will also need to assign the REVOKE_ROLE at contract deployment to a separate address that is not a multisig role or a DEFAULT_ADMIN_ROLE. Please also ensure that none of the roles that are assigned at deployment can renounce their own roles.

### No Owner check before internal RenounceOwnership function

I appreciate this is out of scope, but i wanted to add some value here. At present, there is not an onlyOwner modifier to solicit a call by the owner in Ownable.sol, nor a check in place that the owner is calling this function. 

```   
function _renounceOwnership() internal {
        emit OwnershipTransferred(_owner, address(0));
        _owner = address(0);
    }
```

This means that the owner can accidently call this function without throwing a revert message to warn them 
beforehand. This will lead to the contract not having an owner and modification of the protocols functions
will be impossible, specifically the functions assigned to the onlyOwner. 

Recommendation: Add a require statement to check that the owner is calling this function.

I would also recommend adding a transferownership function to allow the owner to transfer ownership to another address and 
to make matters more secure, add a 2-factor step process to transfer ownership. (see bottom 2 functions).

```
 function _renounceOwnership() internal onlyOwner {
    emit OwnershipTransferred(_owner, address(0));
    _owner = address(0);
    require(_owner != msg.sender, "Ownership cannot be renounced by the owner.");
}
```

```   
function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0), "Ownable: new owner is the zero address");
    emit OwnershipTransferred(_owner, newOwner);
    _owner = newOwner;
}
```
```
function setPendingOwner(address newPendingOwner) external {
    require(msg.sender == owner, "Ownable: caller is not the owner");
    require(newPendingOwner != address(0), "Ownable: new pending owner is the zero address");
    emit newPendingOwner(newPendingOwner);
    _pendingOwner = newPendingOwner;
}
```

```
function acceptOwnership() external {
    require(msg.sender == _pendingOwner, "Ownable: caller is not the pending owner");
    emit newOwner(_pendingOwner);
    _owner = _pendingOwner;
    _pendingOwner = address(0);
}
```
### No check for valid _collateral address

412: Contracts/BorrowerOperations.sol

```
   function claimCollateral(address _collateral) external override {
        // send collateral from CollSurplus Pool to owner
        collSurplusPool.claimColl(msg.sender, _collateral);
    }
```

Add this require statement to validate _collateral address:

```    
require(isValidCollateral(_collateral), "Invalid collateral address");
```




