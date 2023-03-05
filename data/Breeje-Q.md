# QA Report

## Low Risk Issues
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | `pragma experimental ABIEncoderV2` Used is deprecated | 1 |
| [L-02] | Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions | 1 |
| [L-03] | Use of `ecrecover` can lead to signature mallebility vulnerability | 1 |

| Total Low Risk Issues | 3 |
|:--:|:--:|

## Non-Critical Issues
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | Function state mutability can be restricted to pure | 2 |
| [N-02] | Variable name should be in CamelCase | 1 |
| [N-03] | No Error Message provided for `require` | 3 |
| [N-04] | Spelling Errors in Natspec | 1 |
| [N-05] | Recommended to use 2 step while Updating Critical addresses | 2 |

| Total Non-Critical Issues | 5 |
|:--:|:--:|

### [L-01] `pragma experimental ABIEncoderV2` Used is deprecated

## Description

`pragma experimental ABIEncoderV2` Used is deprecated. Should use `pragma abicoder v2` instead which supports more types than v1 and performs more sanity checks on the inputs.

ABI coder v2 is activated by default in Solidity Version ^0.8.0. So it is already Enabled without explictly enabling it.

## Reference

* [Breaking Changes in Solidity ^0.8.0](https://github.com/ethereum/solidity/blob/69411436139acf5dbcfc5828446f18b9fcfee32c/docs/080-breaking-changes.rst#silent-changes-of-the-semantics)

## Recommendation Mitigation Step

Remove pragma experimental ABIEncoderV2 from the following code instance.

*Instance (1):*
```solidity
File: DistributionTypes.sol

2:    pragma experimental ABIEncoderV2;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/libraries/DistributionTypes.sol#L2)

### [L-02] Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions

See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

*Instance (1):*
```solidity
File: ReaperBaseStrategyv4.sol

14:   abstract contract ReaperBaseStrategyv4 is

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L14-L17)

### [L-03] Use of `ecrecover` can lead to signature mallebility vulnerability

It is also recommended to use OpenZeppelin’s ECDSA library instead of ecrecover: [ECDSA.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol)

Can check Latest `permit` method of Openzeppelin [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Permit.sol).

*Instance (1):*
```solidity
File: LUSDToken.sol

    address recoveredAddress = ecrecover(digest, v, r, s);
    require(recoveredAddress == owner, 'LUSD: invalid signature');

```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)

### [N-01] Function state mutability can be restricted to pure

Recommend to Change the state mutability of the following function to pure instead of view.

*Instances (2):*
```solidity
File: ReaperVaultV2.sol

659:    function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {

```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L659)

```solidity
File: ReaperBaseStrategyv4.sol

203:    function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {

```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L203)

### [N-02] Variable name should be in CamelCase

*Instance (1):*
```solidity
File: ReaperVaultV2.sol

473:    struct LocalVariables_report {

```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L473)

### [N-03] No Error Message provided for `require`

Provide an error message for `require`.

*Instances (3):*
```solidity
File: ReaperStrategyGranarySupplyOnly.sol

167:    require(step[0] != address(0));
168:    require(step[1] != address(0));

```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167-L168)

```solidity
File: VeloSolidMixin.sol

99:     require(
100:        _tokenIn != _tokenOut && _path.length >= 2 && _path[0] == _tokenIn && _path[_path.length - 1] == _tokenOut
101:    );

```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/mixins/VeloSolidMixin.sol#L99-L101)

### [N-04] Spelling Errors in Natspec

*Instance (1):*

Correct the Spelling of `Ammount` to `Amount` and `Stricly` to `Strictly`.

```solidity
File: ReaperBaseStrategyv4.sol

98:     require(_amount <= balanceOf(), "Ammount must be less than balance");

203:    * {KEEPER} - Stricly permissioned trustless access for off-chain programs or third party keepers.

```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L203)

### [N-05] Recommended to use 2 step while Updating Critical addresses

It is recommended to Use 2 Step ownership transfer for critical functions to avoid any foul circumstances. Here if governance address is set to a wrong address, `LUSDToken` can never be `unpause` from `pause` state.

*Instance (2):*
```solidity
File: LUSDToken.sol

146:    function updateGovernance(address _newGovernanceAddress) external {

153:    function updateGuardian(address _newGuardianAddress) external {

```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L146)