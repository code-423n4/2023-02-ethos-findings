(1) Hardcoded addresses cannot be changed, risk if they get compromised. https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L24-L29 If any of the contracts get compromised or malfunction in some way, there's no real way to change any addresses, so this could result in lost funds.
Solution for this could be to simply create functions for initializing and changing each hardcoded address just in case.


(2)
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L288
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L334
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L365 

 ## Impact
Detailed description of the impact of this finding.

In the case that _freeFunds() returns a 0, which is possible if the balance() is not big enough, or the locked profit is big enough, the user will get 0 shares minted, and while withdrawing, will burn all his shares and get 0 tokens back.

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

The balance of the pool is 100 tokens.
The locked profit is 100.
The _freeFunds() function returns a 0.
Alice tries to withdraw her shares. All the shares are burned and 0 tokens are sent to her. 

## Tools Used
manual review
## Recommended Mitigation Steps
Handle the case where _freeFunds() returns a 0. Instead of burning the users shares and then sending 0 tokens, revert, also handle the depositing case where sending tokens to the contract gives an user 0 shares.


(3)  _freeFunds() can revert if the _calculateLockedProfit() is a larger number than the current balance()
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L288

## Impact
Detailed description of the impact of this finding.

The function _freeFunds() will revert if enough time has passed and the _calculateLockedProfit() amount is larger than the current balance.
Then, the user can't withdraw or deposit, because the tx reverts each time.

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

The _calculateLockedProfit() function can return a value that is bigger than the current vault's balance if enough time has passed and vault's balance may not be that high.

Let's say the current vault's balance is 100 tokens.
If the locked profit is 101, the function reverts and the user can't deposit or withdraw.

It's hard to say how often something like this can happen, but there could be a scenario where there isn't enough balance and enough time has passed for the profit to be bigger than the current balance, in that case it will revert each time.

## Tools Used
manual review
## Recommended Mitigation Steps
Handle the case where the lockedProfit is bigger than the current balance. If it's bigger, make sure it doesn't revert and handle it.






