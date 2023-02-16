# [L-01] Missing Zero Address checker in constructor.
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136

Constructors run once and every thing is intialized and can't be changed. Using 0 as _token will cause alot of problems

### Recommendation:
Zero address checker should be implemented in the constructor to make sure 0 isn't  inputted as _token. 

# [L-02] Missing zero address checker in initialize function.
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L72

This is very similar to L-01 because initialize() functions act as constructors in upgradable contracts.
Inputting 0 as vault will cause a lot of problems.  

### Recommendation:
Zero address checker should be implemented in this function to make sure 0 isn't inputted as vault.