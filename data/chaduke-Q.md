QA1.  The ``updateDistributionPeriod()`` function has no range check for the input ``_newDistributionPeriod``. As a result, it might cause an overflow for L112 of function ``fund()``: 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101-L117](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101-L117)

In addition, as ``_newDistributionPeriod`` is an important configuration parameter, a timelock should be introduced  help a smooth transition. 

Mitigation: 1) introduce a range check for ``_newDistributionPeriod``; 2) introduce a timelock delay; 
