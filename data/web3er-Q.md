## Title
Double Precision Losses in Function ```withdraw```

## Lineinfo
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L202-L210
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L359-L365

## Impact
In contract ```ReaperVaultERC4626```, there is a function named ```withdraw``` which user can withdraw asset. In function ```withdraw```, function ```previewWithdraw``` is called to calculate the shares that the user needs to burn to withdraw the asset. This place is rounded up, which means that the user needs to burn more shares to withdraw the asset. This is the first precision loss that can harm the interests of users.
```
function withdraw(
    uint256 assets,
    address receiver,
    address owner
) external override returns (uint256 shares) {
    shares = previewWithdraw(assets); // previewWithdraw() rounds up so exactly "assets" are withdrawn and not 1 wei less
    if (msg.sender != owner) _spendAllowance(owner, msg.sender, shares);
    _withdraw(shares, receiver, owner);
}
```
Then function ```_withdraw``` with rounded up parameter ```shares``` in contract ReaperVaultV2 is called. In the function, assets that user want to withdraw will be recalculated with rounding up burned shares. The finally calculated assets that the user can get back will be rounded down. This is the second precision loss that can harm the interests of users.
```
function _withdraw(
    uint256 _shares,
    address _receiver,
    address _owner
) internal nonReentrant returns (uint256 value) {
    require(_shares != 0, "Invalid amount");
    value = (_freeFunds() * _shares) / totalSupply();
    _burn(_owner, _shares);
    ······
}
```
Although the user's loss will be controlled within a certain value ```withdrawMaxLoss``` in the end, in fact, there can be better operations to reduce the user's loss as much as possible.

## Tools Used

## Recommended Mitigation Steps
In function ```_withdraw```, before calling the function ```_burn```, use ```assets``` passed in by the user as the initial value of ```value```.