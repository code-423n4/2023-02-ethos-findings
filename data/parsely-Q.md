# [N-01] Consider in certain instances only importing only what is needed rather than using global imports.
### Context
All contracts
### Description
Importing only what is necessary is preferred above global imports

### Recommendation
```import {contractName} from "theContractFile.sol";```

# [N-02]  NatSpec comments should be added to public contracts
### Context
All public contracts eg. BorrowerOperations
### Description
According to the Solidity documentation it is recommended for all public contracts to have NatSpec comments added.
### Recommendation
Add NatSpec comments to the contracts that may be interacted with.  

# [N-03] Remove unused import of Ownable in TroveManager.sol
### Context
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L14
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L18
### Description
The reference and import for Ownable is commented out.
### Recommendation
Remove unused import and reference to Ownable.

# [N-04] Remove unused import of console
### Context
```
LUSDToken.sol:9:import "./Dependencies/console.sol";
StabilityPool.sol:18:import "./Dependencies/console.sol";
LQTY/LQTYStaking.sol:9:import "../Dependencies/console.sol";
BorrowerOperations.sol:15:import "./Dependencies/console.sol";
ActivePool.sol:15:import "./Dependencies/console.sol";
```
### Description
It does not seem that the logging function is used within the code.
### Recommendation
Remove unused imports.

# [N-05] Once the setAddresses function is called there is no way to correct any incorrect addresses
### Context
```
LQTY/LQTYStaking.sol:66:    function setAddresses
LQTY/CommunityIssuance.sol:61:    function setAddresses
TroveManager.sol:232:    function setAddresses(
BorrowerOperations.sol:110:    function setAddresses(
ActivePool.sol:71:    function setAddresses(
```
### Description
Once the setAddresses function has been called the ownership is changed to address(0) and there is no way to correct any address that may have a mistake.
### Recommendation
Implement a way to change any addresses that may be incorrect using either MultiSig or Timelock.

# [N-06] Without context of the deployment it may be possible for important addresses to not have been initialised.
### Context
```
LQTY/LQTYStaking.sol:66:    function setAddresses
LQTY/CommunityIssuance.sol:61:    function setAddresses
TroveManager.sol:232:    function setAddresses(
BorrowerOperations.sol:110:    function setAddresses(
ActivePool.sol:71:    function setAddresses(
```
### Description
Without context of the deployment it is worth mentioning that if setAddresses is not called during deployment, it is assumed in the rest of the code that the addresses have been initialised.
### Recommendation
Either implement a deployment pipeline ensuring that the values are initialised or have modifiers that check per function.

# [N-07] A malicious privileged user could manipulate the upgrade time in ReaperBaseStrategyv4.sol
### Context
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L167
### Description
The function to initiate the upgrade time is callable by any privileged user at any stage. This could allow them to delay the upgrade as long as they still have the necessary role.
```
function initiateUpgradeCooldown() external {
        _atLeastRole(STRATEGIST);
        upgradeProposalTime = block.timestamp;
    }
```
### Recommendation
Consider implementing MultiSig or a strategy where it cannot be the same ```msg.sender``` twice in a row.


