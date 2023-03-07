QA/LOW Issues

### Title 1: Function NEVER USED _cascadingAccessRoles() 

#### Impact:
The contract declared internal functions but was not using them in any of the functions or contracts.
Since internal functions can only be called from inside the contracts, it makes no sense to have them if they are not used. This uses up gas and causes issues for auditors when understanding the contract logic.

#### PoC
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L659

#### Mitigation
Having dead code in the contracts uses up unnecessary gas and increases the complexity of the overall smart contract.
It is recommended to remove the internal functions from the contracts if they are never used.

### Title 2: Missing important Events

#### Impact:

Events are inheritable members of contracts. When you call them, they cause the arguments to be stored in the transaction’s log a special data structure in the blockchain.
These logs are associated with the address of the contract which can then be used by developers and auditors to keep track of the transactions.
The contract BorrowerOperations was found to be missing these events on the function claimCollateral which would make it difficult or impossible to track these transactions off-chain.

#### PoC
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L412

#### Mitigation
Consider emitting events for the functions mentioned above. It is also recommended to have the addresses indexed.

### Title 3: REQUIRE INSTEAD OF REVERT

#### Impact:

The require Solidity function guarantees the validity of the condition(s) passed as a parameter that cannot be detected before execution. It checks inputs, contract state variables, and return values from calls to external contracts.
Using require instead of revert improves the overall readability of the contract code.

#### PoC
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L279

#### Mitigation
Depending on the contract's validation checks, it is recommended to use the require function to handle input validations.

### Title 4: IN-LINE ASSEMBLY DETECTED

#### Impact:
Inline assembly is a way to access the Ethereum Virtual Machine at a low level. This bypasses several important safety features and checks of Solidity. This should only be used for tasks that need it and if there is confidence in using it.

Multiple vulnerabilities have been detected previously when the assembly is not properly used within the Solidity code; therefore, caution should be exercised while using them.

#### PoC
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L299

#### Mitigation

Avoid using inline assembly instructions if possible because it might introduce certain issues in the code if not dealt with properly because it bypasses several safety features that are already implemented in Solidity.

### Title 5: PRESENCE OF OVERPOWERED ROLE

#### Impact:
The overpowered owner (i.e., the person who has too much power) is a project design where the contract is tightly coupled to their owner (or owners); only they can manually invoke critical functions.
Due to the fact that this function is only accessible from a single address, the system is heavily dependent on the address of the owner. In this case, there are scenarios that may lead to undesirable consequences for investors, e.g., if the private key of this address is compromised, then an attacker can take control of the contract.

#### PoC
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L85
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L110

#### Mitigation
Contracts must be developed in a trust-less manner. For instance, this functionality can be implemented in the contract's constructor. Another option is to use a MultiSig wallet for this address. For systems that are provisioned for a single user, you can use [Ownable.sol].
For systems that require provisioning users in a group, you can use [@openzeppelin/Roles.sol] or [@hq20/Whitelist.sol].


### Title 6: BLOCK VALUES AS A PROXY FOR TIME
#### Impact:
Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp and block.number can be used to determine the current time or the time delta. However, they are not recommended for most use cases.
Due to variable block times, block.number should not be relied on for precise calculations of time.

#### PoC
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1501

#### Mitigation
It is recommended to use trusted external time sources, block numbers instead of timestamps, and/or utilizing multiple time sources to increase reliability. These practices can help mitigate risks of timestamp manipulation and inaccurate timing, increasing the reliability and security of the smart contract.


### Title 7: FUNCTION SHOULD RETURN STRUCT

#### Impact:
The function _getOffsetAndRedistributionVals was detected to be returning multiple values.
Consider using a struct instead of multiple return values for the function. It can improve code readability.

#### PoC
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L442

#### Mitigation
Use struct for returning multiple values inside a function, which returns several parameters and improves code readability.

### Title 8: GAS OPTIMIZATION FOR STATE VARIABLES
#### Impact:
Plus equals costs more gas than addition operator. Hence x +=y costs more gas than x = x + y.

#### PoC
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168

#### Mitigation

Please Consider

- addition operator over plus equals
- subtraction operator over minus equals
- division operator over divide equals
- multiplication operator over multiply equals
