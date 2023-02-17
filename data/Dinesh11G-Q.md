### Unspecific Compiler Version Pragma

#### Impact
Issue Information: 
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

Example
🤦 Bad:

pragma solidity ^0.8.0;
🚀 Good:

pragma solidity 0.8.4;

#### Findings:
```
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IAToken.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IAaveIncentivesController.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IAaveProtocolDataProvider.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IERC20Minimal.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IERC4626Events.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IERC4626Functions.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IInitializableAToken.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/ILendingPool.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/ILendingPoolAddressesProvider.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IScaledBalanceToken.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IStrategy.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IVault.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IVeloPair.sol::2 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/interfaces/IVeloRouter.sol::2 => pragma solidity ^0.8.0;
```
#### Tools used
Manual VS Code review