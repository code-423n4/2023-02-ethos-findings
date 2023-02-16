### [L-01] Missing Zero Address checker in constructor.
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136

Constructors run once and every thing is intialized and can't be changed. Using 0 as token will cause alot of problems

### Recommendation:
zero address checker should be implemented in the constructor to make sure 0 isn't  inputted as _token. 


