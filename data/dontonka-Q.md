**ReaperAccessControl._cascadingAccessRoles** should be **pure** instead of **view** has it is not relying on the state variable of the contract and the same goes for the contract(s) that inherits this contract.

The code would be clearner if this 'require' statement would be at the top of the function https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L93