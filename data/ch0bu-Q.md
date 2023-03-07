# Non-critical

## 1. Use a more recent version of Solidity

- Use a solidity version of at least 0.8.0 to get overflow/underflow protection without `SafeMath`  
- Use a solidity version of at least 0.8.2 to get compiler automatic inlining  
- Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads  
- Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than `revert()/require()` strings  
- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value
- Use a solidity version of at least 0.8.12 to get `string.concat()` to be used instead of `abi.encodePacked(<str>,<str>)`
- Use a solidity version of at least 0.8.13 to get the ability to use `using for` with a list of free functions
- Use a solidity version of 0.8.15 where conditions necessary for inlining are relaxed. It shows significant dicrease in the bytecode size (and thus reduced deployment cost)
- Use a solidity version of 0.8.17 to prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero; More efficient overflow checks for multiplication

```
All contracts
```


## 2. Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. 
- If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified `(if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})`. 
- Empty `receive()`/`fallback()` payable functions that are not used, can be removed to save deployment gas.

```
61       constructor() initializer {}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
```
16	constructor(
17		address _token,
18		string memory _name,
19		string memory _symbol,
20		uint256 _tvlCap,
21		address _treasury,
22		address[] memory _strategists,
23		address[] memory _multisigRoles
24	) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol


## 3. Unused constructor

The constructor does nothing.

```
61       constructor() initializer {}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol


## 4. For modern and more readable code; update import usages

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

Recommendation
`import {contract1 , contract2} from "filename.sol";

```
All contracts
```


## 5. Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

```
71       function setAddresses(
125      function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
132      function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
138      function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
144      function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol
```
110      function setAddresses(
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol
```
61       function setAddresses
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
```
66       function setAddresses
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol
```
160      function setHarvestSteps(address[2][] calldata _newSteps) external {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
```
258      function setWithdrawalQueue(address[] calldata _withdrawalQueue) external {
603      function setEmergencyShutdown(bool _active) external {
617      function setLockedProfitDegradation(uint256 degradation) external {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol
```
261      function setAddresses(
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol
```
232      function setAddresses(
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol


## 6. Mark visibility of init(…)/initialize() functions as `external`

External instead of public would give more the sense of the `initialize(…)` functions to behave like a constructor (only called on deployment, so should only be called externally)

From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

It might cost a bit less gas to use external over public.

It is possible to override a function from external to public, but it is not possible to override a function from public to external

For above reasons you can change `initialize(…)` to external
```
62	function initialize(
63		address _vault,
64		address[] memory _strategists,
65		address[] memory _multisigRoles,
66		IAToken _gWant
67	) public initializer {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol


## 7. Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

Use (e.g. 1e6) rather than decimal literals (e.g. 1000000), for better code readability.

```
40       uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
125      lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol


## 8. Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant

```
49       bytes32 public constant KEEPER = keccak256("KEEPER");
50       bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51       bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52       bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
```
73       bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74       bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75       bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76       bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol


## 9. Avoid nested `if` statements

For better readability and analysis it is better to avoid nested `if` blocks.

```
118	if (emergencyExit) {
119		uint256 amountFreed = _liquidateAllPositions();
120		if (amountFreed < debt) {
121			roi = -int256(debt - amountFreed);
122		} else if (amountFreed > debt) {
123			roi = int256(amountFreed - debt);
124		}
126		repayment = debt;
127		if (roi < 0) {
128			repayment -= uint256(-roi);
129		}
130	} else {
131		(roi, repayment) = _harvestCore(debt);
132	}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
```
234	if (stratCurrentAllocation > stratMaxAllocation) {
235		return -int256(stratCurrentAllocation - stratMaxAllocation);
236	} else if (stratCurrentAllocation < stratMaxAllocation) {
237		uint256 vaultMaxAllocation = (totalAllocBPS * balance()) / PERCENT_DIVISOR;
238		uint256 vaultCurrentAllocation = totalAllocated;
240		if (vaultCurrentAllocation >= vaultMaxAllocation) {
241			return 0;
242		}
244		uint256 available = stratMaxAllocation - stratCurrentAllocation;
245		available = Math.min(available, vaultMaxAllocation - vaultCurrentAllocation);
246		available = Math.min(available, token.balanceOf(address(this)));
248		return int256(available);
249	} else {
250		return 0;
251	}
...
368	if (value > token.balanceOf(address(this))) {
369		uint256 totalLoss = 0;
370		uint256 queueLength = withdrawalQueue.length;
371		uint256 vaultBalance = 0;
372		for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
373			vaultBalance = token.balanceOf(address(this));
374			if (value <= vaultBalance) {
375			break;
376		}
378		address stratAddr = withdrawalQueue[i];
379		uint256 strategyBal = strategies[stratAddr].allocated;
380		if (strategyBal == 0) {
381			continue;
382		}
384		uint256 remaining = value - vaultBalance;
385		uint256 loss = IStrategy(stratAddr).withdraw(Math.min(remaining, strategyBal));
386		uint256 actualWithdrawn = token.balanceOf(address(this)) - vaultBalance;
388		// Withdrawer incurs any losses from withdrawing as reported by strat
389		if (loss != 0) {
390			value -= loss;
391			totalLoss += loss;
392			_reportLoss(stratAddr, loss);
393		}
395		strategies[stratAddr].allocated -= actualWithdrawn;
396		totalAllocated -= actualWithdrawn;
397	}
...
438	if (totalAllocBPS != 0) {
439		// reduce strat's allocBPS proportional to loss
440		uint256 bpsChange = Math.min((loss * totalAllocBPS) / totalAllocated, stratParams.allocBPS);
442		// If the loss is too small, bpsChange will be 0
443		if (bpsChange != 0) {
444			stratParams.allocBPS -= bpsChange;
445			totalAllocBPS -= bpsChange;
446		}
447	}
...
509	if (vars.available < 0) {
510		vars.debt = uint256(-vars.available);
511		vars.debtPayment = Math.min(vars.debt, _repayment);
513		if (vars.debtPayment != 0) {
514			strategy.allocated -= vars.debtPayment;
515			totalAllocated -= vars.debtPayment;
516			vars.debt -= vars.debtPayment; // tracked for return value
517		}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol
```
286	if (vars.profit != 0) {
287		// profit is ultimately (coll at hand) + (coll allocated to yield generator) - (recorded total coll Amount in pool)
288		vars.profit = IERC20(_collateral).balanceOf(address(this)).add(vars.yieldingAmount).sub(collAmount[_collateral]);
289		if (vars.profit != 0) {
290			// distribute to treasury, staking pool, and stability pool
291			vars.treasurySplit = vars.profit.mul(yieldSplitTreasury).div(10_000);
292			if (vars.treasurySplit != 0) {
293				IERC20(_collateral).safeTransfer(treasuryAddress, vars.treasurySplit);
294			}
296			vars.stakingSplit = vars.profit.mul(yieldSplitStaking).div(10_000);
297			if (vars.stakingSplit != 0) {
298				IERC20(_collateral).safeTransfer(lqtyStakingAddress, vars.stakingSplit);
299				ILQTYStaking(lqtyStakingAddress).increaseF_Collateral(_collateral, vars.stakingSplit);
300			}
302			vars.stabilityPoolSplit = vars.profit.sub(vars.treasurySplit.add(vars.stakingSplit));
303			if (vars.stabilityPoolSplit != 0) {
304				IERC20(_collateral).safeTransfer(stabilityPoolAddress, vars.stabilityPoolSplit);
305				IStabilityPool(stabilityPoolAddress).updateRewardSum(_collateral, vars.stabilityPoolSplit);   
306			}
307		}
308	}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol
```
615	if (!vars.backToNormalMode) {
616		// Break the loop if ICR is greater than MCR and Stability Pool is empty
617		if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { break; }
		...
...
814	if (!vars.backToNormalMode) {
816		// Skip this trove if ICR is greater than MCR and Stability Pool is empty
817		if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { continue; }
		...
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol
```
595	if (_isRecoveryMode) {
596		_requireNoCollWithdrawal(_collWithdrawal);
597		if (_isDebtIncrease) {
598			_requireICRisAboveCCR(_vars.newICR, _vars.collCCR);
599			_requireNewICRisAboveOldICR(_vars.newICR, _vars.oldICR);
600		}
		...
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol


## 10. NatSpec comments should be increased in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

Generaly contracts in scope do not have NatSpec or are not covered fully. There are a lot of comments for major functions, which is good, but good NatSpec should be applied.


## 11. Solidity compiler optimizations can be problematic

Protocol has enabled optional compiler optimizations in Solidity.

There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised.

High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe.

It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario

A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

Recommended Mitigation Steps
Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug.

Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.


## 12. `Function writing` that does not comply with the `Solidity Style Guide`


Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last


## 13. State variable visibility is not set

It is best practice to set visibility for state variables.

```
28	address stabilityPoolAddress;
30	address gasPoolAddress;
32	ICollSurplusPool collSurplusPool;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol
```
31	address gasPoolAddress;
33	ICollSurplusPool collSurplusPool;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol


# LOW

## 1. Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions

See the following link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

- https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps


https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol


## 2. `_safeMint()` should be used rather than `_mint()` wherever possible

`_mint()` is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. Both OpenZeppelin and solmate have versions of this function so that NFTs aren’t lost if they’re minted to contracts that cannot transfer them back out.

```
188      _mint(_account, _amount);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol
```
336      _mint(_receiver, shares);
467      _mint(treasury, shares);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol


## 3. Owner can renounce Ownership

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L26

Mitigation

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.


## 4. Using Vulnerable Version of Openzeppelin

Project is using Openzeppelin version 3.3.0, which has known vulnerabilities.

It is recommended to switch to the latest stable version of Openzeppelin contracts.

