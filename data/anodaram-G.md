In Ethos-Core/contracts/LQTY/LQTYStaking.sol, these two lines (both have the same issue) need to check `LUSDGain` is non zero.
[Line 135](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L135) and [Line 171](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L171)

In case of no update on user snapshot, LUSDGain can be zero. Then in these lines, the smart contract processes the zero transaction which is a gas wasting. (This is not a critical issue)