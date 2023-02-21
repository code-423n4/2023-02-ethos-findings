## Unauthorized access to Permit

Contract:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287

Issue:

1. The address recovery from signature is done through `ecrecover` function.
2. This function gives address(0) in case of invalid signature
3. There is no check to validate recoveredAddress!=address(0) which means you can approve any amount from address(0)

```
 address recoveredAddress = ecrecover(digest, v, r, s);
        require(recoveredAddress == owner, 'LUSD: invalid signature');
        _approve(owner, spender, amount);
```

Recommendation:
Add a check to ensure that recoveredAddress!=address(0)

```
 require(recoveredAddress!=address(0) && recoveredAddress == owner, 'LUSD: invalid signature');
```


## Duplicate strategy are allowed

Contract:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L258

Issue:

1. Duplicate strategy can be added using `setWithdrawalQueue` function
2. This means `withdrawalQueue` could contain multiple copies of same strategy
3. Since `withdrawalQueue` is used to decide the strategy used to pull fund from, a duplicate strategy could cause using same strategy multiple times which was not intended

Recommendation:
If a strategy already exist in `withdrawalQueue` then revert