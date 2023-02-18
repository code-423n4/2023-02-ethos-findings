## Divide Before Multiply

The following line: https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L54-L55 performs a divide before multiply.

```
    uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
    uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%
```
As `DECIMAL_PRECISION` is set to `1e18` there is no lose of precision but this is a bad practice to exhibit. It is recommend that you should always multiply before you divide to lower the potential for precision lose.

For example:

```
➜ 1e18/100*5
Type: uint
├ Hex: 0xb1a2bc2ec50000
└ Decimal: 50000000000000000
➜ 1e18*5/100
Type: uint
├ Hex: 0xb1a2bc2ec50000
└ Decimal: 50000000000000000
```