# Multiplication between huge numbers may overflow

Overflow may occur in calculations like the following:
```
a * b / c
```
If an overflow occurs at the point of a * b, then the subsequent division becomes meaningless.

Since Solidity cannot handle decimals, it deals with large numbers instead. This increases the risk of overflow.

The following lines are particularly risky:

[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L334](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L334)

[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L421](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L421)

## Recommendation

It is recommended to use a library like FullMath in Uniswap.

[https://github.com/Uniswap/v3-core/blob/main/contracts/libraries/FullMath.sol](https://github.com/Uniswap/v3-core/blob/main/contracts/libraries/FullMath.sol)

The previous example can be rewritten as follows:
```
FullMath.mulDiv(a,b,c)
```
If overflow occurs with a * b, it will perform the division first. This makes it more secure.