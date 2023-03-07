## 1. NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

	pragma solidity ^0.8.0;

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3

## 2. USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’

Can import the required specific contracts, functions or variables by using the named imports explicitly. Plain imports will import the entire context of the imported contract which could lead into variable name conflicts etc ...

Currently the `ReaperAccessControl` is imported as follows:

	import "./mixins/ReaperAccessControl.sol";

But it can be imported explicitly by the name as follows:

	import {ReaperAccessControl} from "./mixins/ReaperAccessControl.sol";

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L5-L14
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L5-L14
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L5-L6
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L5-L12
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L5-L8

There are 15 more instances of this issue:

## 3. DO NOT USE MAGIC NUMBERS, USE `10**` TO DEPICT NUMBERS FOR BETTER READABILITY OF THE CODE.

    uint256 public constant PERCENT_DIVISOR = 10000;
	
Above line of code can be rewritten as follows for better code readability.

    uint256 public constant PERCENT_DIVISOR = 10**4;	

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L41

## 4. CONSTRUCTOR LACKS `address(0)` CHECKS

Zero-address check should be used in the constructors, to avoid the risk of setting protocol critical parameters as address(0) at contract deployment time.

        token = IERC20Metadata(_token);

above line of code should be preceded by the `address(0)` check as follows:

        require(_token != address(0), "_token can not be zero address");
        token = IERC20Metadata(_token);

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L120
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L124

## 5. USE BOTH OLD AND NEW VALUES WHEN EMITTING EVENTS FOR CRITICAL PARAMETER UPDATES

For the use of off-chain tools it is recommended to emit both old and new values when critical parameters are updated in the protocol.

    function updateStrategyFeeBPS(address _strategy, uint256 _feeBPS) external {
        _atLeastRole(ADMIN);
        require(strategies[_strategy].activation != 0, "Invalid strategy address");
        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
        strategies[_strategy].feeBPS = _feeBPS;
        emit StrategyFeeBPSUpdated(_strategy, _feeBPS);
    }
		
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L178-L184
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L198
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L216 
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L270
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L570
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L581
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L621

## 6. CONDUCT `address(0)` CHECKS FOR THE PROTOCOL CRITICAL ADDRESSES

        _mint(_receiver, shares);
		
In the above example `_mint()` is used to mint shares to the `_reciever` address. Hence `_reciever` address should be checked for address(0) as follows (before the `_mint` function is called):

	require(_reciever != address(0), "Receiver can not be zero address");

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L336
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L70

## 7. EMIT `events` FOR PROTOCOL CRITICAL PARAMETER CHANGES 

Events should be emitted appropriately for protocol critical parameter changes to be used by off-chain tools.

    function updateTreasury(address newTreasury) external {
        _atLeastRole(DEFAULT_ADMIN_ROLE);
        require(newTreasury != address(0), "Invalid address");
        treasury = newTreasury;
    }
	
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L627-L631

## 8. MALICIOUS ADMIN CAN DRAIN THE CONTRACT IF THE UNDERLYING TOKEN IS A PROXIED TOKEN WITH MULTIPLE ADDRESSES.

If the underlying token of the vault is a proxied token with multiple addresses, then the `Admin` can drain the vault assets by bypassing the following `require` statement and transfering the entire contract balance to admin himself.

        require(_token != address(token), "!token");
		
        uint256 amount = IERC20Metadata(_token).balanceOf(address(this));
        IERC20Metadata(_token).safeTransfer(msg.sender, amount); 
        emit InCaseTokensGetStuckCalled(_token, amount);		
		
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L637-L644

## 9. `initialize()` FUNCTIONS SHOULD BE USING ACCESS CONTROL

`Initialize()` function should be secured by access control to prevent front running the `initialize()` function by an attacker. 
In the worst case if the `initialize()` function is not secured by access control, it could prompt redeployment of the contract as well.
		
    function initialize( 
        address _vault,
        address[] memory _strategists,
        address[] memory _multisigRoles,
        IAToken _gWant
    ) public initializer {
        gWant = _gWant;
        want = _gWant.UNDERLYING_ASSET_ADDRESS();
        __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles); 
        rewardClaimingTokens = [address(_gWant)];
    }
	
Only let permissioned users to initialize the `ReaperStrategyGranarySupplyOnly` contract by calling the `initialize()` function.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L72

## 10. USE MORE RECENT VERSION OF SOLIDITY

Old version of solidity is used (^0.8.0 and 0.6.11), but newer version 0.8.17 can be used.

For security, it is recommended to use the latest solidity version.
security fix list for different solidity versions can be found in the following link.

https://github.com/ethereum/solidity/blob/develop/Changelog.md

Above issue is found in all contracts.

## 11. USING VULNERABLE DEPENDENCY OF OPENZEPPELIN

According to the package.json file, the project is using 4.7.0, to be exact ^4.7.3 which is not the latest updated version.

      "dependencies": {
        "@openzeppelin/contracts": "^4.7.3",

It is recommended to use the latest non vulnerable version 4.8.0

## 12. NATSPEC IS MISSING

NatSpec is missing for the following contracts, functions, constructor and modifier

`ReaperBaseStrategyv4.sol`, `ReaperVaultERC4626.sol` and `ReaperStrategyGranarySupplyOnly.sol` are main contracts of the project. But NatSpec is missing in the code. 

It is recommended to use the NatSpec for commenting for better readability and understanding of the code base.

## 13. `want` IS AN ADDRESS AND NO NEED TO RECAST IT BACK INTO AN ADDRESS

        ILendingPool(ADDRESSES_PROVIDER.getLendingPool()).withdraw(address(want), _withdrawAmount, address(this)); 
		
Above line of code can be rewritten as follows (by removing the address recast of want):

        ILendingPool(ADDRESSES_PROVIDER.getLendingPool()).withdraw(want, _withdrawAmount, address(this)); 
		
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L200
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L232