- Use `uint256 i;` instead of `uint256 i = 0;`
  If the uint (i) in any loop is initialized to zero, it shouldn't be explicit.

- Use `++i` instead of `i++`
   If there's no need to retrieve the original state of `i`, `++i`  should be used to save gas.

! The above findings can be found throughout the entire Echos repository.

! However, I highly recommend to start here: https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol
