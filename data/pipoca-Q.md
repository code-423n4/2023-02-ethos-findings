QA Report - Ethos Reserve

#1) CollateralConfig.sol – duplicates can be initialized 
The initialize() function uses arrays to keep track of whitelisted collateral and does not check for duplicates.
Double-counting of collateral value: If the same collateral is added to the whitelist twice, it could result in double-counting of its value. This could lead to an overestimation of the system's total collateral value, which could result in incorrect calculations of the MCR and CCR.
Inconsistent collateral configuration: If the same collateral is added to the whitelist with different MCR and CCR values, it could result in inconsistent collateral configuration. This could lead to unexpected behavior in the system, such as triggering recovery mode at the wrong time or not triggering it when it should be triggered.
Impacted code: https://github.com/search?q=repo%3Acode-423n4/2023-02-ethos%20%20getAllowedCollaterals&type=code
Solution: modify https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#LL46C5-L76C6 to include the following between lines 57 and 58:
require(!addedCollaterals[collateral], "Duplicate collateral");
addedCollaterals[collateral] = true;

the new mapping can be declared before line 46:
mapping(address => bool) memory addedCollaterals;

#2) TroveManager.sol – Use native time units when dealing with time
when working with time-based calculations in Solidity, it is important to use native time units rather than arbitrary units. This can make the code more readable and intuitive, and can also help prevent errors that might arise from using arbitrary units. In the context of the SECONDS_IN_ONE_MINUTE constant, using a native time unit like minutes can make the code more readable and reduce the risk of errors.
Impacted Code: line 48 = uint constant public SECONDS_IN_ONE_MINUTE = 60;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L48
Solution: uint constant public SECONDS_IN_ONE_MINUTE = 60 seconds;


#3) ActivePool.sol – Missing emit 
There are two functions increaseLUSDDebt and decreaseLUSDDebt that are missing the emit keyword when updating the ActivePoolLUSDDebtUpdated event. The ActivePoolLUSDDebtUpdated event is used to notify listeners about updates to the LUSD debt associated with a particular collateral asset. To correct this issue, add the emit keyword before the ActivePoolLUSDDebtUpdated event in each function
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L194
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L194

#4) ActivePool.sol – the setYieldingPercentage and setYieldingPercentageDrift functions should not be set independently + there should be flexibility to set their maximum amounts to be able to adapt to unforeseen situations.
The functions setYieldingPercentage and setYieldingPercentageDrift should not be set independently. This is because these two functions are interdependent and directly impact each other. Setting one function independently could potentially cause unexpected behavior and disrupt the overall functionality of the contract.
Furthermore, it is recommended that the contract provides flexibility to set the maximum amounts of these two functions. This would enable the contract to adapt to unforeseen situations, such as market fluctuations or unexpected events that could affect the yield percentage. Without this flexibility, the contract may be vulnerable to emergency situations where the yield percentage drops below the set YieldingPercentageDrift threshold. 
In the _rebalance function, the drift is used to determine whether the current allocation of funds in the contract needs to be adjusted. If the percentage of the final pool balance allocated to yielding is greater than the set YieldingPercentage plus the allowed YieldingPercentageDrift, the system will withdraw some funds to prevent overallocation. Conversely, if the percentage of the final pool balance allocated to yielding is less than the set YieldingPercentage minus the allowed YieldingPercentageDrift, the system will deposit more funds to prevent underallocation.

Solution:  it is essential to set the maximum drift as a function of the YieldingPercentage to ensure that the system can adapt to changing market conditions and maintain the desired allocation of funds. If the maximum drift is set too high or too low, the system may either overallocate or underallocate funds, resulting in significant losses or missed opportunities for profit. By setting the maximum drift as a function of the YieldingPercentage, the system ensures that the drift threshold is directly tied to the allocation of funds, which is critical to the system's stability and performance.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L125
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L132

#5) StabilityPool.sol –  Line 768 is not necessary as current mechanism already achieves the perceived intended result, even though it might not be the best solution(If time allows, will send separate report for more critical issue with scale changes)
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L768

#6) StabilityPool.Sol - LUSDLoss may underflow if compoundedLUSDDeposit is > than initialDeposit. 
 LUSDLoss may underflow if compoundedLUSDDeposit is > than initialDeposit. Moreover, if the only purpose is event logging, emit an event instead. 
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L339

#7) CommunityIssuance.sol - use safeTransferFrom instead of transferFrom on line 103 
The reason for using safeTransferFrom instead of transferFrom in the fund function of the OathToken contract is to prevent potential vulnerabilities related to the transfer of ERC20 tokens.
The transferFrom function is a standard ERC20 token function used to transfer tokens from one address to another. However, it does not have built-in safety checks to ensure that the receiving address can accept the tokens. This can lead to potential issues such as the receiving address being a contract with a default fallback function that reverts or locks up the tokens, resulting in permanent loss of funds.
On the other hand, the safeTransferFrom function is a safer alternative that performs additional safety checks before transferring the tokens. It first checks if the receiving address is a contract and if it implements the ERC20Receiver interface. If the receiving address is not a contract or does not implement the interface, the transfer will be reverted to prevent loss of funds.
Therefore, using safeTransferFrom instead of transferFrom can help prevent potential vulnerabilities related to the transfer of ERC20 tokens and ensure the safety of funds being transferred.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103

#8) CommunityIssuance.sol – sendOath function missing fundamental checks and event emission 
1.Check that _OathAmount is greater than zero to prevent a transfer of an invalid amount.
2.Check that the contract has enough Oath tokens available to transfer before proceeding with the transfer
3.Add an event to log the details of the transaction for auditing purposes.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L124

#9) CommunityIssuance.sol – Use safetransfer instead of transfer in sendOath() 
The function sendOath uses the transfer function to transfer tokens. It is recommended to use safeTransfer instead to avoid potential vulnerabilities in the event of an attacker exploiting a reentrant function call.
modify the fund function to use the safeTransfer or safeTransferFrom function to ensure secure token transfers and prevent vulnerabilities from potential reentrant attacks.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127

#10) LUSDToken.sol – Add ability to pause transfer in case of an emergency 
Adding the ability to pause transfers in the LUSDToken contract is an important safety feature that should be considered as it would help to prevent potential losses and damages during emergencies. By pausing transfers, the development team can quickly investigate and resolve critical security vulnerabilities or bugs that may lead to the loss of user funds.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

#11) ReaperBaseStartegyv4.sol –  PERCENT_DIVISOR variable declared but not used 
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L23

#12)ReaperBaseStartegyv4.sol – Consider adding minimum requirements for emergency mode to be triggered
Adding minimum requirements for the emergency mode to be triggered in ReaperBaseStrategyv4.sol can be a good practice to ensure that the mode is only triggered under specific conditions. This can help prevent false positives and reduce the risk of unnecessary liquidations that can impact the overall performance of the contract.

The specific requirements for the emergency mode will depend on the specifics of the contract and the underlying assets. However, some common requirements that can be considered are:

A threshold for the available capital: The emergency mode can be triggered only if the available capital in the contract falls below a certain threshold. This can help prevent the emergency mode from being triggered due to small fluctuations in the contract balance.

A time threshold: The emergency mode can be triggered only if the available capital has been below the threshold for a certain period of time. This can help prevent the emergency mode from being triggered due to temporary market fluctuations or network congestion.

An external trigger: The emergency mode can be triggered only if an external trigger, such as a governance vote or a multi-sig decision, is executed. This can help ensure that the emergency mode is triggered only in extreme cases and with the agreement of the contract owners.

Adding these requirements to the emergencyExit() function can help ensure that the emergency mode is triggered only when necessary and with the agreement of the contract owners. By defining specific requirements, the contract owners can also provide more transparency to the users and external contracts about the circumstances under which the emergency mode can be triggered.

#13 ReaperVaultStrategyv4.sol – Typo: Amount instead of Ammount (QA)
Line 98, require statement: Ammount must be less than balance
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L98
#14 ReaperStrategyGranarySupplyOnly.sol - Require without explanatory statement
When a require statement is used without an accompanying explanatory message, it can make it difficult for auditors or other developers to understand the intention behind the condition being checked. This can lead to confusion, mistakes, or security vulnerabilities, as the reason for the requirement is not clearly communicated.
Line 167: 
require(step[0] != address(0));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167
Line 168: 
require(step[1] != address(0));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L168

#14 TroveManager.Sol – Require without explanatory statement 
When a require statement is used without an accompanying explanatory message, it can make it difficult for auditors or other developers to understand the intention behind the condition being checked. This can lead to confusion, mistakes, or security vulnerabilities, as the reason for the requirement is not clearly communicated.
File: TroveManager.sol | Line: 250 | require(msg.sender == owner);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L250
File: TroveManager.sol | Line: 551 | require(totals.totalDebtInSequence > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L551
File: TroveManager.sol | Line: 716 | require(_troveArray.length != 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L716
File: TroveManager.sol | Line: 754 | require(totals.totalDebtInSequence > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754
File: TroveManager.sol | Line: 1450 | require(redemptionFee < _collateralDrawn);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1450
File: TroveManager.sol | Line: 1523 | require(msg.sender == borrowerOperationsAddress);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1523
File: TroveManager.sol | Line: 1527 | require(msg.sender == address(redemptionHelper));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1527
File: TroveManager.sol | Line: 1531 | require(msg.sender == borrowerOperationsAddress || msg.sender == address(redemptionHelper));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1531
File: TroveManager.sol | Line: 1535 | require(Troves[_borrower][_collateral].status == Status.active);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1535

#15 improper use of assert 
Instead of using assert(), custom errors (or requires, depending on the situation and design option) should be used to enhance the contract's robustness and facilitate error detection and handling. The current use of assert() may lead to a Panic error on failure, which hinders debugging and causes unexpected behavior. While assert() now uses the revert opcode and creates a Panic(uint256) error in the latest versions of Solidity, it should only be used to test for internal errors and check invariants, according to Solidity's documentation. 
Occurrences: 
File: BorrowerOperations.sol | Line: 128 | assert(MIN_NET_DEBT > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L128
File: BorrowerOperations.sol | Line: 197 | assert(vars.compositeDebt > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197
File: BorrowerOperations.sol | Line: 301 | assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L301
File: BorrowerOperations.sol | Line: 331 | assert(_collWithdrawal <= vars.coll);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L331
File: LUSDToken.sol | Line: 312 | assert(sender != address(0));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312
File: LUSDToken.sol | Line: 313 | assert(recipient != address(0));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L313
File: LUSDToken.sol | Line: 321 | assert(account != address(0));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L321
File: LUSDToken.sol | Line: 329 | assert(account != address(0));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L329
File: LUSDToken.sol | Line: 337 | assert(owner != address(0));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L337
File: LUSDToken.sol | Line: 338 | assert(spender != address(0));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L338
File: StabilityPool.sol | Line: 526 | assert(_debtToOffset <= _totalLUSDDeposits);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L526
File: StabilityPool.sol | Line: 551 | assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L551
File: StabilityPool.sol | Line: 591 | assert(newP > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L591
File: TroveManager.sol | Line: 417 | assert(_LUSDInStabPool != 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L417
File: TroveManager.sol | Line: 1224 | assert(totalStakesSnapshot[_collateral] > 0);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1224
File: TroveManager.sol | Line: 1279 | assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1279
File: TroveManager.sol | Line: 1342 | assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1342
File: TroveManager.sol | Line: 1348 | assert(index <= idxLast);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1348
File: TroveManager.sol | Line: 1414 | assert(newBaseRate > 0); // Base rate is always non-zero after redemption
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1414
File: TroveManager.sol | Line: 1489 | assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1489


#16 Better to use type(uint256).max – variable instead of uint256(-variable) 
Using type(uint256).max - vars.available is generally considered better than uint256(-vars.available) because it makes the intention clearer and avoids relying on implementation-specific details. type(uint256).max explicitly represents the maximum value of a uint256 type, which is guaranteed to be well-defined across different implementations and compilers. On the other hand, using - to negate a value and cast it to an unsigned integer relies on the implementation's two's complement representation of signed integers, which might not be consistent across all platforms or compilers.

Occurrences: 

ActivePool.sol | Line: 258 | vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L258
ActivePool.sol | Line: 282 | IERC4626(yieldGenerator[_collateral]).withdraw(uint256(-vars.netAssetMovement), address(this), address(this));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L282
ReaperBaseStrategyv4.sol | Line: 114 | debt = uint256(-availableCapital);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L114
ReaperBaseStrategyv4.sol | Line: 128 | repayment -= uint256(-roi);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L128
ReaperVaultV2.sol | Line: 500 | vars.loss = uint256(-_roi);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L500

ReaperVaultV2.sol | Line: 510 | vars.debt = uint256(-vars.available);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L510

#17 – Inconsistent use of uint vs uint 256 
While both uint and uint256 represent an unsigned integer, the use of either one should be consistent throughout the codebase.
Using uint instead of uint256 may not have any immediate negative impact on the code's functionality, but it could cause issues in the future if there are changes to the code or if different parts of the code are combined. This inconsistency could also make the code less readable and harder to maintain, as developers need to be aware of which integer type is being used at any given point
To ensure code readability, consistency, and future-proofing, it is recommended to use either uint or uint256 consistently throughout the codebase. The choice of which one to use can be based on personal preference or industry best practices.
Example occurrence: line 47 of BorrowerOperations.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L47

#18 Best Practices for Using Immutable Variables, Specifically for keccak256() Calls in Smart Contracts 
It is important to use the appropriate variable type when writing smart contracts to ensure security and efficiency. This audit finding highlights the inconsistent use of constant and immutable variables in contracts, particularly when making keccak256() calls. While both types may not affect gas savings, using the correct type of variable improves readability, reduces maintenance costs, and enhances contract security. Best practice dictates using immutable variables for expressions or values calculated in or passed into the constructor, while constant variables are suitable for literal values written into the code. As such, it is recommended to use immutable variables for constant values such as keccak256() calls to ensure consistency in code structure and security.
Occurrences:
File: LUSDToken.sol | Line: 120 | bytes32 hashedName = keccak256(bytes(_NAME));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L120
File: LUSDToken.sol | Line: 121 | bytes32 hashedVersion = keccak256(bytes(_VERSION));
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L121
File: LUSDToken.sol | Line: 283 | bytes32 digest = keccak256(abi.encodePacked('\x19\x01',
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L283
File: ReaperBaseStrategyv4.sol | Line: 49 | bytes32 public constant KEEPER = keccak256("KEEPER");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49
File: ReaperBaseStrategyv4.sol | Line: 50 | bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50
File: ReaperBaseStrategyv4.sol | Line: 51 | bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51
File: ReaperBaseStrategyv4.sol | Line: 52 | bytes32 public constant ADMIN = keccak256("ADMIN");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52
File: ReaperVaultV2.sol | Line: 73 | bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73
File: ReaperVaultV2.sol | Line: 74 | bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74
File: ReaperVaultV2.sol | Line: 75 | bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L75
File: ReaperVaultV2.sol | Line: 76 | bytes32 public constant ADMIN = keccak256("ADMIN");
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L76

#19 Best Practices for Using Immutable variables, Specifically for constructor variables that can not be modified 
Best practice dictates using immutable variables for expressions or values calculated in or passed into the constructor, while constant variables are suitable for literal values written into the code.
Occurrences:
File: CommunityIssuance.sol | Line: 57 | constructor() public {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L57
File: ReaperBaseStrategyv4.sol | Line: 61 | constructor() initializer {}
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L61
File: ReaperVaultERC4626.sol | Line: 16 | constructor(
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L16
File: ReaperVaultV2.sol | Line: 111 | constructor(
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111
File: TroveManager.sol | Line: 225 | constructor() public {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L225
#22 use bytes(x) instead of string to save gas 
By using bytes instead of strings for string variables, you can save gas in your smart contracts. This is because bytes are more efficient in terms of gas usage as they have a fixed size, whereas strings are dynamic and can change size during execution. Additionally, bytes are a more appropriate data type for certain operations, such as hashing and encryption.
Regarding the lower limit for bytes, it is generally not recommended to use less than 8 bytes for a variable. This is because it can cause alignment issues and lead to inefficient memory usage. It's important to keep in mind the tradeoff between gas usage and memory usage when deciding on the appropriate size for your variables.

Occurrences:
File: ActivePool.sol | Line: 30 | string constant public NAME = "ActivePool";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30
File: BorrowerOperations.sol | Line: 21 | string constant public NAME = "BorrowerOperations";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21
File: CommunityIssuance.sol | Line: 19 | string constant public NAME = "CommunityIssuance";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19
File: LQTYStaking.sol | Line: 23 | string constant public NAME = "LQTYStaking";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23
File: LUSDToken.sol | Line: 32 | string constant internal _NAME = "LUSD Stablecoin";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L32
File: LUSDToken.sol | Line: 33 | string constant internal _SYMBOL = "LUSD";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L33
File: LUSDToken.sol | Line: 34 | string constant internal _VERSION = "1";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L34
File: ReaperVaultERC4626.sol | Line: 18 | string memory _name,
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L18
File: ReaperVaultERC4626.sol | Line: 19 | string memory _symbol,
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L19
File: ReaperVaultV2.sol | Line: 113 | string memory _name,
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L113
File: ReaperVaultV2.sol | Line: 114 | string memory _symbol,
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L114
File: StabilityPool.sol | Line: 150 | string constant public NAME = "StabilityPool";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150
File: TroveManager.sol | Line: 19 | // string constant public NAME = "TroveManager";
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L19


#20 using outdated version of solidity inconsistently
The use of the latest version of Solidity is a critical component of secure and efficient smart contract development. Below are some reasons why the latest version should always be used consistently. 
Security: The latest version of Solidity includes security updates and bug fixes that improve the overall security of the contract. Using an outdated version of Solidity may expose the contract to known vulnerabilities and increase the risk of a successful attack.
Performance: The latest version of Solidity is typically optimized for performance, which means that contracts written in this version will run faster and consume less gas.
Compatibility: Using the latest version of Solidity ensures that the contract is compatible with the latest features and functionalities of the Ethereum network. This is important because older versions of Solidity may not be compatible with the latest network upgrades, which could result in unexpected behavior or errors.
Community support: Solidity is an open-source programming language, which means that it is constantly evolving with the help of a large community of developers. Using the latest version of Solidity ensures that the contract is built using the most up-to-date best practices and development techniques.
File: ActivePool.sol | Line: 3 | pragma solidity 0.6.11;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3
File: BorrowerOperations.sol | Line: 3 | pragma solidity 0.6.11;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3
File: CommunityIssuance.sol | Line: 3 | pragma solidity 0.6.11;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3
File: LQTYStaking.sol | Line: 3 | pragma solidity 0.6.11;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3
File: LUSDToken.sol | Line: 3 | pragma solidity 0.6.11;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3
File: ReaperBaseStrategyv4.sol | Line: 3 | pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/ReaperBaseStrategyv4.sol#L3
File: ReaperStrategyGranarySupplyOnly.sol | Line: 3 | pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3
File: ReaperVaultERC4626.sol | Line: 3 | pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3
File: ReaperVaultV2.sol | Line: 3 | pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3
File: StabilityPool.sol | Line: 3 | pragma solidity 0.6.11;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3
File: TroveManager.sol | Line: 3 | pragma solidity 0.6.11;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3