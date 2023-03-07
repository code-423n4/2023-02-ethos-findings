# Report

## Don't Initialize Variables with Default Value

Uninitialized variables are assigned with the types default value.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108 => for(uint256 i = 0; i < numCollaterals; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56 => for(uint256 i = 0; i < collaterals.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206 => for (uint i = 0; i < assets.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228 => for (uint i = 0; i < collaterals.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240 => for (uint i = 0; i < numCollaterals; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/MultiTroveGetter.sol#L72 => for (uint idx = 0; idx < startIdx; ++idx) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/MultiTroveGetter.sol#L78 => for (uint idx = 0; idx < count; ++idx) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/MultiTroveGetter.sol#L101 => for (uint idx = 0; idx < startIdx; ++idx) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/MultiTroveGetter.sol#L107 => for (uint idx = 0; idx < count; ++idx) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L115 => for (uint256 i = 0; i < numCollaterals; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351 => for (uint i = 0; i < numCollaterals; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L397 => for (uint i = 0; i < numCollaterals; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640 => for (uint i = 0; i < assets.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810 => for (uint i = 0; i < collaterals.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831 => for (uint i = 0; i < collaterals.length; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L859 => for (uint i = 0; i < numCollaterals; i++) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L117 => for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L165 => for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L128 => for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L264 => for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L372 => for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L77 => for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {

## Unsafe ERC20 Operation(s)

ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103 => OathToken.transferFrom(msg.sender, address(this), amount);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127 => OathToken.transfer(account, OathAmount);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L135 => lusdToken.transfer(msg.sender, LUSDGain);
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L171 => lusdToken.transfer(msg.sender, LUSDGain);

## Unspecific Compiler Version Pragma

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IAToken.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IAaveIncentivesController.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IAaveProtocolDataProvider.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IERC20Minimal.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IERC4626Events.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IERC4626Functions.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IInitializableAToken.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/ILendingPool.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/ILendingPoolAddressesProvider.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IScaledBalanceToken.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IStrategy.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IVault.sol#L3 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IVeloPair.sol#L2 => pragma solidity ^0.8.0;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/interfaces/IVeloRouter.sol#L2 => pragma solidity ^0.8.0;

## Do not use Deprecated Library Functions

The usage of deprecated library functions should be discouraged.

This issue is mostly related to OpenZeppelin libraries.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74 => IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);

## Unused Local Variable

Unused local variables can indicate a lack of code maintenance, as well as potential security vulnerabilities. Unused variables can be difficult to detect and may contain malicious code, as they can be used to bypass security checks. It is important to audit all code for unused variables and remove them to prevent any potential security risks.

https://github.com/Pfed-prog/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L339

https://github.com/Pfed-prog/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L384
