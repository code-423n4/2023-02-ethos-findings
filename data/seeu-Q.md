| Index |
| --- |
| [Low issues](#low-issues) |
| [Non-Critical issues](#non-critical-issues) |

# Low issues

| ID                                                       | Issue                                      | Contexts | Instances |
| -------------------------------------------------------- | ------------------------------------------ | -------- | --------- |
| [L-01](#l-01-outdated-openzeppelin-contracts-dependency) | Outdated OpenZeppelin contracts dependency | 2        | 3         |
| [L-02](#l-02-avoid-using-abiencodepacked-with-dynamic-types-when-passing-the-result-to-a-hash-function) | Avoid using abi.encodePacked() with dynamic types when passing the result to a hash function | 1 | 1 |
| [L-03](#l-03-outdated-compiler-version) | Outdated Compiler Version | 12 | 12 |
| [L-04](#l-04-compiler-version-pragma-is-non-specific) | Compiler version Pragma is non-specific | 4 | 4 |
| [L-05](#l-05-ecrecover-does-not-check-for-address0) | ecrecover() does not check for address(0) | 1 | 1 |
| [L-06](#l-06-upgradeable-contract-is-lacking-the-storage-variable-__gap50) | Upgradeable contract is lacking the storage variable `__gap[50]` | 1 | 1 |
| [L-07](#l-07-unsafe-erc20-operations) | Unsafe ERC20 Operations | 1 | 1 |
| [L-08](#l-08-deprecated-openzeppelin-library-functions) | Deprecated OpenZeppelin library functions | 1 | 1 |
| [L-09](#l-09-use-require-instead-of-assert) | Use require instead of assert | 4 | 20 |
| [L-10](#l-10-decimals-is-not-part-of-erc20-standard-and-it-may-fail) | decimals() is not part of ERC20 standard and it may fail | 2 | 2 |
| [L-11](#l-11-timestamp-dependency-use-of-blocktimestamp-or-now-instances) | Timestamp dependency: use of block.timestamp (or now) Instances | 5 | 21 |
| [L-12](#l-12-use-_safemint-instead-of-_mint) | Use `_safeMint` instead of `_mint` | 2 | 4 |

| Total issues | Total contexts | Total instances |
| ------------ | -------------- | --------------- |
| 12           | 36             | 71              |


## [L-01] Outdated OpenZeppelin contracts dependency

### Description

In `package.json` it was found an outdated version for @openzeppelin/contracts and @openzeppelin/contracts-upgradeable. Older versions have more bugs and security issues.

It is reccomended to update [@openzeppelin/contracts](https://www.npmjs.com/package/@openzeppelin/contracts) and [@openzeppelin/contracts-upgradeable](https://www.npmjs.com/package/@openzeppelin/contracts-upgradeable) to the most recent version, `4.8.1`.

### Findings

- [Ethos-Vault/package.json#L41-L42](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/package.json#L41-L42)
  ```json
  "@openzeppelin/contracts": "^4.7.3",
  "@openzeppelin/contracts-upgradeable": "^4.7.3",
  ```
- [Ethos-Core/package.json#L34](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/package.json#L34)
  ```json
  "@openzeppelin/contracts": "^3.3.0",
  ```

### References

- [@openzeppelin/contracts](https://www.npmjs.com/package/@openzeppelin/contracts)
- [@openzeppelin/contracts-upgradeable](https://www.npmjs.com/package/@openzeppelin/contracts-upgradeable)

## [L-02] Avoid using abi.encodePacked() with dynamic types when passing the result to a hash function

### Description

Instead of using `abi.encodePacked()` use `abi.encode()`. It will pad items to 32 bytes, which will prevent [hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode).

It is possible to cast to `bytes()` or `bytes32()` in place of `abi.encodePacked()` when there is just one parameter, see "[how to compare strings in solidity?](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739)". `bytes.concat()` should be used if all parameters are strings or bytes.

### Findings

- [Ethos-Core/contracts/LUSDToken.sol#L283-L286](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L283-L286)
  ```Solidity
  bytes32 digest = keccak256(abi.encodePacked('\x19\x01', 
	domainSeparator(), keccak256(abi.encode(
	_PERMIT_TYPEHASH, owner, spender, amount, 
	_nonces[owner]++, deadline))));
  ```

### References

- [[L-1] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()](https://gist.github.com/GalloDaSballo/39b929e8bd48704b9d35b448aaa29480#l-1--abiencodepacked-should-not-be-used-with-dynamic-types-when-passing-the-result-to-a-hash-function-such-as-keccak256)



## [L-03] Outdated Compiler Version

### Description

Using an older compiler version might be risky, especially if the version in question has faults and problems that have been made public.

### Findings

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol) => `0.6.11`
- [Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol) => `0.6.11`
- [Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol) => `0.6.11`
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol) => `0.6.11`
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol) => `0.6.11`
- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol) => `0.6.11`
- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol) => `0.6.11`
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol) => `0.6.11`
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol) => `^0.8.0`
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol) => `^0.8.0`
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol) => `^0.8.0`
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol) => `^0.8.0`

### References

- [SWC-102](https://swcregistry.io/docs/SWC-102)
- [Etherscan Solidity Bug Info](https://etherscan.io/solcbuginfo)



## [L-04] Compiler version Pragma is non-specific

### Description

For non-library contracts, floating pragmas may be a security risk for application implementations, since a known vulnerable compiler version may accidentally be selected or security tools might fallback to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

### Findings

- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
  ```Solidity
  pragma solidity ^0.8.0;
  ```
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)
  ```Solidity
  pragma solidity ^0.8.0;
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  pragma solidity ^0.8.0;
  ```
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)
  ```Solidity
  pragma solidity ^0.8.0;
  ```

### References

- [L003 - Unspecific Compiler Version Pragma](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l003---unspecific-compiler-version-pragma)
- [Version Pragma | Solidity documents](https://docs.soliditylang.org/en/latest/layout-of-source-files.html#version-pragma)
- [4.6 Unspecific compiler version pragma | Consensys Audit of 1inch Liquidity Protocol](https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#unspecific-compiler-version-pragma)



## [L-05] ecrecover() does not check for address(0)

### Description

In the contract it was found the use of `ecrecover()` without implementing proper checks for `address(0)`. When a signature is incorrect, ecrecover may occasionally provide a random address rather than 0.

It is reccomended to add another check for address(0), here's an example:
```Solidity
require(signer != address(0), "ECDSA: invalid signature");
```

It is also reccomended to implement the OpenZeppelin soludion [ECDSA.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol)

### Findings

- [Ethos-Core/contracts/LUSDToken.sol#L287](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287)
  ```Solidity
  address recoveredAddress = ecrecover(digest, v, r, s);
  ```

### References

- [What is ecrecover in Solidity?](https://soliditydeveloper.com/ecrecover)



## [L-06] Upgradeable contract is lacking the storage variable `__gap[50]`

### Description

Even if it's possible that certain contracts aren't already sub-classed, including the storage variable `__gap[50]` now will prevent forgetting to do so in the future.

### Findings

- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol#L14-L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L14-L19) => missing `__gap[50]`
  ```Solidity
  abstract contract ReaperBaseStrategyv4 is
      ReaperAccessControl,
      IStrategy,
      UUPSUpgradeable,
      AccessControlEnumerableUpgradeable
  {
  ```

### Resources

- [Storage Gaps | OpenZeppelin docs](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps)
- [[L-02] UPGRADEABLE CONTRACT IS MISSING A __GAP[50] STORAGE VARIABLE TO ALLOW FOR NEW STORAGE VARIABLES IN LATER VERSIONS](https://code4rena.com/reports/2022-06-badger#l-02-upgradeable-contract-is-missing-a-__gap50-storage-variable-to-allow-for-new-storage-variables-in-later-versions)



## [L-07] Unsafe ERC20 Operations

### Description

ERC20 operations might not be secure due to multiple implementations and vulnerabilities in the standard. It is advised to use OpenZeppelin's SafeERC20 or, at least, wrap each operation in a `require` statement.

### Findings

- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103)
  ```Solidity
  OathToken.transferFrom(msg.sender, address(this), amount);
  ```

### References

- [L001 - Unsafe ERC20 Operation(s)](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l001---unsafe-erc20-operations) 
- [ERC20 OpenZeppelin documentation, contracts/IERC20.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol#L43) 



## [L-08] Deprecated OpenZeppelin library functions

### Description

The contracts use deprecated OpenZeppelin Library functions. It is recommend that you avoid using these features

### Findings

- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol#L74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74)
  ```Solidity
  IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);
  ```

### References

- [openzeppelin-contracts/issues/1064](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1064)
- [[L-1] Do not use deprecated library functions](https://gist.github.com/Picodes/ab2df52379e4b4993709be1b91aab651#l-1-do-not-use-deprecated-library-functions)



## [L-09] Use require instead of assert

### Description

It is reccomended to use `require` instead of `assert` since the latest, when false, uses up all the remaining gas and reverts all the changes made.

### Findings

- [Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
  ```Solidity
  ::128 =>         assert(MIN_NET_DEBT > 0);
  ::197 =>         assert(vars.compositeDebt > 0);
  ::301 =>         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
  ::331 =>         assert(_collWithdrawal <= vars.coll);
  ```
- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
  ```Solidity
  ::312 =>         assert(sender != address(0));
  ::313 =>         assert(recipient != address(0));
  ::321 =>         assert(account != address(0));
  ::329 =>         assert(account != address(0));
  ::337 =>         assert(owner != address(0));
  ::338 =>         assert(spender != address(0));
  ```
- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
  ```Solidity
  ::526 =>         assert(_debtToOffset <= _totalLUSDDeposits);
  ::551 =>         assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
  ::591 =>         assert(newP > 0);
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::417 =>         assert(_LUSDInStabPool != 0);
  ::1224 =>        assert(totalStakesSnapshot[_collateral] > 0);
  ::1279 =>        assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
  ::1342 =>        assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
  ::1348 =>        assert(index <= idxLast);
  ::1414 =>        assert(newBaseRate > 0); // Base rate is always non-zero after redemption
  ::1489 =>        assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
  ```

### References

- [Require vs Assert in Solidity](https://dev.to/tawseef/require-vs-assert-in-solidity-5e9d)



## [L-10] decimals() is not part of ERC20 standard and it may fail

Since `decimals()` is not a part of the official ERC20 standard, it could not work for some tokens.

### Description

### Findings

- [Ethos-Core/contracts/CollateralConfig.sol#L63](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L63)
  ```Solidity
  uint256 decimals = IERC20(collateral).decimals();
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L651](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L651)
  ```Solidity
  return token.decimals();
  ```

### References

- [[L-02] DECIMALS() NOT PART OF ERC20 STANDARD](https://code4rena.com/reports/2022-07-axelar#l-02-decimals-not-part-of-erc20-standard)



## [L-11] Timestamp dependency: use of block.timestamp (or now) Instances

### Description

The timestamp of a block is provided by the miner who mined the block. As a result, the timestamp is not guaranteed to be accurate or to be the same across different nodes in the network. In particular, an attacker can potentially mine a block with a timestamp that is favorable to them, known as "selective packing".

For example, an attacker could mine a block with a timestamp that is slightly in the future, allowing them to bypass a time-based restriction in a smart contract that relies on `block.timestamp`. This could potentially allow the attacker to execute a malicious action that would otherwise be blocked by the restriction.

### Findings

- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
  ```Solidity
  ::87 =>             uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;
  ::93 =>         lastIssuanceTimestamp = block.timestamp;
  ::113 =>         lastDistributionTime = block.timestamp.add(distributionPeriod);
  ::114 =>         lastIssuanceTimestamp = block.timestamp;
  ```
- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
  ```Solidity
  ::128 =>         deploymentStartTime = block.timestamp;
  ::282 =>         require(deadline >= now, 'LUSD: expired deadline');
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::1501 =>         uint timePassed = block.timestamp.sub(lastFeeOperationTime);
  ::1504 =>             lastFeeOperationTime = block.timestamp;
  ::1505 =>             emit LastFeeOpTimeUpdated(block.timestamp);
  ::1517 =>         return (block.timestamp.sub(lastFeeOperationTime)).div(SECONDS_IN_ONE_MINUTE);
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  ::121 =>         constructionTime = block.timestamp;
  ::122 =>         lastReport = block.timestamp;
  ::159 =>             activation: block.timestamp,
  ::165 =>             lastReport: block.timestamp
  ::419 =>         uint256 lockedFundsRatio = (block.timestamp - lastReport) * lockedProfitDegradation;
  ::540 =>         strategy.lastReport = block.timestamp;
  ::541 =>         lastReport = block.timestamp;
  ```
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)
  ```Solidity
  ::137 =>         lastHarvestTimestamp = block.timestamp;
  ::169 =>         upgradeProposalTime = block.timestamp;
  ::181 =>         upgradeProposalTime = block.timestamp + FUTURE_NEXT_PROPOSAL_TIME;
  ::192 =>             upgradeProposalTime + UPGRADE_TIMELOCK < block.timestamp,
  ```

### References
- [Timestamp dependence | Solidity Best Practices for Smart Contract Security](https://consensys.net/blog/developers/solidity-best-practices-for-smart-contract-security/)
- [What Is Timestamp Dependence?](https://halborn.com/what-is-timestamp-dependence/)



## [L-12] Use `_safeMint` instead of `_mint`

### Description

In favor of `_safeMint()`, which guarantees that the receiver is either an EOA or implements IERC721Receiver, `_mint()` is deprecated.

### Findings

- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
  ```Solidity
  ::188 =>         _mint(_account, _amount);
  ::320 =>     function _mint(address account, uint256 amount) internal {
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  ::336 =>         _mint(_receiver, shares);
  ::467 =>             _mint(treasury, shares);
  ```

### References
- [OpenZeppelin warning ERC721.sol#L271](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271)
- [solmate _safeMint](https://github.com/transmissions11/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L180)
- [OpenZeppelin _safeMint](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250)



# Non-Critical issues

| ID    | Issue | Contexts | Instances |
| ----- | ----- | -------- | --------- |
| [NC-01](#nc-01-requirerevert-statements-should-have-descriptive-reason-strings) | require()/revert() statements should have descriptive reason strings | 2 | 11 |
| [NC-02](#nc-02-unnamed-return-parameters) | Unnamed return parameters | 11 | 85 |
| [NC-03](#nc-03-for-modern-and-more-readable-code-update-import-usages) | For modern and more readable code; update import usages | 12 | 109 |
| [NC-04](#nc-04-code-base-comments-with-todos) | Code base comments with TODOs | 1 | 2 |

| Total issues | Total contexts | Total instances |
| ------------ | -------------- | --------------- |
| 4            | 26             | 207             |


## [NC-01] require()/revert() statements should have descriptive reason strings

### Description

To increase overall code clarity and aid in debugging whenever a need is not met, think about adding precise, informative error messages to all `require` and `revert` statements.

### Findings

- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::250 =>         require(msg.sender == owner);
  ::551 =>         require(totals.totalDebtInSequence > 0);
  ::716 =>         require(_troveArray.length != 0);
  ::754 =>         require(totals.totalDebtInSequence > 0);
  ::1450 =>         require(redemptionFee < _collateralDrawn);
  ::1523 =>         require(msg.sender == borrowerOperationsAddress);
  ::1527 =>         require(msg.sender == address(redemptionHelper));
  ::1531 =>         require(msg.sender == borrowerOperationsAddress || msg.sender == address(redemptionHelper));
  ::1535 =>         require(Troves[_borrower][_collateral].status == Status.active);
  ```
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
  ```Solidity
  ::167 =>             require(step[0] != address(0));
  ::168 =>             require(step[1] != address(0));
  ```

### References

- [Error handling: Assert, Require, Revert and Exceptions](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#error-handling-assert-require-revert-and-exceptions)
- [Missing error messages in require statements | Opyn Bull Strategy Contracts Audit](https://blog.openzeppelin.com/opyn-bull-strategy-contracts-audit/#missing-error-messages-in-require-statements)



## [NC-02] Unnamed return parameters

### Description

To increase explicitness and readability, take into account introducing and utilizing named return parameters.

### Findings

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
  ```Solidity
  ::159 =>     function getCollateral(address _collateral) external view override returns (uint) {
  ::164 =>     function getLUSDDebt(address _collateral) external view override returns (uint) {
  ```
- [Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
  ```Solidity
  ::419 =>     function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint) {
  ::432 =>     function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {
  ::748 =>     function getCompositeDebt(uint _debt) external pure override returns (uint) {
  ```
- [Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)
  ```Solidity
  ::102 =>     function getAllowedCollaterals() external override view returns (address[] memory) {
  ::106 =>     function isCollateralAllowed(address _collateral) external override view returns (bool) {
  ```
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
  ```Solidity
  ::199 =>     function getPendingCollateralGain(address _user) external view override returns (address[] memory, uint[] memory) {
  ::213 =>     function getPendingLUSDGain(address _user) external view override returns (uint) {
  ::217 =>     function _getPendingLUSDGain(address _user) internal view returns (uint) {
  ```
- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
  ```Solidity
  ::206 =>     function getDeploymentStartTime() external view override returns (uint256) {
  ::212 =>     function totalSupply() external view override returns (uint256) {
  ::216 =>     function balanceOf(address account) external view override returns (uint256) {
  ::220 =>     function transfer(address recipient, uint256 amount) external override returns (bool) {
  ::226 =>     function allowance(address owner, address spender) external view override returns (uint256) {
  ::230 =>     function approve(address spender, uint256 amount) external override returns (bool) {
  ::235 =>     function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool) {
  ::242 =>     function increaseAllowance(address spender, uint256 addedValue) external override returns (bool) {
  ::247 =>     function decreaseAllowance(address spender, uint256 subtractedValue) external override returns (bool) {
  ::254 =>     function domainSeparator() public view override returns (bytes32) {
  ::292 =>     function nonces(address owner) external view override returns (uint256) { // FOR EIP 2612
  ::304 =>     function _buildDomainSeparator(bytes32 typeHash, bytes32 name, bytes32 version) private view returns (bytes32) {
  ::399 =>     function name() external view override returns (string memory) {
  ::403 =>     function symbol() external view override returns (string memory) {
  ::407 =>     function decimals() external view override returns (uint8) {
  ::411 =>     function version() external view override returns (string memory) {
  ::415 =>     function permitTypeHash() external view override returns (bytes32) {
  ```
- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
  ```Solidity
  ::307 =>     function getCollateral(address _collateral) external view override returns (uint) {
  ::311 =>     function getTotalLUSDDeposits() external view override returns (uint) {
  ::410 =>     function depositSnapshots_S(address _depositor, address _collateral) external override view returns (uint) {
  ::439 =>     function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {
  ::657 =>     function _getSingularCollateralGain(uint _initialDeposit, address _collateral, Snapshots storage _snapshots) internal view returns (uint) {
  ::683 =>     function getDepositorLQTYGain(address _depositor) public view override returns (uint) {
  ::693 =>     function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {
  ::718 =>     function getCompoundedLUSDDeposit(address _depositor) public view override returns (uint) {
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::299 =>     function getTroveOwnersCount(address _collateral) external view override returns (uint) {
  ::303 =>     function getTroveFromTroveOwnersArray(address _collateral, uint _index) external view override returns (address) {
  ::1045 =>     function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {
  ::1066 =>     function _getCurrentTroveAmounts(address _borrower, address _collateral) internal view returns (uint, uint) {
  ::1123 =>     function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (uint) {
  ::1138 =>     function getPendingLUSDDebtReward(address _borrower, address _collateral) public view override returns (uint) {
  ::1152 =>     function hasPendingRewards(address _borrower, address _collateral) public view override returns (bool) {
  ::1195 =>     function updateStakeAndTotalStakes(address _borrower, address _collateral) external override returns (uint) {
  ::1201 =>     function _updateStakeAndTotalStakes(address _borrower, address _collateral) internal returns (uint) {
  ::1213 =>     function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {
  ::1361 =>     function getTCR(address _collateral, uint _price) external view override returns (uint) {
  ::1366 =>     function checkRecoveryMode(address _collateral, uint _price) external view override returns (bool) {
  ::1425 =>     function getRedemptionRate() public view override returns (uint) {
  ::1429 =>     function getRedemptionRateWithDecay() public view override returns (uint) {
  ::1433 =>     function _calcRedemptionRate(uint _baseRate) internal pure returns (uint) {
  ::1440 =>     function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {
  ::1444 =>     function getRedemptionFeeWithDecay(uint _collateralDrawn) external view override returns (uint) {
  ::1448 =>     function _calcRedemptionFee(uint _redemptionRate, uint _collateralDrawn) internal pure returns (uint) {
  ::1456 =>     function getBorrowingRate() public view override returns (uint) {
  ::1460 =>     function getBorrowingRateWithDecay() public view override returns (uint) {
  ::1464 =>     function _calcBorrowingRate(uint _baseRate) internal pure returns (uint) {
  ::1471 =>     function getBorrowingFee(uint _LUSDDebt) external view override returns (uint) {
  ::1475 =>     function getBorrowingFeeWithDecay(uint _LUSDDebt) external view override returns (uint) {
  ::1479 =>     function _calcBorrowingFee(uint _borrowingRate, uint _LUSDDebt) internal pure returns (uint) {
  ::1509 =>     function _calcDecayedBaseRate() internal view returns (uint) {
  ::1516 =>     function _minutesPassedSinceLastFeeOp() internal view returns (uint) {
  ::1544 =>     function getTroveStatus(address _borrower, address _collateral) external view override returns (uint) {
  ::1548 =>     function getTroveStake(address _borrower, address _collateral) external view override returns (uint) {
  ::1552 =>     function getTroveDebt(address _borrower, address _collateral) external view override returns (uint) {
  ::1556 =>     function getTroveColl(address _borrower, address _collateral) external view override returns (uint) {
  ::1567 =>     function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint) {
  ::1574 =>     function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint) {
  ::1581 =>     function increaseTroveDebt(address _borrower, address _collateral, uint _debtIncrease) external override returns (uint) {
  ::1588 =>     function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint) {
  ```
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
  ```Solidity
  ::222 =>     function balanceOf() public view override returns (uint256) {
  ::226 =>     function balanceOfWant() public view returns (uint256) {
  ::230 =>     function balanceOfPool() public view returns (uint256) {
  ```
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)
  ```Solidity
  ::269 =>     function roundUpDiv(uint256 x, uint256 y) internal pure returns (uint256) {
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  ::225 =>     function availableCapital() public view returns (int256) {
  ::278 =>     function balance() public view returns (uint256) {
  ::287 =>     function _freeFunds() internal view returns (uint256) {
  ::295 =>     function getPricePerFullShare() public view returns (uint256) {
  ::418 =>     function _calculateLockedProfit() internal view returns (uint256) {
  ::462 =>     function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {
  ::493 =>     function report(int256 _roi, uint256 _repayment) external returns (uint256) {
  ::650 =>     function decimals() public view override returns (uint8) {
  ::659 =>     function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {
  ::673 =>     function _hasRole(bytes32 _role, address _account) internal view override returns (bool) {
  ```
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)
  ```Solidity
  ::203 =>     function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {
  ::217 =>     function _hasRole(bytes32 _role, address _account) internal view override returns (bool) {
  ```

### References

- [Unnamed return parameters | Opyn Bull Strategy Contracts Audit](https://blog.openzeppelin.com/opyn-bull-strategy-contracts-audit/#unnamed-return-parameters)


## [NC-03] For modern and more readable code; update import usages

### Description

A less obvious way that solidity code is clearer is the struct Point. Prior to now, we imported it via global import, but we didn't use it. The Point struct contaminated the source code with an extra object that was not needed and that we were not utilizing.

To be sure to only import what you need, use specific imports using curly brackets.

### Findings

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
  ```Solidity
  ::5 => import './Interfaces/IActivePool.sol';
  ::6 => import "./Interfaces/ICollateralConfig.sol";
  ::7 => import './Interfaces/IDefaultPool.sol';
  ::8 => import "./Interfaces/ICollSurplusPool.sol";
  ::9 => import "./Interfaces/ILQTYStaking.sol";
  ::10 => import "./Interfaces/IStabilityPool.sol";
  ::11 => import "./Interfaces/ITroveManager.sol";
  ::12 => import "./Dependencies/SafeMath.sol";
  ::13 => import "./Dependencies/Ownable.sol";
  ::14 => import "./Dependencies/CheckContract.sol";
  ::15 => import "./Dependencies/console.sol";
  ::16 => import "./Dependencies/SafeERC20.sol";
  ::17 => import "./Dependencies/IERC4626.sol";
  ```
- [Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
  ```Solidity
  ::5 => import "./Interfaces/IBorrowerOperations.sol";
  ::6 => import "./Interfaces/ICollateralConfig.sol";
  ::7 => import "./Interfaces/ITroveManager.sol";
  ::8 => import "./Interfaces/ILUSDToken.sol";
  ::9 => import "./Interfaces/ICollSurplusPool.sol";
  ::10 => import "./Interfaces/ISortedTroves.sol";
  ::11 => import "./Interfaces/ILQTYStaking.sol";
  ::12 => import "./Dependencies/LiquityBase.sol";
  ::13 => import "./Dependencies/Ownable.sol";
  ::14 => import "./Dependencies/CheckContract.sol";
  ::15 => import "./Dependencies/console.sol";
  ::16 => import "./Dependencies/SafeERC20.sol";
  ```
- [Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)
  ```Solidity
  ::5 => import "./Dependencies/CheckContract.sol";
  ::6 => import "./Dependencies/Ownable.sol";
  ::7 => import "./Dependencies/SafeERC20.sol";
  ::8 => import "./Interfaces/ICollateralConfig.sol";
  ```
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
  ```Solidity
  ::5 => import "../Dependencies/IERC20.sol";
  ::6 => import "../Interfaces/ICommunityIssuance.sol";
  ::7 => import "../Dependencies/BaseMath.sol";
  ::8 => import "../Dependencies/LiquityMath.sol";
  ::9 => import "../Dependencies/Ownable.sol";
  ::10 => import "../Dependencies/CheckContract.sol";
  ::11 => import "../Dependencies/SafeMath.sol";
  ```
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
  ```Solidity
  ::5 => import "../Dependencies/BaseMath.sol";
  ::6 => import "../Dependencies/SafeMath.sol";
  ::7 => import "../Dependencies/Ownable.sol";
  ::8 => import "../Dependencies/CheckContract.sol";
  ::9 => import "../Dependencies/console.sol";
  ::10 => import "../Dependencies/IERC20.sol";
  ::11 => import "../Interfaces/ICollateralConfig.sol";
  ::12 => import "../Interfaces/ILQTYStaking.sol";
  ::13 => import "../Interfaces/ITroveManager.sol";
  ::14 => import "../Dependencies/LiquityMath.sol";
  ::15 => import "../Interfaces/ILUSDToken.sol";
  ::16 => import "../Dependencies/SafeERC20.sol";
  ```
- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
  ```Solidity
  ::5 => import "./Interfaces/ILUSDToken.sol";
  ::6 => import "./Interfaces/ITroveManager.sol";
  ::7 => import "./Dependencies/SafeMath.sol";
  ::8 => import "./Dependencies/CheckContract.sol";
  ::9 => import "./Dependencies/console.sol";
  ```
- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
  ```Solidity
  ::5 => import './Interfaces/IBorrowerOperations.sol';
  ::6 => import "./Interfaces/ICollateralConfig.sol";
  ::7 => import './Interfaces/IStabilityPool.sol';
  ::8 => import './Interfaces/IBorrowerOperations.sol';
  ::9 => import './Interfaces/ITroveManager.sol';
  ::10 => import './Interfaces/ILUSDToken.sol';
  ::11 => import './Interfaces/ISortedTroves.sol';
  ::12 => import "./Interfaces/ICommunityIssuance.sol";
  ::13 => import "./Dependencies/LiquityBase.sol";
  ::14 => import "./Dependencies/SafeMath.sol";
  ::15 => import "./Dependencies/LiquitySafeMath128.sol";
  ::16 => import "./Dependencies/Ownable.sol";
  ::17 => import "./Dependencies/CheckContract.sol";
  ::18 => import "./Dependencies/console.sol";
  ::19 => import "./Dependencies/SafeERC20.sol";
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::5 => import "./Interfaces/ICollateralConfig.sol";
  ::6 => import "./Interfaces/ITroveManager.sol";
  ::7 => import "./Interfaces/IStabilityPool.sol";
  ::8 => import "./Interfaces/ICollSurplusPool.sol";
  ::9 => import "./Interfaces/ILUSDToken.sol";
  ::10 => import "./Interfaces/ISortedTroves.sol";
  ::11 => import "./Interfaces/ILQTYStaking.sol";
  ::12 => import "./Interfaces/IRedemptionHelper.sol";
  ::13 => import "./Dependencies/LiquityBase.sol";
  ::15 => import "./Dependencies/CheckContract.sol";
  ::16 => import "./Dependencies/IERC20.sol";
  ```
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
  ```Solidity
  ::5 => import "./abstract/ReaperBaseStrategyv4.sol";
  ::6 => import "./interfaces/IAToken.sol";
  ::7 => import "./interfaces/IAaveProtocolDataProvider.sol";
  ::8 => import "./interfaces/ILendingPool.sol";
  ::9 => import "./interfaces/ILendingPoolAddressesProvider.sol";
  ::10 => import "./interfaces/IRewardsController.sol";
  ::11 => import "./libraries/ReaperMathUtils.sol";
  ::12 => import "./mixins/VeloSolidMixin.sol";
  ::13 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
  ::14 => import "@openzeppelin/contracts-upgradeable/utils/math/MathUpgradeable.sol";
  ```
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)
  ```Solidity
  ::5 => import "./ReaperVaultV2.sol";
  ::6 => import "./interfaces/IERC4626Functions.sol";
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  ::5 => import "./interfaces/IERC4626Events.sol";
  ::6 => import "./interfaces/IStrategy.sol";
  ::7 => import "./libraries/ReaperMathUtils.sol";
  ::8 => import "./mixins/ReaperAccessControl.sol";
  ::9 => import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";
  ::10 => import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
  ::11 => import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
  ::12 => import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
  ::13 => import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
  ::14 => import "@openzeppelin/contracts/utils/math/Math.sol";
  ```
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)
  ```Solidity
  ::5 => import "../interfaces/IStrategy.sol";
  ::6 => import "../interfaces/IVault.sol";
  ::7 => import "../libraries/ReaperMathUtils.sol";
  ::8 => import "../mixins/ReaperAccessControl.sol";
  ::9 => import "@openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";
  ::10 => import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
  ::11 => import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
  ::12 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
  ```

### References
- [[N-03] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES | PoolTogether contest](https://code4rena.com/reports/2022-12-pooltogether#n-03-for-modern-and-more-readable-code-update-import-usages)



## [NC-04] Code base comments with TODOs

### Description

Before deploying the system, the following instances of TODO comments in the codebase should be noted in the project's problems backlog and eliminated.

Having clearly stated TODO notes throughout development will facilitate monitoring and resolving them. Without that knowledge, these remarks could become outdated and crucial details for the system's security might be lost by the time it is put into production.

Consider keeping track of all TODO comments in the backlog of issues and connecting each inline TODO to the related item. Before deploying to a production environment, all TODOs must be completed.

### Findings

- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
  ```Solidity
  ::335 =>         /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
  ::380 =>         /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
  ```

### References

- [TODO comments in the code base | zkSync Layer 1 Audit](https://blog.openzeppelin.com/zksync-layer-1-audit/#todo-comments-in-the-code-base)