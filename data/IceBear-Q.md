## Non Critical Issues


### [N-1] Use of block.timestamp
Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.
#### Recommended Mitigation Steps
Block timestamps should not be used for entropy or generating random numbers — i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use block.number and an average block time to estimate times; with a 10 second block time, 1 week equates to approximately, 60480 blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.

Instances where block.timestamp is used:

*Find (24) instance(s) in contracts*:
```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

87:             uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;

87:             uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;

93:         lastIssuanceTimestamp = block.timestamp;

113:         lastDistributionTime = block.timestamp.add(distributionPeriod);

114:         lastIssuanceTimestamp = block.timestamp;

```
[Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

128:         deploymentStartTime = block.timestamp;

```
[Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)

```solidity
File: Ethos-Core/contracts/TroveManager.sol

1501:         uint timePassed = block.timestamp.sub(lastFeeOperationTime);

1504:             lastFeeOperationTime = block.timestamp;

1505:             emit LastFeeOpTimeUpdated(block.timestamp);

1517:         return (block.timestamp.sub(lastFeeOperationTime)).div(SECONDS_IN_ONE_MINUTE);

```
[Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

26:         uint256 activation; // Activation block.timestamp

32:         uint256 lastReport; // block.timestamp of the last time a report occured

46:     uint256 public lastReport; // block.timestamp of last report from any strategy

121:         constructionTime = block.timestamp;

122:         lastReport = block.timestamp;

159:             activation: block.timestamp,

165:             lastReport: block.timestamp

419:         uint256 lockedFundsRatio = (block.timestamp - lastReport) * lockedProfitDegradation;

540:         strategy.lastReport = block.timestamp;

541:         lastReport = block.timestamp;

```
[Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

137:         lastHarvestTimestamp = block.timestamp;

169:         upgradeProposalTime = block.timestamp;

181:         upgradeProposalTime = block.timestamp + FUTURE_NEXT_PROPOSAL_TIME;

192:             upgradeProposalTime + UPGRADE_TIMELOCK < block.timestamp,

```
[Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)

### [N-2] Signature malleability of EVM’S ecrecover()
The function calls the Solidity ecrecover() function directly to verify the given signatures. However, the ecrecover() EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

  Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin’s ECDSA library.
  
  #### Recommended Mitigation Steps
  Use the ecrecover function from OpenZeppelin’s ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

*Find (1) instance(s) in contracts*:
```solidity
File: Ethos-Core/contracts/LUSDToken.sol

287:         address recoveredAddress = ecrecover(digest, v, r, s);

```
[Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)



### [N-3] Unused imports

*Find (8) instance(s) in contracts*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

15: import "./Dependencies/console.sol";

```
[Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

15: import "./Dependencies/console.sol";

```
[Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

8: import "../Dependencies/LiquityMath.sol";

```
[Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

9: import "../Dependencies/console.sol";

```
[Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

6: import "./Interfaces/ITroveManager.sol";

9: import "./Dependencies/console.sol";

```
[Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

18: import "./Dependencies/console.sol";

```
[Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

10: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

```
[Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)

### [N-4] Use require instead of assert
Assert should not be used except for tests, require should be used.

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

`assert() and reqire();`

The big difference between the two is that the assert()function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay.This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states “The Assert function generates an error of type Panic(uint256). Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there’s a mistake”.

*Find (20) instance(s) in contracts*:
```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

128:         assert(MIN_NET_DEBT > 0);

197:         assert(vars.compositeDebt > 0);

301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

331:         assert(_collWithdrawal <= vars.coll); 

```
[Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

312:         assert(sender != address(0));

313:         assert(recipient != address(0));

321:         assert(account != address(0));

329:         assert(account != address(0));

337:         assert(owner != address(0));

338:         assert(spender != address(0));

```
[Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

526:         assert(_debtToOffset <= _totalLUSDDeposits);

551:         assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);

591:         assert(newP > 0);

```
[Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Core/contracts/TroveManager.sol

417:             assert(_LUSDInStabPool != 0);

1224:             assert(totalStakesSnapshot[_collateral] > 0);

1279:         assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

1342:         assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

1348:         assert(index <= idxLast);

1414:         assert(newBaseRate > 0); // Base rate is always non-zero after redemption

1489:         assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0

```
[Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)



### [N-5] Constant values such as a call to keccak256(), should used to immutable rather than constant
There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.
  
Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

*Find (12) instance(s) in contracts*:
```solidity
File: Ethos-Core/contracts/TroveManager.sol

54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%

55:     uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%

```
[Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)


```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.

73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```
[Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

25:     uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;

49:     bytes32 public constant KEEPER = keccak256("KEEPER");

50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```
[Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)


### [N-6] Natspec comments


https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the tag, they do incomplete analysis.

#### Context
All Contracts.


#### Recommended Mitigation Steps
Include parameters in NatSpec comments

Recommendation Code Style:
```
		/// @notice information about what a function does
		/// @param pageId The id of the page to get the URI for.
		/// @return Returns a page's URI if it has been minted 
		function tokenURI(uint256 pageId) public view virtual override returns (string memory) {
				if (pageId == 0 || pageId > currentId) revert("NOT_MINTED");
				return string.concat(BASE_URI, pageId.toString());
		}
```

### [N-7] For modern and more readable code; update import usages

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

#### Context
All Contracts.

#### Recommendation: 
import {contract1 , contract2} from "filename.sol";

A good example from the ArtGobblers project;
```
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```
### [N-8] Large multiples of ten should use scientific notation rather than decimal literals, for readability
#### Recommendation
Scientific notation should be used for better code readability.
Use scientific notation (e.g. 1e18, 100000) rather than exponentiation (e.g. 10**18, 1e5)

*Find (4) instance(s) in contracts*:


```solidity
File: Ethos-Core/contracts/TroveManager.sol

54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%

```
[Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.

41:     uint256 public constant PERCENT_DIVISOR = 10000;

125:         lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks

```
[Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)



## Low Issues



### [L-1] Using older solidity version
Some contracts use an older solidity version which may contain security vulnerabilities. Using an older version could put your smart contracts at risk of being exploited by attackers.
    
#### Recommendation
Recommend locking pragmas to a specific Solidity version. Consider the compiler bugs in the following lists and ensure the contracts are not affected by them. It is also recommended to use the latest version of Solidity when deploying contracts.

Solidity compiler bugs: 
1. [Solidity repo - known bugs](https://github.com/ethereum/solidity/blob/develop/docs/bugs.json)
2. [Solidity repo - bugs by version](https://github.com/ethereum/solidity/blob/develop/docs/bugs_by_version.json)

*Find (8) instance(s) in contracts*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

3: pragma solidity 0.6.11;

```
[Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

3: pragma solidity 0.6.11;

```
[Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

3: pragma solidity 0.6.11;

```
[Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

3: pragma solidity 0.6.11;

```
[Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

3: pragma solidity 0.6.11;

```
[Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

3: pragma solidity 0.6.11;

```
[Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

3: pragma solidity 0.6.11;

```
[Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Core/contracts/TroveManager.sol

3: pragma solidity 0.6.11;

```
[Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)

### [L-2] Owner can renounce ownership
Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.
    
Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.
    
#### Recommendation
We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

*Find (20) instance(s) in contracts*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

13: import "./Dependencies/Ownable.sol";

83:         onlyOwner

125:     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

132:     function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138:     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144:     function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

214:     function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {

```
[Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

13: import "./Dependencies/Ownable.sol";

125:         onlyOwner

```
[Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

6: import "./Dependencies/Ownable.sol";

50:     ) external override onlyOwner {

89:     ) external onlyOwner checkCollateral(_collateral) {

```
[Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

9: import "../Dependencies/Ownable.sol";

67:         onlyOwner 

101:     function fund(uint amount) external onlyOwner {

120:     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {

```
[Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

7: import "../Dependencies/Ownable.sol";

76:         onlyOwner 

```
[Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

16: import "./Dependencies/Ownable.sol";

273:         onlyOwner

```
[Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)



### [L-3] Non-library/interface files should use fixed compiler versions, not floating ones

*Find (3) instance(s) in contracts*:
```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

3: pragma solidity ^0.8.0;

```
[Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

3: pragma solidity ^0.8.0;

```
[Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

3: pragma solidity ^0.8.0;

```
[Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)


### [L-4] Lack of input validation

For defence-in-depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid.
similar findings:
https://code4rena.com/reports/2022-11-redactedcartel/#l-11-lack-of-input-validation

*Find (4) instance(s) in contracts*:
```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

110: function deposit(uint256 assets, address receiver) external override returns (uint256 shares) {

154: function mint(uint256 shares, address receiver) external override returns (uint256 assets) {

202: function withdraw(

258: function redeem(
```
[Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)

### [L-5] Loss of precision due to rounding
#### Recommendation
Multiplication should always be performed before division to avoid loss of precision.

*Find (2) instance(s) in contracts*:
```solidity
File: Ethos-Core/contracts/TroveManager.sol

54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%

55:     uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%

```
[Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)



### [S-1] Generate perfect code headers every time

I recommend using header for Solidity code layout and readability

https://github.com/transmissions11/headers
