Use custom solidity errors instead of require statements to save gas on string allocation.

Ex:  Instead of this
`require(someBool, "Error Message");`

use this
```
error ErrorMessage(); // declare error

// logically same as require.  Reverts state to before function was called.
// Saves gas on string allocation.
if(!someBool){
  revert ErrorMessage();
}
```

Saves ~50 gas each time the error triggers (https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746)
Also saves gas on deployment.

Some examples of `require` statements that can be gas optimized:
 - Ethos-Vault/contracts/ReaperVaultERC4626.sol line 270
 - Ethos-Vault/contracts/ReaperVaultV2.sol lines 150-156, 180, 181, 193, 197, 261, 267, 321, 322, 324, 364, 404, 436, 497, 568, 619, 629, 639
 - Ethos-Vault/contracts/ReaperBaseStrategyv4.sol lines 81, 96-98, 191

