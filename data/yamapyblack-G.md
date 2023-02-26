# 1. Can `actualWithdrawn` be calculated without using `balanceOf` or another storage access?

In order to compute `actualWithdrawn`, `balanceOf` is run in a loop.

[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L386](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L386)

`balanceOf` is also executed twice in the same loop.

[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L373](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L373)

If `actualWithdrawn` can be calculated in another way, `balanceOf` in a loop is omitted and the code around here can be simplified.

## Recommendation

`IStrategy`'s `withdraw` had better return loss and `actualWithdrawn`.

`IStrategy`

```jsx
function withdraw(uint256 _amount) external returns (uint256 loss, uint256 actualWithdrawn);
```

ReaperBaseStrategyv4.sol (implementation)

```jsx
function withdraw(uint256 _amount) external override returns (uint256 loss, uint256 amountFreed) {
```

The entire loop looks like the following.

```solidity
uint256 vaultBalance = token.balanceOf(address(this));
for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
    if (value <= vaultBalance) {
        break;
    }

    address stratAddr = withdrawalQueue[i];
    uint256 strategyBal = strategies[stratAddr].allocated;
    if (strategyBal == 0) {
        continue;
    }

    uint256 remaining = value - vaultBalance;
    (uint256 loss, uint256 actualWithdrawn) = IStrategy(stratAddr).withdraw(Math.min(remaining, strategyBal));
		vaultBalance -= actualWithdrawn;

    // Withdrawer incurs any losses from withdrawing as reported by strat
    if (loss != 0) {
        value -= loss;
        totalLoss += loss;
        _reportLoss(stratAddr, loss);
    }

    strategies[stratAddr].allocated -= actualWithdrawn;
    totalAllocated -= actualWithdrawn;
}
```

# 2. It is not necessary to get the token balances before and after.

[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L327-L330](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L327-L330)

```jsx
				uint256 balBefore = token.balanceOf(address(this));
        token.safeTransferFrom(msg.sender, address(this), _amount);
        uint256 balAfter = token.balanceOf(address(this));
        _amount = balAfter - balBefore;
```

As the reduced amount always matches _amount, there is no need to calculate balBefore and balAfter. The above four lines can be simplified to the following one line.

```jsx
        token.safeTransferFrom(msg.sender, address(this), _amount);
```

# 3. ERC20’s `name` and `symbol` aren’t necessary to cast string

[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L119](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L119)

```jsx
ERC20(string(_name), string(_symbol))
```

## Recommendation

```jsx
ERC20(_name, _symbol)
```
# 4. `debt` can be simplified by the ternary operator

When assigning to variables with if statement, the ternary operator would be useful.

[https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L112-L115](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L112-L115)

# Recommendation

```jsx
uint256 debt = availableCapital < 0 ? uint256(-availableCapital) : 0;
```