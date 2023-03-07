### Checking in the incorrect position.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L324
```
323        uint256 pool = balance();
324        require(pool + _amount <= tvlCap, "Vault is full");
325
326        uint256 freeFunds = _freeFunds();
327        uint256 balBefore = token.balanceOf(address(this));
328        token.safeTransferFrom(msg.sender, address(this), _amount);
329        uint256 balAfter = token.balanceOf(address(this));
330        _amount = balAfter - balBefore;
```

`require(pool + _amount <= tvlCap, "Vault is full");` should be checked after `_amount` reassignment.
If the token is an inflation token with fee logic when transferring, tvlCap should be checked based on the amount reached in the contract.