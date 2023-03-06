1. ## Use a more recent version of solidity

- Use a solidity version of at least 0.8.0 to get overflow/underflow protection without `SafeMath`  
- Use a solidity version of at least 0.8.2 to get compiler automatic inlining  
- Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads  
- Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than `revert()/require()` strings  
- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value
- Use a solidity version of at least 0.8.12 to get `string.concat()` to be used instead of `abi.encodePacked(<str>,<str>)`
- Use a solidity version of at least 0.8.13 to get the ability to use `using for` with a list of free functions
- Use a solidity version of 0.8.15 where conditions necessary for inlining are relaxed. It shows significant dicrease in the bytecode size (and thus reduced deployment cost)
- Use a solidity version of 0.8.17 to prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero; More efficient overflow checks for multiplication

```
All contracts in scope
```

2. ## Using unchecked blocks to save gas - increments in for loop can be unchecked

If the team decides to migrate all contracts in scope to solidity ^0.8.0 then the majority of Solidity for loops increment a uint256 variable that starts at 0. These increment operations never need to be checked for overflow/underflow because the variable will never reach the max number of uint256 (will run out of gas long before that happens). The default overflow/underflow check wastes gas in every iteration of virtually every for loop.

This saves 30-40 gas per loop iteration.

e.g Let’s work with a sample loop below.
```
for(uint256 i; i < 10; i++){
//doSomething
}
```
can be written as shown below.
```
for(uint256 i; i < 10;) {
  // loop logic
  unchecked { i++; }
}
```

```
108      for(uint256 i = 0; i < numCollaterals; i++) {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol
```
56       for(uint256 i = 0; i < _collaterals.length; i++) {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol
```
206      for (uint i = 0; i < assets.length; i++) {
228      for (uint i = 0; i < collaterals.length; i++) {
240      for (uint i = 0; i < numCollaterals; i++) {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol
```
351      for (uint i = 0; i < numCollaterals; i++) {
397      for (uint i = 0; i < numCollaterals; i++) {
640      for (uint i = 0; i < assets.length; i++) {
810      for (uint i = 0; i < collaterals.length; i++) {
831      for (uint i = 0; i < collaterals.length; i++) {
859      for (uint i = 0; i < numCollaterals; i++) {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol
```
608      for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
690      for (vars.i = 0; vars.i < _n; vars.i++) {
808      for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
882      for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol


## 3. `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables

```
168      totalAllocBPS += _allocBPS;
194      totalAllocBPS -= strategies[_strategy].allocBPS;
196      totalAllocBPS += _allocBPS;
214      totalAllocBPS -= strategies[_strategy].allocBPS;
396      totalAllocated -= actualWithdrawn;
445      totalAllocBPS -= bpsChange;
452      totalAllocated -= loss;
521      totalAllocated += vars.credit;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol


## 4. Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. 
- If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified `(if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})`. 
- Empty `receive()`/`fallback()` payable functions that are not used, can be removed to save deployment gas.

```
61       constructor() initializer {}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
```
16	constructor(
17		address _token,
18		string memory _name,
19		string memory _symbol,
20		uint256 _tvlCap,
21		address _treasury,
22		address[] memory _strategists,
23		address[] memory _multisigRoles
24	) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol


## 5. Caching global variables is more expensive than using the actual variable

Use `msg.sender` instead of caching it.

```
226      address stratAddr = msg.sender;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol


## 6. Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

Use a larger size then downcast where needed.

```
uint128 can be seen on multiple occasions in StabilityPool.sol and TroveManager.sol contracts
```


## 7. Setting the constructor to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

```
All contracts
```

## 8. Use `bytes32` instead of `string`

Use `bytes32` instead of `string` to save gas whenever possible. `string` is a dynamic data structure and therefore is more gas consuming then `bytes32`.

```
30       string constant public NAME = "ActivePool";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol
```
21       string constant public NAME = "BorrowerOperations";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol
```
19       string constant public NAME = "CommunityIssuance";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
```
23       string constant public NAME = "LQTYStaking";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol
```
32       string constant internal _NAME = "LUSD Stablecoin";
33       string constant internal _SYMBOL = "LUSD";
34       string constant internal _VERSION = "1";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol
```
150      string constant public NAME = "StabilityPool";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol


## 9. Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand. There is a difference between `constant` variables and `immutable` variables, and they should each be used in their appropriate contexts. `constants` should be used for literal values written into the code, and `immutable` variables should be used for expressions, or values calculated in, or passed into the constructor.

```
49       bytes32 public constant KEEPER = keccak256("KEEPER");
50       bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51       bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52       bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
```
73       bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74       bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75       bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76       bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

