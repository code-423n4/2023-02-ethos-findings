### [NC] Outdated compiler version 

**pragma solidity 0.6.11;**

**\Ethos-Core\contracts\CollateralConfig.sol 
\Ethos-Core\contracts\BorrowerOperations.sol 
\Ethos-Core\contracts\TroveManager.sol
\Ethos-Core\contracts\ActivePool.sol
\Ethos-Core\contracts\StabilityPool.sol
\Ethos-Core\contracts\LQTY\CommunityIssuance.sol
\Ethos-Core\contracts\LQTY\LQTYStaking.sol
\Ethos-Core\contracts\LUSDToken.sol**

It's a best practice to use the latest compiler version.
The specified minimum compiler version is not up to date. Older compilers might be susceptible to some bugs. We recommend changing the solidity version pragma to the latest version to enforce the use of an up-to-date compiler.

A list of known compiler bugs and their severity can be found here: https://etherscan.io/solcbuginfo

### [NC] INTERCHANGEABLE USAGE OF UINT AND UINT256

Consider using only one approach throughout the codebase, e.g. only uint or only uint256.

### [NC] USE TIME UNITS DIRECTLY

**\Ethos-Core\contracts\TroveManager.sol**

    48: uint constant public SECONDS_IN_ONE_MINUTE = 60;

The value of 1 minute can be used directly.

### [NC] NAMED IMPORTS CAN BE USED

It’s possible to name the imports to improve code readability for all files in scope.

### [NC] EMIT KEYWORD IS MISSING BEFORE THE EVENT NAME.

**\Ethos-Core\contracts\ActivePool.sol**

    194: ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
    201: ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);

### [NC] TYPO IN REQUIRE STATEMENT IN ReaperVaultV2 CONTRACT.

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

    155: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
    181: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
    PERCENT_DIVISOR = 10000 = 100%, so 10000/5 = 2000 = 20%. In require we see: 20BPS = 0.2 %




### [NC] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USED TO IMMUTABLE RATHER THAN CONSTANT

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.
While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.
Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

**\Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol**

    bytes32 public constant KEEPER = keccak256("KEEPER"); 
    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
    bytes32 public constant ADMIN = keccak256("ADMIN");

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

    bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR"); 
    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
    bytes32 public constant ADMIN = keccak256("ADMIN");

### [NC] TEST ENVIRONMENT COMMENTS AND CODES SHOULD NOT BE IN THE MAIN VERSION

**\Ethos-Core\contracts\TroveManager.sol**

    14: // import "./Dependencies/Ownable.sol";
    18: contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager {
    // string constant public NAME = "TroveManager";
    431: // if (_ICR >= _MCR && ( _ICR >= _TCR || singleLiquidation.entireTroveDebt > _LUSDInStabPool))
    539: // if !vars.recoveryModeAtStart

### [L-01] – PRAGMA FLOATING

**\Ethos-Vault\contracts\ReaperVaultV2.sol
\Ethos-Vault\contracts\ReaperVaultERC4626.sol
\Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol
\Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol**

**pragma solidity ^0.8.0;**

Recommendation
Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case of contracts in a library or a package.

### [L-02] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY

initialize() function can be called by anybody when the contract is not initialized.

**\Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol**

    62: function initialize(
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


### [L-03] REQUIRE() SHOULD BE USED INSTEAD OF ASSERT()

Before solidity version 0.8.0, hitting an assert consumes the remainder of the transaction’s available gas rather than returning it, as require()/revert() do.
assert() should be avoided even past solidity version 0.8.0 as its documentation states that “The assert function creates an error of type Panic(uint256). … Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract that you should fix”.

**\Ethos-Core\contracts\BorrowerOperations.sol**

    128: assert(MIN_NET_DEBT > 0);
    197: assert(vars.compositeDebt > 0);
    301: assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
    331: assert(_collWithdrawal <= vars.coll);

**\Ethos-Core\contracts\StabilityPool.sol**

    526: assert(_debtToOffset <= _totalLUSDDeposits);
    551: assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
    591: assert(newP > 0);

**\Ethos-Core\contracts\TroveManager.sol**

    417: assert(_LUSDInStabPool != 0);
    1225: assert(totalStakesSnapshot[_collateral] > 0);
    1280: assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
    1343: assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
    1349: assert(index <= idxLast);
    1415: assert(newBaseRate > 0);
    1490: assert(decayedBaseRate <= DECIMAL_PRECISION);

### [L-04] USE OF ECRECOVER IS SUSCEPTIBLE TO SIGNATURE MALLEABILITY

**\Ethos-Core\contracts\LUSDToken.sol**

The ecrecover function is used to verify and execute Meta transactions. The built-in EVM precompile ecrecover is susceptible to signature malleability (because of non-unique s and v values) which could lead to replay attacks.
While this is not exploitable for replay attacks in the current implementation because of the use of nonces, this may become a vulnerability if used elsewhere.

Recommend considering using OpenZeppelin’s ECDSA library (which prevents this malleability) instead of the built-in function.

### [L-05] ADD A TIMELOCK updateStrategyFeeBPS() FUNCTION

**/Ethos-Vault/contracts/ReaperVaultV2.sol#L178-L184**

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing the risk for users. It also indicates that the project is legitimate.
However, it appears that no timelock capabilities have been utilized, which could have a significant impact on multiple users, prompting them to respond or receive advance notifications.

### [L-06] PREVENT DIVISION BY 0

On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.
These functions can be called with a 0 value in the input, this value is not checked for being bigger than 0, which that means in some scenarios can potentially trigger a division by zero.

**\Ethos-Vault\contracts\ReaperVaultV2.sol**

    347: shares = (_amount * totalSupply()) / freeFunds;
    472: uint256 bpsChange = Math.min((loss * totalAllocBPS) / totalAllocated, stratParams.allocBPS);
    499: uint256 shares = supply == 0 ? performanceFee : (performanceFee * supply) / _freeFunds();


### [L-07] MISSING EVENTS FOR INITIALIZE AND CRITICAL CHANGES

**\Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol**

    62: function initialize(
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

**E:\audits\2023-02-ethos\Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol**

    168: function setEmergencyExit() external {
        _atLeastRole(GUARDIAN);
        emergencyExit = true;
        IVault(vault).revokeStrategy(address(this));
    }
