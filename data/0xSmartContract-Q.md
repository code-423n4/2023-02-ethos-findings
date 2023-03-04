## Summary
### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]|Use ERC-5143: Slippage Protection for Tokenized Vault| 1 |
|[L-02]|Head Overflow Bug in Calldata Tuple ABI-Reencoding| 1 |
|[L-03]|`uint` type argument was use instead `uint256` for `permit`function , this will cause the function selector to be miscalculated | 1 |
|[L-04]|First ERC4626 deposit exploit can break share calculation |1 |
|[L-05]|initialize() function can be called by anybody| 1 |
|[L-06]|Insufficient coverage|All Contracts  |
|[L-07]|Missing Event for initialize| 1 |
|[L-08]|Project Upgrade and Stop Scenario should be| 1 |
|[L-09]|Loss of precision due to rounding in ` amount / uint256(rewardsPerSecond) `|1 |
|[L-10]|Use Fuzzing Test for complicated code bases | |
|[L-11]|Update codes to avoid Compile Errors|11 |
|[L-12]|Add to Event-Emit for critical functions|3 |
|[L-13]|Use `uint256` instead `uint`|477 |
|[L-14]|Signature Malleability of EVM's ecrecover()|1 |
|[L-15]|`sendCollateral` event is missing parameters|1 |
|[L-16]|Lack of Input Validation|1 |
|[L-17]|Prevent division by `0`|1 |
|[L-18]|Project has NPM Dependency which uses a vulnerable version : `@openzeppelin`|1 |
|[L-19]|Keccak Constant values should used to immutable rather than constant|8 |
|[L-20]|Due to the novelty of the ERC4626 standard, it is safer to use as upgradeable| |
|[L-21]|In the `setAddresses` function, there is no return of incorrect address identification|1 |

Total 21 issues


### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01] |Omissions in Events|2|
| [N-02] |NatSpec comments should be increased in contracts |All Contracts|
| [N-03] |`Function writing` that does not comply with the `Solidity Style Guide`| All Contracts |
| [N-04] |Include return parameters in NatSpec comments| All Constracs |
| [N-05] |Tokens accidentally sent to the contract cannot be recovered|1 |
| [N-06] |Repeated validation logic can be converted into a function modifier | 13 |
| [N-07] |Use a more recent version of Solidity| 14 |
| [N-08] |For functions, follow Solidity standard naming conventions (internal function style rule)| 4 |
| [N-09] |Floating pragma| 4 |
| [N-10] |Use descriptive names for Contracts and Libraries| All Contracts  |
| [N-11] |Use SMTChecker|All Contracts |
| [N-12] |Add NatSpec Mapping comment | 13 |
| [N-13] |Showing the actual values of numbers in NatSpec comments makes checking and reading code easier | 2 |
| [N-14] |Lines are too long | 16 |
| [N-15] |Constants on the left are better| 36 |
| [N-16] |`constants` should be defined rather than using magic numbers | 3 |
| [N-17] |Missing timelock for critical parameter change | 1 |
| [N-18] |Use the `delete` keyword instead of assigning a value of `0` | 17 |
| [N-19] |Not using the named return variables anywhere in the function is confusing | 2 |
| [N-20] |Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked | 2 |
| [N-21] |According to the syntax rules, use `mapping ()` instead of `mapping()` using spaces as keyword | 2 |
| [N-22] |For modern and more readable code; update import usages | 110 |
| [N-23] |Use `require` instead of `assert` | 20 |
| [N-24] |Implement some type of version counter that will be incremented automatically for contract upgrades | 1 |
| [N-25] |Use a single file for all system-wide constants | 18 |
| [N-26] |Assembly Codes Specific – Should Have Comments | 1 |
| [N-27] |Highest risk must be specified in NatSpec comments and documentation| 1 |
| [N-28] |Include proxy contracts to Audit |  |

Total 28 issues


### Suggestions
| Number | Suggestion Details |
|:--:|:-------|
| [S-01] |Generate perfect code headers every time |

Total 2 suggestions


### [L-01] Use ERC-5143: Slippage Protection for Tokenized Vault

Project use ERC-4626 standart

EIP-4626 is vulnerable to the so-called inflation attacks. This attack results from the possibility to manipulate the exchange rate and front run a victim’s deposit when the vault has low liquidity volume.


**Proposed mitigation**


The following standard extends the EIP-4626 Tokenized Vault standard with functions dedicated to the safe interaction between EOAs and the vault when price is subject to slippage.

https://eips.ethereum.org/EIPS/eip-5143

### [L-02] Head Overflow Bug in Calldata Tuple ABI-Reencoding

On July 5, 2022, Chance Hudson (@vimwitch) from the Ethereum Foundation discovered a bug in the Solidity code generator.
The earliest affected version of the compiler is 0.5.8, which introduced ABI-reencoding of calldata arrays and structs. Solidity version 0.8.16, released on August 08, 2022, provides a fix.

 Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.


```solidity
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol:
  159       */
  160:     function setHarvestSteps(address[2][] calldata _newSteps) external {
  161:         _atLeastRole(ADMIN);
  162:         delete steps;
  163: 
  164:         uint256 numSteps = _newSteps.length;
  165:         for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
  166:             address[2] memory step = _newSteps[i];
  167:             require(step[0] != address(0));
  168:             require(step[1] != address(0));
  169:             steps.push(step);
  170:         }
  171:     }

```

### Proof of Concept

https://blog.soliditylang.org/2022/08/08/calldata-tuple-reencoding-head-overflow-bug/


### Recommended Mitigation Steps

Use  solidity of version min. 0.8.16

### [L-03] `uint` type argument was use instead `uint256` for `permit`function , this will cause the function selector to be miscalculated

`LUSDToken.permit()` function is important external function and use for external account interaction, when external contract accounts interact with the contract, they trigger the `permit` function with interface or low level call

However, the best practice of the `permit` function is to use `uint256` type arguments, in the project it cannot interact with the `permit` function of the contract because it is used with `uint` type, the operation will be reverted


```solidity
Ethos-Core/contracts/LUSDToken.sol:
  261  
  262:     function permit
  263:     (
  264:         address owner, 
  265:         address spender, 
  266:         uint amount, 
  267:         uint deadline, 
  268:         uint8 v, 
  269:         bytes32 r, 
  270:         bytes32 s
  271:     ) 
  272:         external 
  273:         override 
  274:     {
```

### Proof of Concept

```solidity

contract CheckFunctionSelector {

    // 0x97a6e84a
    function functionSelector_uint() external pure returns (bytes4 fourbyte) {
       return bytes4(keccak256(bytes("permit(address,address,uint,uint,uint8,bytes32,bytes32)")));
    }

    // 0xd505accf
    function functionSelector_uint256() external pure returns (bytes4 fourbyte) {
       return bytes4(keccak256(bytes("permit(address,address,uint256,uint256,uint8,bytes32,bytes32)")));
    }

}
```


### Recommended Mitigation Steps


```diff
Ethos-Core/contracts/LUSDToken.sol:
  261  
  262:     function permit
  263:     (
  264:         address owner, 
  265:         address spender, 
- 266:         uint amount, 
+ 266:         uint256 amount, 
- 267:         uint deadline, 
+ 267:         uint256 deadline,
  268:         uint8 v, 
  269:         bytes32 r, 
  270:         bytes32 s
  271:     ) 
  272:         external 
  273:         override 
  274:     {
```

### [L-04] First ERC4626 deposit exploit can break share calculation


### Impact

A well known attack vector for almost all shares based liquidity pool contracts, where an early user can manipulate the price per share and profit from late users' deposits because of the precision loss caused by the rather large value of price per share.


```solidity

Ethos-Vault/contracts/ReaperVaultERC4626.sol:
  50      // the “average-user’s” price-per-share, meaning what the average user should expect to see when exchanging to and from.
  51:     function convertToShares(uint256 assets) public view override returns (uint256 shares) {
  52:         if (totalSupply() == 0 || _freeFunds() == 0) return assets;
  53:         return (assets * totalSupply()) / _freeFunds();
  54:     }

```
### Proof Of Concept
1 - A malicious early user can `deposit()` with 1 wei of asset token as the first depositor of the LToken, and get 1 wei of shares

2 - Then the attacker can send 10000e18 - 1 of asset tokens and inflate the price per share from 1.0000 to an extreme value of 1.0000e22 ( from (1 + 10000e18 - 1) / 1)

3 - As a result, the future user who deposits 19999e18 will only receive 1 wei (from 19999e18 * 1 / 10000e18) of shares token

4 - They will immediately lose 9999e18 or half of their deposits if they `redeem()` right after the `deposit()`

The attacker can profit from future users' deposits. While the late users will lose part of their funds to the attacker.


### Recommended Mitigation Steps

Consider sending the first 1000 shares to the address 0, a mitigation used in Uniswap V2


### [L-05] initialize() function can be called by anybody

`initialize()` function can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the ` __ReaperBaseStrategy_init()` function and 
`__ReaperBaseStrategy_init()`  function sets DEFAULT_ADMIN_ROLE , so this is very important risk

Here is a definition of `initialize()` function.

```solidity

Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol:
  61       */
  62:     function initialize(
  63:         address _vault,
  64:         address[] memory _strategists,
  65:         address[] memory _multisigRoles,
  66:         IAToken _gWant
  67:     ) public initializer {
  68:         gWant = _gWant;
  69:         want = _gWant.UNDERLYING_ASSET_ADDRESS();
  70:         __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
  71:         rewardClaimingTokens = [address(_gWant)];
  72:     }
```


### Recommended Mitigation Steps

Add a control that makes `initialize()` only call the Deployer Contract or EOA;

```js
if (msg.sender != DEPLOYER_ADDRESS) {
            revert NotDeployer();
        }
```
### [L-06] Insufficient coverage

**Description:**
The test coverage rate of the project is ~93%. Testing all functions is best practice in terms of security criteria.
Due to its capacity, test coverage is expected to be 100%.


```js

1 result - 1 file

README.md:
  91: - What is the overall line coverage percentage provided by your tests?:  93

```

### [L-07] Missing Event for initialize

**Context:**
```solidity

Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol:
  61       */
  62:     function initialize(
  63:         address _vault,
  64:         address[] memory _strategists,
  65:         address[] memory _multisigRoles,
  66:         IAToken _gWant
  67:     ) public initializer {
  68:         gWant = _gWant;
  69:         want = _gWant.UNDERLYING_ASSET_ADDRESS();
  70:         __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
  71:         rewardClaimingTokens = [address(_gWant)];
  72:     }


```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit

### [L-08] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol


### [L-09] Loss of precision due to rounding in ` amount / uint256(rewardsPerSecond) `

If totalSupply is too low then the rounding problem is big 

Add scalar so roundings are negligible

```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol:
  365:         value = (_freeFunds() * _shares) / totalSupply();

```



### [L-10]  Use Fuzzing Test for complicated code bases 


I recommend fuzzing testing in complex code structures, especially Ethos Reserve, where there is an innovative code base and a lot of computation.

**Recommendation:**

Use should fuzzing test like Echidna.

As Alberto Cuesta Canada said:

_Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now._



https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05


### [L-11] Update codes to avoid Compile Errors


```js
contracts/StabilityPool.sol:413:1: Warning: Unused local variable.
uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit); // Needed only for event log
^-----------^

contracts/StabilityPool.sol:502:1: Warning: Unused local variable.
uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit); // Needed only for event log
^-----------^

contracts/ActivePool.sol:26:1: Warning: Contract code size exceeds 24576 bytes (a limit introduced in Spurious Dragon). This contract may not be deployable on mainnet. Consider enabling the optimizer (with a low "runs" value!), turning off revert strings, or using libraries.
contract ActivePool is Ownable, CheckContract, IActivePool {
^ (Relevant source part starts here and spans across multiple lines).

contracts/BorrowerOperations.sol:18:1: Warning: Contract code size exceeds 24576 bytes (a limit introduced in Spurious Dragon). This contract may not be deployable on mainnet. Consider enabling the optimizer (with a low "runs" value!), turning off revert strings, or using libraries.
contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOperations {
^ (Relevant source part starts here and spans across multiple lines).

contracts/LQTY/LQTYStaking.sol:18:1: Warning: Contract code size exceeds 24576 bytes (a limit introduced in Spurious Dragon). This contract may not be deployable on mainnet. Consider enabling the optimizer (with a low "runs" value!), turning off revert strings, or using libraries.
contract LQTYStaking is ILQTYStaking, Ownable, CheckContract, BaseMath {
^ (Relevant source part starts here and spans across multiple lines).

contracts/TroveManager.sol:18:1: Warning: Contract code size exceeds 24576 bytes (a limit introduced in Spurious Dragon). This contract may not be deployable on mainnet. Consider enabling the optimizer (with a low "runs" value!), turning off revert strings, or using libraries.
contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager {
^ (Relevant source part starts here and spans across multiple lines).

contracts/SortedTroves.sol:46:1: Warning: Contract code size exceeds 24576 bytes (a limit introduced in Spurious Dragon). This contract may not be deployable on mainnet. Consider enabling the optimizer (with a low "runs" value!), turning off revert strings, or using libraries.
contract SortedTroves is Ownable, CheckContract, ISortedTroves {
^ (Relevant source part starts here and spans across multiple lines).

contracts/PriceFeed.sol:24:1: Warning: Contract code size exceeds 24576 bytes (a limit introduced in Spurious Dragon). This contract may not be deployable on mainnet. Consider enabling the optimizer (with a low "runs" value!), turning off revert strings, or using libraries.
contract PriceFeed is Ownable, CheckContract, BaseMath, IPriceFeed {
^ (Relevant source part starts here and spans across multiple lines).

contracts/Proxy/BorrowerWrappersScript.sol:21:1: Warning: Contract code size exceeds 24576 bytes (a limit introduced in Spurious Dragon). This contract may not be deployable on mainnet. Consider enabling the optimizer (with a low "runs" value!), turning off revert strings, or using libraries.
contract BorrowerWrappersScript is BorrowerOperationsScript, ERC20TransferScript, LQTYStakingScript {
^ (Relevant source part starts here and spans across multiple lines).

contracts/RedemptionHelper.sol:16:1: Warning: Contract code size exceeds 24576 bytes (a limit introduced in Spurious Dragon). This contract may not be deployable on mainnet. Consider enabling the optimizer (with a low "runs" value!), turning off revert strings, or using libraries.
contract RedemptionHelper is LiquityBase, Ownable, IRedemptionHelper {
^ (Relevant source part starts here and spans across multiple lines).

contracts/StabilityPool.sol:146:1: Warning: Contract code size exceeds 24576 bytes (a limit introduced in Spurious Dragon). This contract may not be deployable on mainnet. Consider enabling the optimizer (with a low "runs" value!), turning off revert strings, or using libraries.
contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
^ (Relevant source part starts here and spans across multiple lines).



```
### [L-12] Add to Event-Emit for critical functions


```solidity
3 functions 


Ethos-Core/contracts/LUSDToken.sol:
  132  
  133:     function pauseMinting() external {
  134:         require(
  135:             msg.sender == guardianAddress || msg.sender == governanceAddress,
  136:             "LUSD: Caller is not guardian or governance"
  137:         );
  138:         mintingPaused = true;
  139:     }
  140: 
  141:     function unpauseMinting() external {
  142:         _requireCallerIsGovernance();
  143:         mintingPaused = false;
  144:     }
  145 

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
  166       */
  167:     function initiateUpgradeCooldown() external {
  168:         _atLeastRole(STRATEGIST);
  169:         upgradeProposalTime = block.timestamp;
  170:     }
```

### [L-13] Use `uint256` instead `uint`

Project use uint and uint256
 
Number of uses:
uint  = 477 results – 8 files
uint256 = 281 results - 11 files


Some developers prefer to use `uint256` because it is consistent with other uint data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs.

For example if doing ```bytes4(keccak('transfer(address, uint)’))```, you'll get a different method sig ID than ```bytes4(keccak('transfer(address, uint256)’))``` and smart contracts will only understand the latter when comparing method sig IDs


### [L-14] Signature Malleability of EVM's ecrecover()

**Context:**

```solidity

Ethos-Core/contracts/LUSDToken.sol:
  261  
  262:     function permit
           // ...

  283:         bytes32 digest = keccak256(abi.encodePacked('\x19\x01', 
  284:                          domainSeparator(), keccak256(abi.encode(
  285:                          _PERMIT_TYPEHASH, owner, spender, amount, 
  286:                          _nonces[owner]++, deadline))));
  287:         address recoveredAddress = ecrecover(digest, v, r, s);

```
**Description:**
Description: The function calls the Solidity ecrecover() function directly to verify the given signatures. However, the ecrecover() EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin's ECDSA library.

*Recommendation:**
Use the ecrecover function from OpenZeppelin's ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).



### [L-15] `sendCollateral` event is missing parameters


The `ActivePool.sol` contract has very important function; `sendCollateral` 


However, only amounts are published in emits, whereas msg.sender must be specified in every transaction, a contract or web page listening to events cannot react to users, emit does not serve the purpose.
Basically, this event cannot be used


```diff
Ethos-Core/contracts/ActivePool.sol:
  170  
  171:     function sendCollateral(address _collateral, address _account, uint _amount) external override {
  172:         _requireValidCollateralAddress(_collateral);
  173:         _requireCallerIsBOorTroveMorSP();
  174:         _rebalance(_collateral, _amount);
  175:         collAmount[_collateral] = collAmount[_collateral].sub(_amount);
  176:         emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);
- 177:         emit CollateralSent(_collateral, _account, _amount);
+ 177:         emit CollateralSent(msg.sender_collateral, _account, _amount);



```



### Recommended Mitigation Steps
add `msg.sender` parameter in event-emit


### [L-16] Lack of Input Validation

For defence-in-depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid.

[OpenZeppelinTokenizedVault.sol#L9](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/7c75b8aa89073376fb67d78a40f6d69331092c94/contracts/token/ERC20/extensions/ERC20TokenizedVault.sol#L95)


```diff

Ethos-Vault/contracts/ReaperVaultV2.sol:
  318      // shares to {_receiver}. Returns the number of shares that were minted.
  319:     function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {
  320:         _atLeastRole(DEPOSITOR);
  321:         require(!emergencyShutdown, "Cannot deposit during emergency shutdown");
  322:         require(_amount != 0, "Invalid amount");
+                 require(shares <= maxMint(receiver), "mint more than max");
  323:         uint256 pool = balance();
  324:         require(pool + _amount <= tvlCap, "Vault is full");
  325: 
  326:         uint256 freeFunds = _freeFunds();
  327:         uint256 balBefore = token.balanceOf(address(this));
  328:         token.safeTransferFrom(msg.sender, address(this), _amount);
  329:         uint256 balAfter = token.balanceOf(address(this));
  330:         _amount = balAfter - balBefore;
  331:         if (totalSupply() == 0) {
  332:             shares = _amount;
  333:         } else {
  334:             shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"
  335:         }
  336:         _mint(_receiver, shares);
  337:         emit Deposit(msg.sender, _receiver, _amount, shares);
  338:     }




```
### [L-17] Prevent division by `0`

On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.
These functions can be calledd with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.



```diff

Ethos-Vault/contracts/ReaperVaultV2.sol:
  358      // and return corresponding assets to {_receiver}. Returns the number of assets that were returned.
  359:     function _withdraw(
  360:         uint256 _shares,
  361:         address _receiver,
  362:         address _owner
  363:     ) internal nonReentrant returns (uint256 value) {
  364:         require(_shares != 0, "Invalid amount");
- 365:         value = (_freeFunds() * _shares) / totalSupply();
+	  if (totalSupply() == 0) {
+  	      revert Unauthorized("total supply is zero");
+	  } else {
+  	       value = (_freeFunds * _shares) / totalSupply();
+	 }

  366:         _burn(_owner, _shares);
```

### [L-18] Project has NPM Dependency which uses a vulnerable version : `@openzeppelin`


```js
Ethos-Vault/package.json:
  39    },
  40:   "dependencies": {
  41:     "@openzeppelin/contracts": "^4.7.3",
  42:     "@openzeppelin/contracts-upgradeable": "^4.7.3",
 
```

### Proof Of Concept
https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/



### Recommended Mitigation Steps
Upgrade OZ to version 4.8.0 or higher


### [L-19] Keccak Constant values should used to immutable rather than constant

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand.



```solidity

8 results - 2 files

Ethos-Vault/contracts/ReaperVaultV2.sol:
  73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
  74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
  75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
  76:     bytes32 public constant ADMIN = keccak256("ADMIN");

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
  49:     bytes32 public constant KEEPER = keccak256("KEEPER");
  50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
  51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
  52:     bytes32 public constant ADMIN = keccak256("ADMIN");
```
### [L-20] Due to the novelty of the ERC4626 standard, it is safer to use as upgradeable



ERC-4626 is a standard to optimize and unify the technical parameters of yield-bearing vaults. It provides a standard API for tokenized yield-bearing vaults that represent shares of a single underlying ERC-20 token. ERC-4626 also outlines an optional extension for tokenized vaults utilizing ERC-20, offering basic functionality for depositing, withdrawing tokens and reading balances.


ERC4626 Tokenized Vauldt was created 2021-12-22 so this standart is safer to use as upgrade


### [L-21] In the `setAddresses` function, there is no return of incorrect address identification

In case of incorrect address definition in the `setAddresses` function, there is no way to fix it because of the `owner = address(0)` at the end of the function

It is recommended to fix the architecture
1- Address definitions can be defined in the constructor
2- Because of `owner = address(0)` at the end of the function, there is no way to fix it, so the owner's authority can be maintained.

```solidity
Ethos-Core/contracts/TroveManager.sol:
  231  
  232:     function setAddresses(
  233:         address _borrowerOperationsAddress,
  234:         address _collateralConfigAddress,
  235:         address _activePoolAddress,
  236:         address _defaultPoolAddress,
  237:         address _stabilityPoolAddress,
  238:         address _gasPoolAddress,
  239:         address _collSurplusPoolAddress,
  240:         address _priceFeedAddress,
  241:         address _lusdTokenAddress,
  242:         address _sortedTrovesAddress,
  243:         address _lqtyTokenAddress,
  244:         address _lqtyStakingAddress,
  245:         address _redemptionHelperAddress
  246:     )
  247:         external
  248:         override
  249:     {
  250:         require(msg.sender == owner);
  251: 
  252:         checkContract(_borrowerOperationsAddress);
  253:         checkContract(_collateralConfigAddress);
  254:         checkContract(_activePoolAddress);
  255:         checkContract(_defaultPoolAddress);
  256:         checkContract(_stabilityPoolAddress);
  257:         checkContract(_gasPoolAddress);
  258:         checkContract(_collSurplusPoolAddress);
  259:         checkContract(_priceFeedAddress);
  260:         checkContract(_lusdTokenAddress);
  261:         checkContract(_sortedTrovesAddress);
  262:         checkContract(_lqtyTokenAddress);
  263:         checkContract(_lqtyStakingAddress);
  264:         checkContract(_redemptionHelperAddress);
  265: 
  266:         borrowerOperationsAddress = _borrowerOperationsAddress;
  267:         collateralConfig = ICollateralConfig(_collateralConfigAddress);
  268:         activePool = IActivePool(_activePoolAddress);
  269:         defaultPool = IDefaultPool(_defaultPoolAddress);
  270:         stabilityPool = IStabilityPool(_stabilityPoolAddress);
  271:         gasPoolAddress = _gasPoolAddress;
  272:         collSurplusPool = ICollSurplusPool(_collSurplusPoolAddress);
  273:         priceFeed = IPriceFeed(_priceFeedAddress);
  274:         lusdToken = ILUSDToken(_lusdTokenAddress);
  275:         sortedTroves = ISortedTroves(_sortedTrovesAddress);
  276:         lqtyToken = IERC20(_lqtyTokenAddress);
  277:         lqtyStaking = ILQTYStaking(_lqtyStakingAddress);
  278:         redemptionHelper = IRedemptionHelper(_redemptionHelperAddress);
  279: 
  280:         emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);
  281:         emit CollateralConfigAddressChanged(_collateralConfigAddress);
  282:         emit ActivePoolAddressChanged(_activePoolAddress);
  283:         emit DefaultPoolAddressChanged(_defaultPoolAddress);
  284:         emit StabilityPoolAddressChanged(_stabilityPoolAddress);
  285:         emit GasPoolAddressChanged(_gasPoolAddress);
  286:         emit CollSurplusPoolAddressChanged(_collSurplusPoolAddress);
  287:         emit PriceFeedAddressChanged(_priceFeedAddress);
  288:         emit LUSDTokenAddressChanged(_lusdTokenAddress);
  289:         emit SortedTrovesAddressChanged(_sortedTrovesAddress);
  290:         emit LQTYTokenAddressChanged(_lqtyTokenAddress);
  291:         emit LQTYStakingAddressChanged(_lqtyStakingAddress);
  292:         emit RedemptionHelperAddressChanged(_redemptionHelperAddress);
  293: 
  294:         owner = address(0);
  295:     }
  296  

```
### [N-01] Omissions in Events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

```diff

Ethos-Core/contracts/LQTY/CommunityIssuance.sol:
  60  
  61:     function setAddresses
  62:     (
  63:         address _oathTokenAddress, 
  64:         address _stabilityPoolAddress
  65:     ) 
  66:         external 
  67:         onlyOwner 
  68:         override 
  69:     {
  70:         require(!initialized, "issuance has been initialized");
  71:         checkContract(_oathTokenAddress);
  72:         checkContract(_stabilityPoolAddress);
  73: 
  74:         OathToken = IERC20(_oathTokenAddress);
  75:         stabilityPoolAddress = _stabilityPoolAddress;
  76: 
  77:         initialized = true;
  78: 
- 79:         emit OathTokenAddressSet(_oathTokenAddress);
- 80:         emit StabilityPoolAddressSet(_stabilityPoolAddress);
+ 79:         emit OathTokenAddressSet(old_oathTokenAddress , _oathTokenAddress);
+ 80:         emit StabilityPoolAddressSet(old_ stabilityPoolAddress, _stabilityPoolAddress);

  81:     }


Ethos-Vault/contracts/ReaperVaultV2.sol:
  257       */
  258:     function setWithdrawalQueue(address[] calldata _withdrawalQueue) external {
  259:         _atLeastRole(ADMIN);
  260:         uint256 queueLength = _withdrawalQueue.length;
  261:         require(queueLength != 0, "Queue must not be empty");
  262: 
  263:         delete withdrawalQueue;
  264:         for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
  265:             address strategy = _withdrawalQueue[i];
  266:             StrategyParams storage params = strategies[strategy];
  267:             require(params.activation != 0, "Invalid strategy address");
  268:             withdrawalQueue.push(strategy);
  269:         }
- 270:         emit UpdateWithdrawalQueue(_withdrawalQueue);
+ 270:         emit UpdateWithdrawalQueue(oldWithdrawalQueue, _withdrawalQueue);

  271:     }


Ethos-Core/contracts/ActivePool.sol:
  137  
  138:     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
  139:         _requireValidCollateralAddress(_collateral);
  140:         yieldClaimThreshold[_collateral] = _threshold;
- 141:         emit YieldClaimThresholdUpdated(_collateral, _threshold);
+ 141:         emit YieldClaimThresholdUpdated(oldCollateral, _collateral, _threshold);
 142      }

```

### [N-02] NatSpec comments should be increased in contracts

**Context:**
All project have 226 functions (without interfaces) but they have only 95 NatSpec’s

**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
NatSpec comments should be increased in contracts

### [N-03] `Function writing` that does not comply with the `Solidity Style Guide`

**Context:**
All Contracts

**Description:**
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

 constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

### [N-04] Include return parameters in NatSpec comments

**Context:**
All project have 118 returns variable (without interfaces) but they have only 20 @return NatSpec’s


**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
Include return parameters in NatSpec comments

_Recommendation  Code Style: (from Uniswap3)_
```js
    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);
```
### [N-05] Tokens accidentally sent to the contract cannot be recovered

Using balanceOf for reward distribution means accidentally sent reward tokens to the contract will be distributed to all users who are eligible for rewards. 

Examine and consider the trade-offs of using internal accounting instead of balanceOf(address(this) . Implement functions that allow to withdraw the accidentally sent tokens from the contracts.


**Contex**
Ethos-Vault/contracts/ReaperVaultERC4626.sol


It can't be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

### Recommended Mitigation Steps

Add this code:

```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```

### [N-06] Repeated validation logic can be converted into a function modifier

If a query or logic is repeated over many lines, using a modifier improves the readability and reusability of the code


```solidity

13 results - 5 files

Ethos-Core/contracts/ActivePool.sol:
  92          checkContract(_collSurplusPoolAddress);
  93:         require(_treasuryAddress != address(0), "Treasury cannot be 0 address");

Ethos-Core/contracts/LUSDToken.sol:
  311      function _transfer(address sender, address recipient, uint256 amount) internal {
  312:         assert(sender != address(0));
  313:         assert(recipient != address(0));
  314  

  320      function _mint(address account, uint256 amount) internal {
  321:         assert(account != address(0));
  322  

  328      function _burn(address account, uint256 amount) internal {
  329:         assert(account != address(0));
  330          

  336      function _approve(address owner, address spender, uint256 amount) internal {
  337:         assert(owner != address(0));
  338:         assert(spender != address(0));
  339  

  347          require(
  348:             _recipient != address(0) && 

Ethos-Core/contracts/TroveManager.sol:
  293  
  294:         owner = address(0);
  295      }

Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol:
  167:             require(step[0] != address(0));
  168:             require(step[1] != address(0));


Ethos-Vault/contracts/ReaperVaultV2.sol:

  151:         require(_strategy != address(0), "Invalid strategy address");
  629:         require(newTreasury != address(0), "Invalid address");

```


### [N-07] Use a more recent version of Solidity

**Context:**
```solidity
14 results - 12 files

Ethos-Core/contracts/ActivePool.sol:
  2  
  3: pragma solidity 0.6.11;
  4  

Ethos-Core/contracts/BorrowerOperations.sol:
  2  
  3: pragma solidity 0.6.11;
  4  

Ethos-Core/contracts/CollateralConfig.sol:
  2  
  3: pragma solidity 0.6.11;
  4  

Ethos-Core/contracts/LUSDToken.sol:
  2  
  3: pragma solidity 0.6.11;
  4  

Ethos-Core/contracts/StabilityPool.sol:
   2  
   3: pragma solidity 0.6.11;
   4  

Ethos-Core/contracts/TroveManager.sol:
  2  
  3: pragma solidity 0.6.11;
  4  

Ethos-Core/contracts/LQTY/CommunityIssuance.sol:
  2  
  3: pragma solidity 0.6.11;
  4  

Ethos-Core/contracts/LQTY/LQTYStaking.sol:
  2  
  3: pragma solidity 0.6.11;
  4  

Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol:
  2  
  3: pragma solidity ^0.8.0;
  4  

Ethos-Vault/contracts/ReaperVaultERC4626.sol:
  2  
  3: pragma solidity ^0.8.0;
  4  

Ethos-Vault/contracts/ReaperVaultV2.sol:
  2  
  3: pragma solidity ^0.8.0;
  4  

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
  2  
  3: pragma solidity ^0.8.0;
  4 

```
**Description:**
For security, it is best practice to use the latest Solidity version.
For the security fix list in the versions;
https://github.com/ethereum/solidity/blob/develop/Changelog.md


**Recommendation:**
Old version of Solidity is used `(0.8.0 / 0.6.11)`, newer version can be used `(0.8.17)` 


### [N-08] For functions, follow Solidity standard naming conventions (internal function style rule)

**Context:**
```solidity

4 results - 2 files

Ethos-Core/contracts/ActivePool.sol:
  41:     mapping (address => uint256) internal collAmount;  // collateral => amount tracker
  42:     mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker

Ethos-Core/contracts/StabilityPool.sol:
  167:     mapping (address => uint256) internal collAmounts;  // deposited collateral tracker
  170:     uint256 internal totalLUSDDeposits;

```


**Description:**
The above codes don't follow Solidity's standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)


### [N-09] Floating pragma

**Description:**
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List: 

```solidity
4 results 

Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol:
  2  
  3: pragma solidity ^0.8.0;
  4  

Ethos-Vault/contracts/ReaperVaultERC4626.sol:
  2  
  3: pragma solidity ^0.8.0;
  4  

Ethos-Vault/contracts/ReaperVaultV2.sol:
  2  
  3: pragma solidity ^0.8.0;
  4  

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
  2  
  3: pragma solidity ^0.8.0;
  4 

```


**Recommendation:**
Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

### [N-10] Use descriptive names for Contracts and Libraries


This codebase will be difficult to navigate, as there are no descriptive naming conventions that specify which files should contain meaningful logic.

Prefixes should be added like this by filing:

Interface I_
absctract contracts Abs_
Libraries Lib_

We recommend that you implement this or a similar agreement.

### [N-11] Use SMTChecker

The *highest* tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

### [N-12]  Add NatSpec Mapping comment

**Description:**
Add NatSpec comments describing mapping keys and values


**Context:**

```solidity
13 results - 4 files

Ethos-Core/contracts/CollateralConfig.sol:
  35:     mapping (address => Config) public collateralConfig; 

Ethos-Core/contracts/LUSDToken.sol:
  54:     mapping (address => uint256) private _nonces;
  57:     mapping (address => uint256) private _balances;
  58:     mapping (address => mapping (address => uint256)) private _allowances;  
  62:     mapping (address => bool) public troveManagers;
  63:     mapping (address => bool) public stabilityPools;
  64:     mapping (address => bool) public borrowerOperations;

Ethos-Core/contracts/StabilityPool.sol:
  167:     mapping (address => uint256) internal collAmounts;  // deposited collateral tracker
  179:         mapping (address => uint) S;
  223:     mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;
  228:     mapping (address => uint) public lastCollateralError_Offset;

Ethos-Core/contracts/LQTY/LQTYStaking.sol:
  35:         mapping (address => uint) F_Collateral_Snapshot;
```

**Recommendation:**

```solidity

 /// @dev    address(user) -> address(ERC0 Contract Address) -> uint256(allowance amount from user)
    mapping(address => mapping(address => uint256)) public allowance;

```

### [N-13] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier


```diff

2 results - 2 files

Ethos-Core/contracts/LQTY/CommunityIssuance.sol:
  57      constructor() public {
- 58:         distributionPeriod = 14 days;
+ 58:         distributionPeriod = 14 days; // 1_209_600 (14*24*60*60)

  59      }

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
-  24      uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF
+            //minimum 48 hours for RF
+ 24      uint256 public constant UPGRADE_TIMELOCK = 48 hours; // 172_800  (48*60*60)

- 25:     uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100; //  3_153_600_000 (365*24*60*60*100)


```
### [N-14] Lines are too long

[TroveManager.sol#L102](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L102)

[TroveManager.sol#L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L97)

[TroveManager.sol#L200-L202](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L200-L202)

[TroveManager.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L351)

[TroveManager.sol#L393](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L393)

[TroveManager.sol#L407](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L407)

[TroveManager.sol#L428](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L428)


[TroveManager.sol#L572](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L572)

[TroveManager.sol#L775](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L775)

[TroveManager.sol#L926](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L926)


**Description:**
It is generally recommended that lines in the source code should not exceed 80-120 characters. Today's screens are much larger, so in some cases it makes sense to expand that.The lines above should be split when they reach that length, as the files will most likely be on GitHub and GitHub always uses a scrollbar when the length is more than 164 characters.

(why-is-80-characters-the-standard-limit-for-code-width)[https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width]

**Recommendation:**
Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html#introduction

```js
thisFunctionCallIsReallyLong(
    longArgument1,
    longArgument2,
    longArgument3
);
```
### [N-15] Constants on the left are better

If you use the constant first you support structures that veil programming errors. And one should only produce code either to add functionality, fix an programming error or trying to support structures to avoid programming errors (like design patterns). 

https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html

```solidity
36 results - 6 files

Ethos-Core/contracts/BorrowerOperations.sol:
  534:         require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");
  568:         require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");

Ethos-Core/contracts/StabilityPool.sol:
  428:         if (totalLUSD == 0 || _LQTYIssuance == 0) {return;}
  469:         if (totalLUSD == 0 || _debtToOffset == 0) { return; }
  489:         if (totalLUSD == 0) { return; }
  575:         if (newProductFactor == 0) {
  685:         if (initialDeposit == 0) {return 0;}
  720:         if (initialDeposit == 0) { return 0; }
  751:         if (scaleDiff == 0) {
  784:         if (_amount == 0) {return;}
  795:         if (LUSDWithdrawal == 0) {return;}
  809:         if (_newValue == 0) {

Ethos-Core/contracts/TroveManager.sol:
   616:                 if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { break; }
   817:                 if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { continue; }
  1127:         if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }
  1142:         if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }
  1215:         if (totalCollateralSnapshot[_collateral] == 0) {
  1238:         if (_debt == 0) { return; }

Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol:
  121:             if (amount == 0) {

Ethos-Vault/contracts/ReaperVaultERC4626.sol:
   52:         if (totalSupply() == 0 || _freeFunds() == 0) return assets;
   67:         if (totalSupply() == 0) return shares;
  140:         if (totalSupply() == 0) return shares;
  184:         if (totalSupply() == 0 || _freeFunds() == 0) return 0;

Ethos-Vault/contracts/ReaperVaultV2.sol:
  152:         require(strategies[_strategy].activation == 0, "Strategy already added");
  210:         if (strategies[_strategy].allocBPS == 0) {
  227:         if (totalAllocBPS == 0 || emergencyShutdown) {
  296:         return totalSupply() == 0 ? 10**decimals() : (_freeFunds() * 10**decimals()) / totalSupply();
  331:         if (totalSupply() == 0) {
  380:                 if (strategyBal == 0) {
  466:             uint256 shares = supply == 0 ? performanceFee : (performanceFee * supply) / _freeFunds();
  555:         if (strategy.allocBPS == 0 || emergencyShutdown) {

```

### [N-16] `constants` should be defined rather than using magic numbers

A magic number is a numeric literal that is used in the code without any explanation of its meaning. The use of magic numbers makes programs less readable and hence more difficult to maintain and update.
Even assembly can benefit from using readable constants instead of hex/numeric literals

```solidity
3 result 

Ethos-Core/contracts/TroveManager.sol:
  54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
  55:     uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
  25:     uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;

```


### [N-17] Missing timelock for critical parameter change


```solidity

1 result - 1 file

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
  155       */
  156:     function setEmergencyExit() external {
  157:         _atLeastRole(GUARDIAN);
  158:         emergencyExit = true;
  159:         IVault(vault).revokeStrategy(address(this));
  160:     }

```

### [N-18] Use the `delete` keyword instead of assigning a value of `0`


Using the 'delete' keyword instead of assigning a '0' value is a detailed optimization that increases code readability and audit quality, and clearly indicates the intent.

Other hand, if use `delete` instead `0` value assign , it will be gas saved

```solidity
17 results - 4 files

Ethos-Core/contracts/ActivePool.sol:
  253:             vars.profit = 0;

Ethos-Core/contracts/StabilityPool.sol:
  529:             lastLUSDLossError_Offset = 0;
  578:             currentScale = 0;
  756:             compoundedDeposit = 0;

Ethos-Core/contracts/TroveManager.sol:
   387:             singleLiquidation.debtToOffset = 0;
   388:             singleLiquidation.collToSendToSP = 0;
   468:             debtToOffset = 0;
   469:             collToSendToSP = 0;
   505:         singleLiquidation.debtToRedistribute = 0;
   506:         singleLiquidation.collToRedistribute = 0;
  1192:         Troves[_borrower][_collateral].stake = 0;
  1285:         Troves[_borrower][_collateral].coll = 0;
  1286:         Troves[_borrower][_collateral].debt = 0;
  1288:         rewardSnapshots[_borrower][_collateral].collAmount = 0;
  1289:         rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;

Ethos-Vault/contracts/ReaperVaultV2.sol:
  215:         strategies[_strategy].allocBPS = 0;
  537:             lockedProfit = 0;

```


### [N-19] Not using the named return variables anywhere in the function is confusing


```solidity

2 results

Ethos-Core/contracts/StabilityPool.sol:
  626:     function getDepositorCollateralGain(address _depositor) public view override returns (address[] memory assets, uint[] memory amounts) {
  627:         uint initialDeposit = deposits[_depositor].initialValue;
  628: 
  629:         if (initialDeposit != 0) { 
  630:             Snapshots storage snapshots = depositSnapshots[_depositor];
  631: 
  632:             return _getCollateralGainFromSnapshots(initialDeposit, snapshots);
  633:         }

Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol:
  103: 
  104:     function _liquidateAllPositions() internal override returns (uint256 amountFreed) {
  105:         _withdrawUnderlying(balanceOfPool());
  106:         return balanceOfWant();
  107:     }

```

**Recommendation:**
Consider adopting a consistent approach to return values throughout the codebase by removing all named return variables, explicitly declaring them as local variables, and adding the necessary return statements where appropriate. This would improve both the explicitness and readability of the code, and it may also help reduce regressions during future code refactors.


### [N-20] Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked


```diff

Ethos-Core/contracts/TroveManager.sol:
  715:     function batchLiquidateTroves(address _collateral, address[] memory _troveArray) public override {
+                require(memory _troveArray.length< max troveArrayLengt, "max length");

  716:         require(_troveArray.length != 0);
  717: 
  718:         IActivePool activePoolCached = activePool;
  719:         IDefaultPool defaultPoolCached = defaultPool;
  720:         IStabilityPool stabilityPoolCached = stabilityPool;
  721: 
  722:         LocalVariables_OuterLiquidationFunction memory vars;
  723:         LiquidationTotals memory totals;

```

**Recommendation:**
Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to gas limitations or failed transactions. Consider bounding the loop length if the array is expected to be growing and/or handling a huge list of elements to avoid unnecessary gas wastage and denial of service.

### [N-21] According to the syntax rules, use `mapping ()` instead of `mapping()` using spaces as keyword

```js
2 results - 2 files

Ethos-Core/contracts/LQTY/LQTYStaking.sol:
- 25:     mapping( address => uint) public stakes;
+ 25:     mapping ( address => uint) public stakes;

Ethos-Vault/contracts/ReaperVaultV2.sol:
- 35:     mapping(address => StrategyParams) public strategies;
+ 35:     mapping (address => StrategyParams) public strategies;

```
### [N-22] For modern and more readable code; update import usages

**Context:**
```solidity

110 results - 12 files

Ethos-Core/contracts/ActivePool.sol:
   4  
   5: import './Interfaces/IActivePool.sol';
   6: import "./Interfaces/ICollateralConfig.sol";
   7: import './Interfaces/IDefaultPool.sol';
   8: import "./Interfaces/ICollSurplusPool.sol";
   9: import "./Interfaces/ILQTYStaking.sol";
  10: import "./Interfaces/IStabilityPool.sol";
  11: import "./Interfaces/ITroveManager.sol";
  12: import "./Dependencies/SafeMath.sol";
  13: import "./Dependencies/Ownable.sol";
  14: import "./Dependencies/CheckContract.sol";
  15: import "./Dependencies/console.sol";
  16: import "./Dependencies/SafeERC20.sol";
  17: import "./Dependencies/IERC4626.sol";
  18  

Ethos-Core/contracts/BorrowerOperations.sol:
   4  
   5: import "./Interfaces/IBorrowerOperations.sol";
   6: import "./Interfaces/ICollateralConfig.sol";
   7: import "./Interfaces/ITroveManager.sol";
   8: import "./Interfaces/ILUSDToken.sol";
   9: import "./Interfaces/ICollSurplusPool.sol";
  10: import "./Interfaces/ISortedTroves.sol";
  11: import "./Interfaces/ILQTYStaking.sol";
  12: import "./Dependencies/LiquityBase.sol";
  13: import "./Dependencies/Ownable.sol";
  14: import "./Dependencies/CheckContract.sol";
  15: import "./Dependencies/console.sol";
  16: import "./Dependencies/SafeERC20.sol";
  17  

Ethos-Core/contracts/CollateralConfig.sol:
  4  
  5: import "./Dependencies/CheckContract.sol";
  6: import "./Dependencies/Ownable.sol";
  7: import "./Dependencies/SafeERC20.sol";
  8: import "./Interfaces/ICollateralConfig.sol";
  9  

Ethos-Core/contracts/LUSDToken.sol:
  4  
  5: import "./Interfaces/ILUSDToken.sol";
  6: import "./Interfaces/ITroveManager.sol";
  7: import "./Dependencies/SafeMath.sol";
  8: import "./Dependencies/CheckContract.sol";
  9: import "./Dependencies/console.sol";
  

Ethos-Core/contracts/StabilityPool.sol:
   4  
   5: import './Interfaces/IBorrowerOperations.sol';
   6: import "./Interfaces/ICollateralConfig.sol";
   7: import './Interfaces/IStabilityPool.sol';
   8: import './Interfaces/IBorrowerOperations.sol';
   9: import './Interfaces/ITroveManager.sol';
  10: import './Interfaces/ILUSDToken.sol';
  11: import './Interfaces/ISortedTroves.sol';
  12: import "./Interfaces/ICommunityIssuance.sol";
  13: import "./Dependencies/LiquityBase.sol";
  14: import "./Dependencies/SafeMath.sol";
  15: import "./Dependencies/LiquitySafeMath128.sol";
  16: import "./Dependencies/Ownable.sol";
  17: import "./Dependencies/CheckContract.sol";
  18: import "./Dependencies/console.sol";
  19: import "./Dependencies/SafeERC20.sol";
  20  

Ethos-Core/contracts/TroveManager.sol:
   4  
   5: import "./Interfaces/ICollateralConfig.sol";
   6: import "./Interfaces/ITroveManager.sol";
   7: import "./Interfaces/IStabilityPool.sol";
   8: import "./Interfaces/ICollSurplusPool.sol";
   9: import "./Interfaces/ILUSDToken.sol";
  10: import "./Interfaces/ISortedTroves.sol";
  11: import "./Interfaces/ILQTYStaking.sol";
  12: import "./Interfaces/IRedemptionHelper.sol";
  13: import "./Dependencies/LiquityBase.sol";
  14: // import "./Dependencies/Ownable.sol";
  15: import "./Dependencies/CheckContract.sol";
  16: import "./Dependencies/IERC20.sol";
  17  

Ethos-Core/contracts/LQTY/CommunityIssuance.sol:
   4  
   5: import "../Dependencies/IERC20.sol";
   6: import "../Interfaces/ICommunityIssuance.sol";
   7: import "../Dependencies/BaseMath.sol";
   8: import "../Dependencies/LiquityMath.sol";
   9: import "../Dependencies/Ownable.sol";
  10: import "../Dependencies/CheckContract.sol";
  11: import "../Dependencies/SafeMath.sol";
  12  

Ethos-Core/contracts/LQTY/LQTYStaking.sol:
   4  
   5: import "../Dependencies/BaseMath.sol";
   6: import "../Dependencies/SafeMath.sol";
   7: import "../Dependencies/Ownable.sol";
   8: import "../Dependencies/CheckContract.sol";
   9: import "../Dependencies/console.sol";
  10: import "../Dependencies/IERC20.sol";
  11: import "../Interfaces/ICollateralConfig.sol";
  12: import "../Interfaces/ILQTYStaking.sol";
  13: import "../Interfaces/ITroveManager.sol";
  14: import "../Dependencies/LiquityMath.sol";
  15: import "../Interfaces/ILUSDToken.sol";
  16: import "../Dependencies/SafeERC20.sol";
  17  

Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol:
   4  
   5: import "./abstract/ReaperBaseStrategyv4.sol";
   6: import "./interfaces/IAToken.sol";
   7: import "./interfaces/IAaveProtocolDataProvider.sol";
   8: import "./interfaces/ILendingPool.sol";
   9: import "./interfaces/ILendingPoolAddressesProvider.sol";
  10: import "./interfaces/IRewardsController.sol";
  11: import "./libraries/ReaperMathUtils.sol";
  12: import "./mixins/VeloSolidMixin.sol";
  13: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
  14: import "@openzeppelin/contracts-upgradeable/utils/math/MathUpgradeable.sol";
  15  

Ethos-Vault/contracts/ReaperVaultERC4626.sol:
  4  
  5: import "./ReaperVaultV2.sol";
  6: import "./interfaces/IERC4626Functions.sol";
  7  

Ethos-Vault/contracts/ReaperVaultV2.sol:
   4  
   5: import "./interfaces/IERC4626Events.sol";
   6: import "./interfaces/IStrategy.sol";
   7: import "./libraries/ReaperMathUtils.sol";
   8: import "./mixins/ReaperAccessControl.sol";
   9: import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";
  10: import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
  11: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
  12: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
  13: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
  14: import "@openzeppelin/contracts/utils/math/Math.sol";
  15  

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
   4  
   5: import "../interfaces/IStrategy.sol";
   6: import "../interfaces/IVault.sol";
   7: import "../libraries/ReaperMathUtils.sol";
   8: import "../mixins/ReaperAccessControl.sol";
   9: import "@openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";
  10: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
  11: import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
  12: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
  13 

```
**Description:**
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it. 
This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

**Recommendation:**
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

### [N-23] Use `require` instead of `assert`

```solidity

20 results - 4 files

Ethos-Core/contracts/BorrowerOperations.sol:
  128:         assert(MIN_NET_DEBT > 0);
  197:         assert(vars.compositeDebt > 0);
  301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
  331:         assert(_collWithdrawal <= vars.coll); 

Ethos-Core/contracts/LUSDToken.sol:
  312:         assert(sender != address(0));
  313:         assert(recipient != address(0));
  321:         assert(account != address(0));
  329:         assert(account != address(0));
  337:         assert(owner != address(0));
  338:         assert(spender != address(0));

Ethos-Core/contracts/StabilityPool.sol:
  526:         assert(_debtToOffset <= _totalLUSDDeposits);
  551:         assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
  591:         assert(newP > 0);

Ethos-Core/contracts/TroveManager.sol:
   417:             assert(_LUSDInStabPool != 0);
  1224:             assert(totalStakesSnapshot[_collateral] > 0);
  1279:         assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
  1342:         assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
  1348:         assert(index <= idxLast);
  1414:         assert(newBaseRate > 0); // Base rate is always non-zero after redemption
  1489:         assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0

```

**Description:**
Assert should not be used except for tests, `require` should be used

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process's available gas instead of returning it, as request()/revert() did.

assert() and ruqire();
The big difference between the two is that the `assert()`function when false, uses up all the remaining gas and reverts all the changes made.
Meanwhile, a  `require()` function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay.
This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states "The Assert function generates an error of type Panic(uint256).
Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there's a mistake".


### [N-24] Implement some type of version counter that will be incremented automatically for contract upgrades

As part of the upgradeability of  Proxies , the contract can be upgraded multiple times, where it is a systematic approach to record the version of each upgrade.

```js
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
  189:     function _authorizeUpgrade(address) internal override {
  190:         _atLeastRole(DEFAULT_ADMIN_ROLE);
  191:         require(
  192:             upgradeProposalTime + UPGRADE_TIMELOCK < block.timestamp,
  193:             "Upgrade cooldown not initiated or still ongoing"
  194:         );
  195:         clearUpgradeCooldown();
  196:     }

```
I suggest implementing some kind of version counter that will be incremented automatically when you upgrade the contract.

**Recommendation:**
```diff

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:

+	uint256 public authorizeUpgradeCounter;

  189:     function _authorizeUpgrade(address) internal override {
  190:         _atLeastRole(DEFAULT_ADMIN_ROLE);
  191:         require(
  192:             upgradeProposalTime + UPGRADE_TIMELOCK < block.timestamp,
  193:             "Upgrade cooldown not initiated or still ongoing"
  194:         );
+	authorizeUpgradeCounter+=1;
  195:         clearUpgradeCooldown();
  196:     }

```

### [N-25] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values)

  This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

```solidity

18 results - 4 files

Ethos-Core/contracts/StabilityPool.sol:
  197:     uint public constant SCALE_FACTOR = 1e9; 

Ethos-Vault/contracts/ReaperVaultV2.sol:
  40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
  41:     uint256 public constant PERCENT_DIVISOR = 10000;
  42      uint256 public tvlCap;
  73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
  74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
  75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
  76:     bytes32 public constant ADMIN = keccak256("ADMIN");
  77  

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
  22  
  23:     uint256 public constant PERCENT_DIVISOR = 10_000;
  24:     uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF
  25:     uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;
  49:     bytes32 public constant KEEPER = keccak256("KEEPER");
  50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
  51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
  52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```
### [N-26] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does


This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Aseembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.


```solidity

Ethos-Core/contracts/LUSDToken.sol:
  297  
  298:     function _chainID() private pure returns (uint256 chainID) {
  299:         assembly {
  300:             chainID := chainid()
  301:         }
  302:     }
```

### [N-27] Highest risk must be specified in NatSpec comments and documentation


```solidity

Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
  189:     function _authorizeUpgrade(address) internal override {
  190:         _atLeastRole(DEFAULT_ADMIN_ROLE);
  191:         require(
  192:             upgradeProposalTime + UPGRADE_TIMELOCK < block.timestamp,
  193:             "Upgrade cooldown not initiated or still ongoing"
  194:         );
  195:         clearUpgradeCooldown();
  196:     }

```

**Description:**
Although the UUPS model has many advantages and the proposed proxy model is currently the UUPS model, there are some caveats we should be aware of before applying it to a real-world project.

One of the main attention: Since upgrades are done through the implementation contract with the help of the 
``` _authorizeUpgrade ``` method, there is a high risk of new implementations such as excluding the ``` _authorizeUpgrade ``` method, which can permanently kill the smart contract's ability to upgrade. Also, this pattern is somewhat complicated to implement compared to other proxy patterns.

**Recommendation:**
Therefore, this highest risk must be found in NatSpec comments and documents.

## [N-28] Include proxy contracts to Audit

The Proxy contract could not be seen in the checklist because it is an upgradable contract, only the implementation contracts are visible, I recommend including the Proxy contract in the audit for integrity in the audit.

### [S-01] Generate perfect code headers every time

**Description:**
I recommend using header for Solidity code layout and readability

https://github.com/transmissions11/headers

```js
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```
