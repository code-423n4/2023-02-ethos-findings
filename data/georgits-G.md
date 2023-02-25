## Don't initialize variables with default value
CollateralConfig.sol - [18](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L18)

LUSDToken.sol - [37](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L37)

CommunityIssuance.sol - [21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L21)

ActivePool.sol - [32](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L32)

## Splitting `require()` statements that use `&&` saves gas
LUSDToken.sol - [347-351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347-L351), [352-357](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L352-L357)

PriceFeed.sol = [131](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L131), [158](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L158)

TroveManager.sol - [1539](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539)


