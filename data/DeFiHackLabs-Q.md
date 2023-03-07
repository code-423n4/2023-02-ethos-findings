# C4_ethos_QA_report

## [L1] Solidity outdate version

### Proof of concept
```js
pragma solidity 0.6.11;
```

in all Ethos contract, ex:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3

### **Recommendation:**
Update solidity 0.8.18 or higher which has the fix for some vulnerabilites.

## [L2] Susceptible to Signature Malleability

In Solidity ecrecover function directly to verify the given signatures. However, the ecrecover EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287
```js
        address recoveredAddress = ecrecover(digest, v, r, s);
        require(recoveredAddress == owner, 'LUSD: invalid signature');
```

### **Recommendation:**
Use the recover function from [OpenZeppelin's ECDSA library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol) for signature verification.

```js
function permit
    (
        address owner, 
        address spender, 
        uint amount, 
        uint deadline, 
        uint8 v, 
        bytes32 r, 
        bytes32 s
    ) 
        external 
        override 
    {
        if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {
            revert('LUSD: Invalid s value');
        }

        require(deadline >= block.timestamp, 'LUSD: expired deadline');      
        bytes32 digest = keccak256(abi.encodePacked(
            '\x19\x01',
            domainSeparator(),
            keccak256(abi.encode(
                _PERMIT_TYPEHASH,
                owner,
                spender,
                amount,
                nonces[owner]++,
                deadline
            ))
        ));
        
        address recoveredAddress = recover(digest, r, s);
        require(recoveredAddress == owner, 'LUSD: invalid signature');
        _approve(owner, spender, amount);
    }
}
```

## [L3] Lack of zero address checks on ecrecover 
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287 

### **Recommendation:**
add require(recoveredAddress != address(0), "recoveredAddress is zero address!");
 
## [L4] TresuryAddress should be multi-sig. 

From code, looks like tressury is a EOA wallet. Better to use multi-sig wallet like Gnosis safe.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L93
### **Recommendation:**
Use checkContract to check address exist.

## [L5] Owner can renounce Ownership

Use Ownable there is only one owner at a time. Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

`onlyOwner` functions;
```js
ActivePool.sol:
function setAddresses(...) external  onlyOwner
function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner
function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner
function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner 
function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner 
function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner

CollateralConfig.sol:
function initialize(...) onlyOwner 
function updateCollateralRatios() onlyOwner 

PriceFeed.sol:
function setAddresses(...) external onlyOwner
function updateChainlinkAggregator(...) external onlyOwner 
function updateTellorCaller(...) external onlyOwner 

/LQTY/CommunityIssuance.sol
function setAddresses(...) external  onlyOwner
function fund(uint amount) external onlyOwner
function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner 
```


The Openzeppelin’s Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

### **Recommendation:**

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

- You can create a new contract class called " x " that inherits from the Ownable contract but overwrites the "renounceOwnership" function to throw an exception instead of allowing the owner to renounce ownership. 
- This avoids the risk of losing control of the contract.

```js
import "@openzeppelin/contracts/access/Ownable.sol";

contract x is Ownable {
    function renounceOwnership() public override onlyOwner {
        revert("Ownership cannot be renounced");
    }
}
```

## [L6] Missing emits for declared events 

### Description:
`updateTreasury(address newTreasury)` is missing event emits which prevents the intended data from being observed easily by off-chain interfaces.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L627-L631

### Recommendation:
Add emits when Treasury address is updated.

## [L7] Missing check `_newTvlCap` isn't a zero value

### Description:
Although the `updateTvlCap()` is checked the `msg.sender` has ADMIN role, But still didn't check paramerter `_newTvlCap` is not 0

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L578-L582

### Recommendation:

Add a `require()` check.

```js
function updateTvlCap(uint256 _newTvlCap) public {
    _atLeastRole(ADMIN);
    require(_newTvlCap != 0);
    tvlCap = _newTvlCap;
    emit TvlCapUpdated(tvlCap);
}
```

## [L8] Missing zero address check in constructor

Allowing zero-addresses will lead to contract reverts and force redeployments if there are no setters for such address variables.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L124
```js
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
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L16-L24
```js
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
```

### Recommended

Add zero-address checks for all initializations/setters of all address state variables.

```js
require(_treasury != address(0), "The address cannot be zero");
treasury = _treasury;
```

To save some gas:

```js
assembly {
    if iszero(_treasury) {
        mstore(0x00, "zero address")
        revert(0x00, 0x20)
    }
}
treasury = _treasury;
```

## [L9] Missing zero address check in function

Missing _token for the zero address check.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L637-L644
```js
    function inCaseTokensGetStuck(address _token) external {
        _atLeastRole(ADMIN);
        require(_token != address(token), "!token");

        uint256 amount = IERC20Metadata(_token).balanceOf(address(this));
        IERC20Metadata(_token).safeTransfer(msg.sender, amount);
        emit InCaseTokensGetStuckCalled(_token, amount);
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L63-L73
```js
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
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L70
```js
    function initialize(
        address _vault,
        address[] memory _strategists,
        address[] memory _multisigRoles,
        IAToken _gWant
    ) public initializer {
        gWant = _gWant;
        want = _gWant.UNDERLYING_ASSET_ADDRESS();
        __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
```
### Recommended

Add zero-address checks for all function of all address state variables.

```js
function inCaseTokensGetStuck(address _token) external {
    _atLeastRole(ADMIN);
    require(_token != address(0), "The address cannot be zero");
    require(_token != address(token), "!token");

    uint256 amount = IERC20Metadata(_token).balanceOf(address(this));
    IERC20Metadata(_token).safeTransfer(msg.sender, amount);
    emit InCaseTokensGetStuckCalled(_token, amount);
}

```
To save some gas:
```js
function inCaseTokensGetStuck(address _token) external {
    _atLeastRole(ADMIN);
     assembly {
        if iszero(_token) {
            mstore(0x00, "zero address")
            revert(0x00, 0x20)
        }
    }
    require(_token != address(token), "!token");

    uint256 amount = IERC20Metadata(_token).balanceOf(address(this));
    IERC20Metadata(_token).safeTransfer(msg.sender, amount);
    emit InCaseTokensGetStuckCalled(_token, amount);
}
```

## [L10] making rewardClaimingTokens immutable 

Since it carries over unclaimed rewards from the previous reward program. It would be safer to keep the reward token immutable as a safeguard against violations of this condition.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L41
```js
address[] public rewardClaimingTokens;
```

### Recommended 

The easiest mitigation would be to pass this variable as immutable

```js
address[] public immutable rewardClaimingTokens;
```

## [L11] Missing visibility setting
The default function visibility level in contracts is public, in interfaces - external. Explicitly define function visibility to prevent confusion.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L28-L32

```solidity
    address stabilityPoolAddress;
    address gasPoolAddress;
    ICollSurplusPool collSurplusPool;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L31
```solidity
    address gasPoolAddress;
```

### Recommendation
It is recommended to make a conscious decision on which visibility type is appropriate for a function. This can dramatically reduce the attack surface of a contract system.


---

## [N1] Lock pragmas to Specific Compiler Version 

### Description:
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
https://swcregistry.io/docs/SWC-103

Affected instance:
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3 
```js
1 result - 1 file
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
1: // SPDX-License-Identifier: BUSL1.1
2: 
3: pragma solidity ^0.8.0;

```

### **Recommendation:**

Lock pragmas to specific compiler version.
[solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)


## [N2] Incorrect error message

Error message miss caller is redemptionHelper.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L257
```js

    function _requireCallerIsTroveManagerOrActivePool() internal view {
        address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());
        require(
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == activePoolAddress,
            "LQTYStaking: caller is not TroveM or ActivePool" // @audit error message miss caller is redemptionHelper.
        );
```

## [N3] Function doesn't initialize return value

https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/LUSDToken.sol#L298

```js
function _chainID() private pure returns (uint256 chainID) {
    assembly {
        chainID := chainid()
    }
}
```

### Recommendation:
```js
function _chainID() private pure returns (uint256 chainID) {
    chainID = 0; 
    assembly {
        chainID := chainid()
    }
}

```

## [N4] Use `_requireNonZeroAmount` instead of `if (_amount !=0)` 

Origin source code:

```js
if (_amount !=0) {_requireNoUnderCollateralizedTroves();}
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L368

### This could increases code readability:

```js
_requireNonZeroAmount(_amount);
_requireNoUnderCollateralizedTroves();
```

### Explanation
- You could use the _requireNonZeroAmount function, which is a function defined in the contract.
- This is used to check if _amount > 0.

## [N5] Without clear error messages 

Lots of require check without error messages. some examples below:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L716
```js
require(_troveArray.length != 0);
```

### Fix:
```js
require(_troveArray.length != 0, "TroveManager: Calldata address array must not be empty");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754

```js
 require(totals.totalDebtInSequence > 0);
```
### Fix:
```js
require(totals.totalDebtInSequence > 0, "TroveManager: nothing to liquidate");
```


## [N6] Open TODOs 
Use temporary TODOs as you work on a feature, but make sure to treat them before merging. Either add a link to a proper issue in your TODO, or remove it from the code
### Context:
[StabilityPool.sol#L335-L338](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L335-L338)
```
     File: Ethos-Core/contracts/StabilityPool.sol
    335:         /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
    336:          * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
    337:          * If needed could create a separate event just to report this.
    338:          */
```
[StabilityPool.sol#L380-L383](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L380-L383)
```
    File: Ethos-Core/contracts/StabilityPool.sol
    380:         /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
    381:          * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
    382:          * If needed could create a separate event just to report this.
    383:          */
```
## [N7] Use `require` instead of `assert` 
### Context :- 
[BorrowerOperations.sol#L301](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L301)
```
        File: Ethos-Core/contracts/BorrowerOperations.sol
        301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0)); 
```
[LUSDToken.sol#L321](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L321)
```
    File: Ethos-Core/contracts/LUSDToken.sol
    321:         assert(account != address(0));
```
[LUSDToken.sol#L312-L313](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312-L313)
```
    File: Ethos-Core/contracts/LUSDToken.sol
    312:         assert(sender != address(0));
    313:         assert(recipient != address(0));

```
    
### Description
Assert should not be used except for tests, require should be used.

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

assert() and require();
The big difference between the two is that the assert()function when false, uses up all the remaining gas and reverts all the changes made.
Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay.
This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states “The Assert function generates an error of type Panic(uint256).Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. There’s a mistake”.

## [N8] use underscore for number literals 
[TroveManager.sol#L53](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L53)
```
    File: Ethos-Core/contracts/TroveManager.sol
    53:     uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;
```
[ReaperVaultV2.sol#L41](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L41)
```
    File: Ethos-Vault/contracts/ReaperVaultV2.sol
    41:     uint256 public constant PERCENT_DIVISOR = 10000;
```
### Description
There is occasions where certain number have been hardcoded, either in variable or in the code itself. Large numbers can become hard to read.

### Recommendation
Consider using underscores for number literals to improve its readability.like this
```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

- 41:   uint256 public constant PERCENT_DIVISOR = 10000;
+       uint256 public constant PERCENT_DIVISOR = 10_000
```


## [S-1] Remove unecessary comments 

here the commented out portions serves no purpose at all,consider removing these
[TroveManager.sol#L14](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L14)
```
File: Ethos-Core/contracts/TroveManager.sol
14: // import "./Dependencies/Ownable.sol";

```
[TroveManager.sol#L18-L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L18-L19)
```
    File: Ethos-Core/contracts/TroveManager.sol
    18: contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager {
    19:     // string constant public NAME = "TroveManager";
```