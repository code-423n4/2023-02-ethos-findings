1.
Title: Using bytes constant is more gas efficient

Reference: [Here](https://ethereum.stackexchange.com/questions/3795/why-do-solidity-examples-use-bytes32-type-instead-of-string)

Proof of Concept:
[BorrowerOperations.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21)
[ActivePool.sol#L30](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30)
[CommunityIssuance.sol#L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19)
[LQTYStaking.sol#L23](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23)

Recommended Mitigation Steps:
Change it to `bytes(1..32) constant`
________________________________________________________________________

2.
Title: Gas savings for using solidity 0.8.10

impact:
Use a solidity version of at least 0.8 to get default underflow/overflow checks, use a solidity version of at least 0.8.2 to get simple compiler automatic inlining Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value

Proof of Concept:
All contract in scope.
by using version 0.8+ we can avoid using library safeMath in [ActivePool.sol#L27](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L27) , [CommunityIssuance.sol#L15](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L15) , [LQTYStaking.sol#L20](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L20) , [LUSDToken.sol#L29](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L29)

Recommended Mitigation Steps:
Consider to upgrade pragma to at least 0.8.10.

Solidity 0.8.10 has a useful change which reduced gas costs of external calls
Reference: [here](https://blog.soliditylang.org/2021/11/09/solidity-0.8.10-release-announcement/)
______________________________________________________________________

3.
Title: function _getCompoundedDepositFromSnapshots() gas improvement on returning value

Proof of Concept:
[StabilityPool.sol#L744](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L744)

Recommended Mitigation Steps:
by set `compoundedDeposit` in returns L#735 and delete L#744 can save gas

```
function _getCompoundedDepositFromSnapshots(
        uint initialDeposit,
        Snapshots memory snapshots
    )
        internal
        view
        returns (uint compoundedDeposit) //@audit-info: set here
```
________________________________________________________________________

4.
Title: Struct optimization

Proof of Concept:
[StabilityPool.sol#L174-L176](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L174-L176)

Recommended Mitigation Steps:
Use elementary types or a user-defined type instead can save gas
________________________________________________________________________

5.
Title: Using `storage` instead of `memory` for structs/arrays saves gas

Proof of Concept:
[StabilityPool.sol#L663](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L663)
[StabilityPool.sol#L722](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L722)
________________________________________________________________________

6.
Title: `>=` is cheaper than `>`

Impact:
Strict inequalities (`>`) are more expensive than non-strict ones (`>=`). This is due to some supplementary checks (ISZERO, 3 gas)

Proof of Concept:
[CommunityIssuance.sol#L87](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L87)

Recommended Mitigation Steps:
Consider using `>=` instead of `>` to avoid some opcodes
________________________________________________________________________

7.
Title: Using multiple `require` instead `&&` can save gas

Proof of Concept:
[BorrowerOperations.sol#L653](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653)
[LUSDToken.sol#L347-L357](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347-L357)

Recommended Mitigation Steps:
Change to:

```
	require(_maxFeePercentage >= BORROWING_FEE_FLOOR,
                "Max fee percentage must be between 0.5% and 100%");
	require(_maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must be between 0.5% and 100%");
```
________________________________________________________________________

8.
Title: Expression for `constant` values such as a call to `keccak256()`, should use `immutable` rather than `constant`

Proof of Concept:
[ReaperVaultV2.sol#L73-L76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73-L76)

Recommended Mitigation Steps:
Change from `constant` to `immutable`
reference: [here](https://github.com/ethereum/solidity/issues/9232)
________________________________________________________________________

9.
Title: x += y costs more gas than x = x + y for state variables

Proof of Concept:
[ReaperVaultV2.sol#L194-L196](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194-L196)
[ReaperVaultV2.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L214)
[ReaperVaultV2.sol#L395-L396](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L395-L396)
________________________________________________________________________

10.
Title: Using `msg.sender` directly is more effective

Proof of Concept:
[ReaperVaultV2.sol#L226](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L226)

Recommended Mitigation Steps:
In function availableCapital() use `msg.sender` directly instead of caching it to `stratAddr`. replace all `stratAddr` with `msg.sender` and delete L#226
________________________________________________________________________