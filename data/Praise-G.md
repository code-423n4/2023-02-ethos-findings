### [G-01]- There is no need to initialize state variables to '0'. 
If a variable is not set/initialized, the default value is assumed (0, false, 0x0 … depending on the data type). You are simply wasting gas if you directly initialize it with its default value.

## Proof of Concept
```
    uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;
```
Affected Source code:
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35