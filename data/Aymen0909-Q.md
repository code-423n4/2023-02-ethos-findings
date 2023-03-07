# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | `delegateBySig` doesn't revert when `signer` is `address(0)` | Low | 1 |
| 2 | Front-runable `initialize` function | Low | 1 |
| 3 | Mistake in the `feeBPS` checks revert string | NC | 2 |
| 4 | Remove old code comment lines | NC | 1 |


## Findings


### 1- `permit` doesn't revert when `recoveredAddress` is `address(0)` :

### Risk : Low

The function `permit` from the LUSDToken contract is used to allow the users to delegate by signatures and it uses the `ecrecover` function to get the `recoveredAddress` address, this function can in some cases return `address(0)` but the `permit` function does not check the `recoveredAddress` address and thus it will not revert in case of `recoveredAddress == address(0)`.

### Proof of Concept

the issue occurs in the `permit` function below :

File: LUSDToken.sol [Line 262-294](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L262-L294)

```solidity
function permit
(
    address owner, 
    address spender, 
    uint amount, 
    uint deadline, 
    uint8 v, 
    bytes32 r, 
    bytes32 s
) 
    external 
    override 
{
    // EIP-2 still allows signature malleability for ecrecover(). Remove this possibility and make the signature
    // unique. Appendix F in the Ethereum Yellow paper (https://ethereum.github.io/yellowpaper/paper.pdf), defines
    // the valid range for s in (301): 0 < s < secp256k1n ÷ 2 + 1, and for v in (302): v ∈ {27, 28}. Most
    // signatures from current libraries generate a unique signature with an s-value in the lower half order.
    if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {
        revert('LUSD: Invalid s value');
    }
    require(deadline >= now, 'LUSD: expired deadline');
    bytes32 digest = keccak256(abi.encodePacked('\x19\x01', 
                        domainSeparator(), keccak256(abi.encode(
                        _PERMIT_TYPEHASH, owner, spender, amount, 
                        _nonces[owner]++, deadline))));
    address recoveredAddress = ecrecover(digest, v, r, s);
    require(recoveredAddress == owner, 'LUSD: invalid signature');
    _approve(owner, spender, amount);
}
```

As you can see the returned `recoveredAddress` address is used without checking its value and thus the call will not revert if its equla to `address(0)`.

### Mitigation
Add a non zero address check on the returned `recoveredAddress` address


### 2- Front-runable `initialize` function :

The `initialize` function in the `ReaperStrategyGranarySupplyOnly` contract have an issue that any attacker can initialize the contract before the legitimate deployer and even if the developers when deploying call immediately the `initialize` function, malicious agents can trace the protocol deployment transactions and insert their own transaction between them and by doing so they front run the developers call and gain the ownership of the contract.

The impact of this issue is : 

* In the best case developers notice the problem and have to redeploy the contract and thus costing more gas.

* In the worst case the protocol continue to work with the wrong owner which could lead to various problems for users.

#### Risk : Low 

#### Proof of Concept

Instances include:

File: ReaperStrategyGranarySupplyOnly.sol[Line 62-72](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L72)

```solidity
function initialize(
    address _vault,
    address[] memory _strategists,
    address[] memory _multisigRoles,
    IAToken _gWant
) public initializer 
```

#### Mitigation

It's recommended to use the constructor to initialize non-proxied contracts. But in this case for initializing proxy (upgradable) contracts deploy contracts using a factory contract that immediately calls initialize after deployment or make sure to call it immediately after deployment and verify that the transaction succeeded.


### 3- Mistake in the `feeBPS` checks revert string :

From the code logic we can see that the `feeBPS` value is supposed to be always less than 2000 BPS (20%) but the error revert string gives wrong information as it said that `feeBPS` value cannot be higher than 20 BPS.

#### Risk : Non Critical

#### Proof of Concept

Instances include:

File: ReaperVaultV2.sol [Line 155](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155)
```solidity
require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
```

File: ReaperVaultV2.sol [Line 181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L181)
```solidity
require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
```

#### Mitigation
Correct the revert string for the checks on the `feeBPS` value.

### 4- Remove old code comment lines :

#### Risk : Non Critical

There are some comments left from the old contracts which are does not correspand to the currect code and thus should be removed to avoid confusion.

Instances include:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L23-L36