#[L-1] INTERNAL FUNCTIONS NEVER USED

The contract declared internal functions but was not using them in any of the functions or contracts.
Since internal functions can only be called from inside the contracts, it makes no sense to have them if they are not used. This uses up gas and causes issues for auditors when understanding the contract logic.

Line nos

[BorrowerOperations.sol#L432-L436](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L432-L436)

Having dead code in the contracts uses up unnecessary gas and increases the complexity of the overall smart contract.
It is recommended to remove the internal functions from the contracts if they are never used.

#[L-2] MISSING EVENTS

Events are inheritable members of contracts. When you call them, they cause the arguments to be stored in the transaction’s log — a special data structure in the blockchain.
These logs are associated with the address of the contract which can then be used by developers and auditors to keep track of the transactions.

[ActivePool.sol#L190-L195](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L190-L195)

[ActivePool.sol#L197-L202](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L197-L202)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L516-L519

[ActivePool.sol#L214-L217](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L214-L217)

[ActivePool.sol#L239-L309](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L239-L309)

[BorrowerOperations.sol#L412-L415](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L412-L415)

[BorrowerOperations.sol#L418-L430](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L418-L430)

[BorrowerOperations.sol#L454-L473](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L454-L473)

[BorrowerOperations.sol#L475-L501](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L475-L501)

[BorrowerOperations.sol#L505-L508](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L505-L508)

[BorrowerOperations.sol#L510-L514](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L510-L514)

[BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L516-L519)

[StabilityPool.sol#L439-L457](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L439-L457)

[StabilityPool.sol#L504-L544](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L504-L544)

[StabilityPool.sol#L856-L867](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L856-L867)

[TroveManager.sol#L915-L923](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L915-L923)

[TroveManager.sol#L926-L930](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L926-L930)

[TroveManager.sol#L957-L960](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L957-L960)

[TroveManager.sol#L1016-L1040](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1016-L1040)

[TroveManager.sol#L1183-L1186](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1183-L1186)

[TroveManager.sol#L1189-L1193](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1189-L1193)

[TroveManager.sol#L1316-L1319](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1316-L1319)

[TroveManager.sol#L1321-L1333](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1321-L1333)

[TroveManager.sol#L1562-L1565](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1562-L1565)

[TroveManager.sol#L1567-L1572](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1567-L1572)

[TroveManager.sol#L1574-L1579](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1574-L1579)

[TroveManager.sol#L1581-L1586](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1581-L1586)

[TroveManager.sol#L1588-L1593](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1588-L1593)


Consider emitting events for the functions mentioned above. It is also recommended to have the addresses indexed.


