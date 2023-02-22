# Gas Report

## `abi.encodePacked` is more gas efficient than `abi.encode`

`abi.encode` pads all elementary types to 32 bytes, whereas `abi.encodePacked` will only use the minimal required memory to encode the data. See [here](https://docs.soliditylang.org/en/v0.8.11/abi-spec.html?highlight=encodepacked#non-standard-packed-mode) for more info.

Instances: 2
- [Ethos-Core/contracts/LUSDToken.sol#L284](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L284)
- [Ethos-Core/contracts/LUSDToken.sol#L305](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L305)

<br>

## Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost (2400 per instance).

Instances: 14
- [Ethos-Core/contracts/CollateralConfig.sol#L47](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L47)
- [Ethos-Core/contracts/CollateralConfig.sol#L86](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L86)
- [Ethos-Core/contracts/BorrowerOperations.sol#L110](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L110)
- [Ethos-Core/contracts/ActivePool.sol#L71](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71)
- [Ethos-Core/contracts/ActivePool.sol#L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L125)
- [Ethos-Core/contracts/ActivePool.sol#L132](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L132)
- [Ethos-Core/contracts/ActivePool.sol#L138](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L138)
- [Ethos-Core/contracts/ActivePool.sol#L144](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144)
- [Ethos-Core/contracts/ActivePool.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L214)
- [Ethos-Core/contracts/StabilityPool.sol#L261](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L261)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66)

<br>

## Use assembly to check for `address(0)`

Saves 16000 deployment gas per instance and 6 runtime gas per instance.

```
assembly {
 if iszero(_addr) {
  mstore(0x00, "zero address")
  revert(0x00, 0x20)
 }
}
```

Instances: 12
- [Ethos-Core/contracts/ActivePool.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L93)
- [Ethos-Core/contracts/LUSDToken.sol#L312](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312)
- [Ethos-Core/contracts/LUSDToken.sol#L313](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L313)
- [Ethos-Core/contracts/LUSDToken.sol#L321](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L321)
- [Ethos-Core/contracts/LUSDToken.sol#L329](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L329)
- [Ethos-Core/contracts/LUSDToken.sol#L337](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L337)
- [Ethos-Core/contracts/LUSDToken.sol#L338](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L338)
- [Ethos-Core/contracts/LUSDToken.sol#L348](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L348)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L151](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L151)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L629](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L629)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L168](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L168)

<br>

## Use assembly to calculate hashes

Saves 5000 deployment gas per instance and 80 runtime gas per instance.

### Unoptimized
```
function solidityHash(uint256 a, uint256 b) public view {
	//unoptimized
	keccak256(abi.encodePacked(a, b));
}
```

### Optimized
```
function assemblyHash(uint256 a, uint256 b) public view {
	//optimized
	assembly {
		mstore(0x00, a)
		mstore(0x20, b)
		let hashedVal := keccak256(0x00, 0x40)
	}
}
```

Instances: 13
- [Ethos-Core/contracts/LUSDToken.sol#L120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L120)
- [Ethos-Core/contracts/LUSDToken.sol#L121](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L121)
- [Ethos-Core/contracts/LUSDToken.sol#L283](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L283)
- [Ethos-Core/contracts/LUSDToken.sol#L284](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L284)
- [Ethos-Core/contracts/LUSDToken.sol#L305](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L305)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L75](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L75)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L76)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52)

<br>

## Cache storage variables rather than re-reading from storage

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

Instances: 25
- [Ethos-Core/contracts/ActivePool.sol#L179](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L179)
- [Ethos-Core/contracts/ActivePool.sol#L180](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L180)
- [Ethos-Core/contracts/ActivePool.sol#L181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L181)
- [Ethos-Core/contracts/ActivePool.sol#L182](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L182)
- [Ethos-Core/contracts/ActivePool.sol#L183](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L183)
- [Ethos-Core/contracts/ActivePool.sol#L184](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L184)
- [Ethos-Core/contracts/ActivePool.sol#L264](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L264)
- [Ethos-Core/contracts/ActivePool.sol#L269](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L269)
- [Ethos-Core/contracts/ActivePool.sol#L298](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L298)
- [Ethos-Core/contracts/ActivePool.sol#L299](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L299)
- [Ethos-Core/contracts/ActivePool.sol#L304](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L304)
- [Ethos-Core/contracts/ActivePool.sol#L305](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L305)
- [Ethos-Core/contracts/ActivePool.sol#L328](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L328)
- [Ethos-Core/contracts/ActivePool.sol#L331](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L331)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L86](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L86)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L87](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L87)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L88](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L88)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L106](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L106)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L107](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L107)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L112](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L112)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L113](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L113)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L252](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L252)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L254](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L254)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L111](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L111)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L134](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L134)

<br>

## Use `calldata` instead of `memory` for function arguments that are read only

Instances: 11
- [Ethos-Core/contracts/TroveManager.sol#L715](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L715)
- [Ethos-Core/contracts/TroveManager.sol#L792](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L792)
- [Ethos-Core/contracts/TroveManager.sol#L872](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L872)
- [Ethos-Core/contracts/TroveManager.sol#L898](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L898)
- [Ethos-Core/contracts/StabilityPool.sol#L693](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L693)
- [Ethos-Core/contracts/StabilityPool.sol#L731](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L731)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L238](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L238)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L66)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L67](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L67)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L64](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L64)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L65](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L65)

<br>

## Initializing variable to default value costs unnecessary gas

Instances: 10
- [Ethos-Core/contracts/CollateralConfig.sol#L18](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L18)
- [Ethos-Core/contracts/ActivePool.sol#L32](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L32)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L21)
- [Ethos-Core/contracts/LUSDToken.sol#L37](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L37)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L369](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L369)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L371](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L371)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L100](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L100)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L112](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L112)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L117](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L117)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35)

<br>

## Duplicated `require`/`revert` checks should be refactored to a modifier or function.

Instances: 6
- [Ethos-Core/contracts/TroveManager.sol#L551](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L551)
- [Ethos-Core/contracts/TroveManager.sol#L754](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L155](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L181)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L180](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L180)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L193](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L193)

<br>

## `require`/`revert` strings longer than 32 bytes cost extra gas

Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met. If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

Instances: 35
- [Ethos-Core/contracts/BorrowerOperations.sol#L525](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L525)
- [Ethos-Core/contracts/BorrowerOperations.sol#L529](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L529)
- [Ethos-Core/contracts/BorrowerOperations.sol#L530](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L530)
- [Ethos-Core/contracts/BorrowerOperations.sol#L534](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L534)
- [Ethos-Core/contracts/BorrowerOperations.sol#L538](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L538)
- [Ethos-Core/contracts/BorrowerOperations.sol#L543](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L543)
- [Ethos-Core/contracts/BorrowerOperations.sol#L552](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L552)
- [Ethos-Core/contracts/BorrowerOperations.sol#L568](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L568)
- [Ethos-Core/contracts/BorrowerOperations.sol#L617](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L617)
- [Ethos-Core/contracts/BorrowerOperations.sol#L621](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L621)
- [Ethos-Core/contracts/BorrowerOperations.sol#L625](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L625)
- [Ethos-Core/contracts/BorrowerOperations.sol#L629](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L629)
- [Ethos-Core/contracts/BorrowerOperations.sol#L637](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L637)
- [Ethos-Core/contracts/BorrowerOperations.sol#L641](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L641)
- [Ethos-Core/contracts/BorrowerOperations.sol#L645](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L645)
- [Ethos-Core/contracts/ActivePool.sol#L107](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L107)
- [Ethos-Core/contracts/ActivePool.sol#L133](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L133)
- [Ethos-Core/contracts/StabilityPool.sol#L849](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L849)
- [Ethos-Core/contracts/StabilityPool.sol#L853](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L853)
- [Ethos-Core/contracts/StabilityPool.sol#L865](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L865)
- [Ethos-Core/contracts/StabilityPool.sol#L870](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L870)
- [Ethos-Core/contracts/StabilityPool.sol#L874](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L874)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L133](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L133)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L262](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L262)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L266](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L266)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L270](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L270)
- [Ethos-Core/contracts/LUSDToken.sol#L362](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L362)
- [Ethos-Core/contracts/LUSDToken.sol#L377](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L377)
- [Ethos-Core/contracts/LUSDToken.sol#L390](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L390)
- [Ethos-Core/contracts/LUSDToken.sol#L394](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L394)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L150](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L150)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L321](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L321)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L436](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L436)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L619](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L619)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L98](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L98)

<br>

## Use `indexed` to save gas
Using `indexed` for value type event parameters (address, uint, bool) saves at least 80 gas in emitting the event (https://gist.github.com/Tomosuke0930/9bf61e01a8c3e214d95a9b84dcb41d97).

Note that for other types however (string, bytes), it is more expensive.

Instances: 113
- [Ethos-Core/contracts/CollateralConfig.sol#L37](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L37)
- [Ethos-Core/contracts/CollateralConfig.sol#L38](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L38)
- [Ethos-Core/contracts/BorrowerOperations.sol#L92](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L92)
- [Ethos-Core/contracts/BorrowerOperations.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L93)
- [Ethos-Core/contracts/BorrowerOperations.sol#L94](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L94)
- [Ethos-Core/contracts/BorrowerOperations.sol#L95](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L95)
- [Ethos-Core/contracts/BorrowerOperations.sol#L96](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L96)
- [Ethos-Core/contracts/BorrowerOperations.sol#L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L97)
- [Ethos-Core/contracts/BorrowerOperations.sol#L98](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L98)
- [Ethos-Core/contracts/BorrowerOperations.sol#L99](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L99)
- [Ethos-Core/contracts/BorrowerOperations.sol#L100](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L100)
- [Ethos-Core/contracts/BorrowerOperations.sol#L101](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L101)
- [Ethos-Core/contracts/BorrowerOperations.sol#L102](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L102)
- [Ethos-Core/contracts/BorrowerOperations.sol#L104](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L104)
- [Ethos-Core/contracts/BorrowerOperations.sol#L105](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L105)
- [Ethos-Core/contracts/BorrowerOperations.sol#L106](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L106)
- [Ethos-Core/contracts/TroveManager.sol#L186](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L186)
- [Ethos-Core/contracts/TroveManager.sol#L187](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L187)
- [Ethos-Core/contracts/TroveManager.sol#L188](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L188)
- [Ethos-Core/contracts/TroveManager.sol#L189](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L189)
- [Ethos-Core/contracts/TroveManager.sol#L190](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L190)
- [Ethos-Core/contracts/TroveManager.sol#L191](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L191)
- [Ethos-Core/contracts/TroveManager.sol#L192](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L192)
- [Ethos-Core/contracts/TroveManager.sol#L193](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L193)
- [Ethos-Core/contracts/TroveManager.sol#L194](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L194)
- [Ethos-Core/contracts/TroveManager.sol#L195](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L195)
- [Ethos-Core/contracts/TroveManager.sol#L196](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L196)
- [Ethos-Core/contracts/TroveManager.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L197)
- [Ethos-Core/contracts/TroveManager.sol#L198](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L198)
- [Ethos-Core/contracts/TroveManager.sol#L200](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L200)
- [Ethos-Core/contracts/TroveManager.sol#L201](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L201)
- [Ethos-Core/contracts/TroveManager.sol#L202](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L202)
- [Ethos-Core/contracts/TroveManager.sol#L203](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L203)
- [Ethos-Core/contracts/TroveManager.sol#L204](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L204)
- [Ethos-Core/contracts/TroveManager.sol#L205](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L205)
- [Ethos-Core/contracts/TroveManager.sol#L206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L206)
- [Ethos-Core/contracts/TroveManager.sol#L207](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L207)
- [Ethos-Core/contracts/TroveManager.sol#L208](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L208)
- [Ethos-Core/contracts/TroveManager.sol#L209](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L209)
- [Ethos-Core/contracts/TroveManager.sol#L211](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L211)
- [Ethos-Core/contracts/TroveManager.sol#L212](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L212)
- [Ethos-Core/contracts/TroveManager.sol#L213](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L213)
- [Ethos-Core/contracts/TroveManager.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L214)
- [Ethos-Core/contracts/TroveManager.sol#L215](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L215)
- [Ethos-Core/contracts/ActivePool.sol#L58](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L58)
- [Ethos-Core/contracts/ActivePool.sol#L59](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L59)
- [Ethos-Core/contracts/ActivePool.sol#L60](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L60)
- [Ethos-Core/contracts/ActivePool.sol#L61](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L61)
- [Ethos-Core/contracts/ActivePool.sol#L62](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L62)
- [Ethos-Core/contracts/ActivePool.sol#L63](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L63)
- [Ethos-Core/contracts/ActivePool.sol#L64](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L64)
- [Ethos-Core/contracts/ActivePool.sol#L65](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L65)
- [Ethos-Core/contracts/ActivePool.sol#L66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L66)
- [Ethos-Core/contracts/ActivePool.sol#L67](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L67)
- [Ethos-Core/contracts/StabilityPool.sol#L233](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L233)
- [Ethos-Core/contracts/StabilityPool.sol#L234](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L234)
- [Ethos-Core/contracts/StabilityPool.sol#L236](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L236)
- [Ethos-Core/contracts/StabilityPool.sol#L237](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L237)
- [Ethos-Core/contracts/StabilityPool.sol#L238](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L238)
- [Ethos-Core/contracts/StabilityPool.sol#L239](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L239)
- [Ethos-Core/contracts/StabilityPool.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L240)
- [Ethos-Core/contracts/StabilityPool.sol#L241](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L241)
- [Ethos-Core/contracts/StabilityPool.sol#L242](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L242)
- [Ethos-Core/contracts/StabilityPool.sol#L243](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L243)
- [Ethos-Core/contracts/StabilityPool.sol#L244](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L244)
- [Ethos-Core/contracts/StabilityPool.sol#L246](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L246)
- [Ethos-Core/contracts/StabilityPool.sol#L247](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L247)
- [Ethos-Core/contracts/StabilityPool.sol#L248](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L248)
- [Ethos-Core/contracts/StabilityPool.sol#L249](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L249)
- [Ethos-Core/contracts/StabilityPool.sol#L250](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L250)
- [Ethos-Core/contracts/StabilityPool.sol#L252](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L252)
- [Ethos-Core/contracts/StabilityPool.sol#L253](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L253)
- [Ethos-Core/contracts/StabilityPool.sol#L255](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L255)
- [Ethos-Core/contracts/StabilityPool.sol#L256](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L256)
- [Ethos-Core/contracts/StabilityPool.sol#L257](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L257)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L50)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L51)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L52)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L53](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L53)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L49](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L49)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L50)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L51)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L52)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L53](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L53)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L54](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L54)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L56](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L56)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L57](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L57)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L58](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L58)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L59](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L59)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L60](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L60)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L61](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L61)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L62](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L62)
- [Ethos-Core/contracts/LUSDToken.sol#L78](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L78)
- [Ethos-Core/contracts/LUSDToken.sol#L79](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L79)
- [Ethos-Core/contracts/LUSDToken.sol#L80](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L80)
- [Ethos-Core/contracts/LUSDToken.sol#L81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L81)
- [Ethos-Core/contracts/LUSDToken.sol#L82](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L82)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L80](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L80)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L81)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L82](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L82)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L85](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L85)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L86](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L86)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L87](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L87)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L88](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L88)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L89](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L89)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L92](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L92)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L93)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L94](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L94)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L95](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L95)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L96](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L96)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L97)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L98](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L98)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L99](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L99)

<br>

## Use `constant`, `immutable` where applicable

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD. Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas).

Instances: 1
- [Ethos-Core/contracts/StabilityPool.sol#L160](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L160)

<br>

## Inline `internal` functions that are only called once

Saves 20-40 gas per instance. See https://blog.soliditylang.org/2021/03/02/saving-gas-with-simple-inliner/ for more details.

Instances: 45
- [Ethos-Core/contracts/BorrowerOperations.sol#L173](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L173)
- [Ethos-Core/contracts/BorrowerOperations.sol#L296](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L296)
- [Ethos-Core/contracts/BorrowerOperations.sol#L297](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L297)
- [Ethos-Core/contracts/BorrowerOperations.sol#L185](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L185)
- [Ethos-Core/contracts/BorrowerOperations.sol#L294](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L294)
- [Ethos-Core/contracts/BorrowerOperations.sol#L596](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L596)
- [Ethos-Core/contracts/BorrowerOperations.sol#L599](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L599)
- [Ethos-Core/contracts/BorrowerOperations.sol#L339](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L339)
- [Ethos-Core/contracts/BorrowerOperations.sol#L249](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L249)
- [Ethos-Core/contracts/TroveManager.sol#L1078](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1078)
- [Ethos-Core/contracts/TroveManager.sol#L1202](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1202)
- [Ethos-Core/contracts/TroveManager.sol#L1318](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1318)
- [Ethos-Core/contracts/TroveManager.sol#L1291](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1291)
- [Ethos-Core/contracts/TroveManager.sol#L1510](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1510)
- [Ethos-Core/contracts/TroveManager.sol#L1282](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1282)
- [Ethos-Core/contracts/ActivePool.sol#L206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L206)
- [Ethos-Core/contracts/ActivePool.sol#L192](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L192)
- [Ethos-Core/contracts/StabilityPool.sol#L418](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L418)
- [Ethos-Core/contracts/StabilityPool.sol#L478](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L478)
- [Ethos-Core/contracts/StabilityPool.sol#L632](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L632)
- [Ethos-Core/contracts/StabilityPool.sol#L641](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L641)
- [Ethos-Core/contracts/StabilityPool.sol#L688](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L688)
- [Ethos-Core/contracts/StabilityPool.sol#L344](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L344)
- [Ethos-Core/contracts/StabilityPool.sol#L389](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L389)
- [Ethos-Core/contracts/StabilityPool.sol#L487](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L487)
- [Ethos-Core/contracts/StabilityPool.sol#L467](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L467)
- [Ethos-Core/contracts/StabilityPool.sol#L370](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L370)
- [Ethos-Core/contracts/StabilityPool.sol#L324](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L324)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L178](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L178)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L188](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L188)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L144](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L144)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L105](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L105)
- [Ethos-Core/contracts/LUSDToken.sol#L188](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L188)
- [Ethos-Core/contracts/LUSDToken.sol#L193](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L193)
- [Ethos-Core/contracts/LUSDToken.sol#L187](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L187)
- [Ethos-Core/contracts/LUSDToken.sol#L192](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L192)
- [Ethos-Core/contracts/LUSDToken.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L197)
- [Ethos-Core/contracts/LUSDToken.sol#L202](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L202)
- [Ethos-Core/contracts/LUSDToken.sol#L186](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L186)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L504](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L504)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L135](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L135)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L119](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L119)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L82](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L82)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L93)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L115](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L115)

<br>

## Expressions for constant values such as a call to `keccak256` should use `immutable` rather than `constant`

See [this](https://github.com/ethereum/solidity/issues/9232) issue for a detail description of the issue.

Instances: 8
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L75](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L75)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L76)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52)

<br>

## Use `unchecked` for operations that cannot overflow/underflow

By bypassing Solidity's built in overflow/underflow checks using `unchecked`, we can save gas. This is especially beneficial for the index variable within for loops (saves 120 gas per iteration).

Instances: 15
- [Ethos-Core/contracts/TroveManager.sol#L608](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608)
- [Ethos-Core/contracts/TroveManager.sol#L690](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690)
- [Ethos-Core/contracts/TroveManager.sol#L808](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808)
- [Ethos-Core/contracts/TroveManager.sol#L882](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882)
- [Ethos-Core/contracts/StabilityPool.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351)
- [Ethos-Core/contracts/StabilityPool.sol#L397](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L397)
- [Ethos-Core/contracts/StabilityPool.sol#L640](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640)
- [Ethos-Core/contracts/StabilityPool.sol#L810](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810)
- [Ethos-Core/contracts/StabilityPool.sol#L831](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831)
- [Ethos-Core/contracts/StabilityPool.sol#L859](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L859)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L181)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L192](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L192)

<br>

## Use `private` rather than `public` for constants

Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table. If needed to be viewed externally, the values can be read from the verified contract source code.

Instances: 27
- [Ethos-Core/contracts/CollateralConfig.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L21)
- [Ethos-Core/contracts/CollateralConfig.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L25)
- [Ethos-Core/contracts/BorrowerOperations.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21)
- [Ethos-Core/contracts/TroveManager.sol#L48](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L48)
- [Ethos-Core/contracts/TroveManager.sol#L53](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L53)
- [Ethos-Core/contracts/TroveManager.sol#L54](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L54)
- [Ethos-Core/contracts/TroveManager.sol#L55](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L55)
- [Ethos-Core/contracts/TroveManager.sol#L61](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L61)
- [Ethos-Core/contracts/ActivePool.sol#L30](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30)
- [Ethos-Core/contracts/StabilityPool.sol#L150](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150)
- [Ethos-Core/contracts/StabilityPool.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L197)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L40](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L41](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L41)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L75](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L75)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L76)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L23](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L23)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L24](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L24)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L25](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L25)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L24](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L24)

<br>

## Change `public` functions to `external`

Functions marked as `public` that are not called internally should be set to `external` to save gas and improve code quality. External call cost is less expensive than of public functions.

Instances: 4
- [Ethos-Core/contracts/TroveManager.sol#L1045](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1045)
- [Ethos-Core/contracts/TroveManager.sol#L1440](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1440)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L295](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L295)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L67](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L67)

<br>

## Use a more recent version of Solidity

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than `revert()`/`require()` strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

Use a Solidity version of at least 0.8.12 to get string.concat() to be used instead of abi.encodePacked(<str>,<str>).

Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions.

Instances: 12
- [Ethos-Core/contracts/CollateralConfig.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3)
- [Ethos-Core/contracts/BorrowerOperations.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3)
- [Ethos-Core/contracts/TroveManager.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3)
- [Ethos-Core/contracts/ActivePool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3)
- [Ethos-Core/contracts/StabilityPool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3)
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3)
- [Ethos-Core/contracts/LUSDToken.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3)
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3)

<br>

## Naming a return variable and still calling the `return` keyword wastes gas

Instances: 18
- [Ethos-Core/contracts/TroveManager.sol#L329](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L329)
- [Ethos-Core/contracts/TroveManager.sol#L369](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L369)
- [Ethos-Core/contracts/TroveManager.sol#L899](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L899)
- [Ethos-Core/contracts/TroveManager.sol#L1316](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1316)
- [Ethos-Core/contracts/TroveManager.sol#L1321](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1321)
- [Ethos-Core/contracts/StabilityPool.sol#L511](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L511)
- [Ethos-Core/contracts/StabilityPool.sol#L626](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L626)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L29](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L29)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L37](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L37)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L96](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L96)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L165](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L165)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L220](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L220)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L240)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L104](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L104)

<br>

## Split `require` statements using `&&`

Replacing with two separate require statements saves 16 gas per instance.

Instances: 1
- [Ethos-Core/contracts/BorrowerOperations.sol#L653](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653)

<br>

## `x += y` costs more gas than `x = x + y` for state variables

Instances: 9
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L168](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L194](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L196](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L196)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L214)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L396](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L396)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L445](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L445)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L452](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L452)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L515](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L515)
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L521](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L521)

<br>

## Usage of `uint` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Consider using a larger size then downcasting where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

Instances: 6
- [Ethos-Core/contracts/StabilityPool.sol#L200](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L200)
- [Ethos-Core/contracts/StabilityPool.sol#L203](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L203)
- [Ethos-Core/contracts/StabilityPool.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L214)
- [Ethos-Core/contracts/StabilityPool.sol#L223](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L223)
- [Ethos-Core/contracts/LUSDToken.sol#L35](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L35)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35)

<br>

## `++i` costs less gas than `i++`

True for `--i`/`i--` as well, and is especially important in for loops. Saves 5 gas per iteration.

Instances: 17
- [Ethos-Core/contracts/CollateralConfig.sol#L56](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L57)
- [Ethos-Core/contracts/TroveManager.sol#L608](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608)
- [Ethos-Core/contracts/TroveManager.sol#L690](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690)
- [Ethos-Core/contracts/TroveManager.sol#L808](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808)
- [Ethos-Core/contracts/TroveManager.sol#L882](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882)
- [Ethos-Core/contracts/ActivePool.sol#L108](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108)
- [Ethos-Core/contracts/StabilityPool.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351)
- [Ethos-Core/contracts/StabilityPool.sol#L397](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L397)
- [Ethos-Core/contracts/StabilityPool.sol#L640](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640)
- [Ethos-Core/contracts/StabilityPool.sol#L810](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810)
- [Ethos-Core/contracts/StabilityPool.sol#L831](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831)
- [Ethos-Core/contracts/StabilityPool.sol#L859](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L859)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228)
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240)
- [Ethos-Core/contracts/LUSDToken.sol#L286](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L286)
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L273](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L273)

<br>
