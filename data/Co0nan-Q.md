The _updateDepositAndSnapshots() function in SP contract deletes a depositSnapshots struct, but the mapping in the Snapshots struct is not deleted. Values from this “deleted” mapping can still be accessed and there are no comments, function names, or other indicators that this is known. This issue is documented in Solidity’s security considerations documentation:
https://docs.soliditylang.org/en/v0.8.10/security-considerations.html#clearing-mappings

The delete operation occurs on 
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L813

The mapping that still exists after the delete on:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L179