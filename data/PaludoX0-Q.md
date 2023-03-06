|  | Issue |  
| --- | --- | 
| [L-01] | Provide a gap at the bottom of upgradeable contracts |
| [L-02] | Function state mutability can be changed to pure | 
| [L-03] | Deployer msg.sender receives DEFAULT_ADMIN_ROLE  |  
| [L-04] | Missing address validation in function withdraw | 
| [L-05] | Use existing function instead of repeating code  | 
| [L-06] | Two step process to change governance address | 
| [N-01] | **Emit** keyword missing | 
| | Instance of Issues not highlighted in 4NALYZER |
|  |  |  


# [L-01] Provide a gap at the bottom of upgradeable contracts
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L277
Since the contract is meant to be upgradeable and inherited by **ReaperStrategyGranarySupplyOnly.sol** it is suggested to add a gap at the end of the contract
    `uint256[50] private __gap;`

This is the storage layout and can be seen there's no gap between `vault` which is last state variable of contract **ReaperBaseStrategyv4**, `veloSwapPath` which is the only state variable of contract **VeloSolidMixin** and `gWant` which is the first variable of **ReaperStrategyGranarySupplyOnly**
| Name                 | Type                                                           | Slot |
|----------------------|----------------------------------------------------------------|------|
| _initialized         | uint8                                                          | 0    | 
| _initializing        | bool                                                           | 0    | 
| __gap                | uint256[50]                                                    | 1    | 
| __gap                | uint256[50]                                                    | 51   | 
| __gap                | uint256[50]                                                    | 101  | 
| __gap                | uint256[50]                                                    | 151  | 
| _roles               | mapping(bytes32 => struct AccessControlUpgradeable.RoleData)   | 201  | 
| __gap                | uint256[49]                                                    | 202  | 
| _roleMembers         | mapping(bytes32 => struct EnumerableSetUpgradeable.AddressSet) | 251  | 
| __gap                | uint256[49]                                                    | 252  | 
| want                 | address                                                        | 301  | 
| emergencyExit        | bool                                                           | 301  | 
| lastHarvestTimestamp | uint256                                                        | 302  | 
| upgradeProposalTime  | uint256                                                        | 303  | 
| vault                | address                                                        | 304  |
| veloSwapPath         | mapping(address => mapping(address => address[]))              | 305  | 
| gWant                | contract IAToken                                               | 306  |
| rewardClaimingTokens | address[]                                                      | 307  | 
| steps                | address[2][]                                                   | 308  | 

# [L-02] Function state mutability can be changed to pure
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L203
View function can be changed to pure because it doesn't write and neither read from state.

# [L-03] Deployer msg.sender receives DEFAULT_ADMIN_ROLE  
Ethos-Vault/contracts/ReaperVaultV2.sol#L132
According to contract comments #L66 **"The DEFAULT_ADMIN_ROLE (in-built access control role) will be granted to a multisig requiring 4 signatures."**
In constructor #L132 the `DEFAULT_ADMIN_ROLE` is granted also contract deployer. 
This is seems not to be strictly necessary, it makes easier the bootstrapping of contract at begininng.
After deployment the deployer shall remember to call function `renounceRole(bytes32,address)`.

# [L-04] Missing address validation in function withdraw
Ethos-Vault/contracts/ReaperVaultERC4626.sol#L202
In `function withdraw( uint256 assets, address receiver, address owner )` in some way all arguments are validate apart `receiver`. 
`owner` is validate in **ERC20.function _approve** while `assets` are validate in **ReaperVaultV2.function _withdraw** by means of `require(_shares != 0, "Invalid amount")`
Please consider to add `require(receiver != 0, "Invalid receiver")`

# [L-05] Use existing function instead of repeating code 
Ethos-Vault/contracts/ReaperVaultV2.sol#L365
In function _withdraw at #L365  there's the following code `value = (_freeFunds() * _shares) / totalSupply();` 
It is already implemented the  function `convertToAssets` that returns the same result and checks if totalSupply() == 0 and in case returns 0  
    
    function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
        if (totalSupply() == 0) return shares;
        return (shares * _freeFunds()) / totalSupply();
    }

Please consider to substitute `value = (_freeFunds() * _shares) / totalSupply();` with `value = convertToAssets(uint256 _shares)` so that in case you change conversion method you don't need to change it everywhere in the code risking to forget somewhere.




# [L-06] Two step process to change governance address
Ethos-Core/contracts/LUSDToken.sol#L146
Function `updateGovernance(address _newGovernanceAddress)` can be called by Governance only, a wrong input parameter could led to losing governance control of the token.
It's strongly recommended to implement a two steps process where for completing ownership change, the new owner shall "accept" ownership. 
As reference use OpenZeppelin ownership change.


# [N-01] **Emit** keyword missing
Ethos-Core/contracts/ActivePool.sol#L194 and L201
`Emit` keyword is Missing for `ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);`
This is a solidity bug that has been fixed with version 0.8.4 
Check for "Type Checker: Fix missing error when events are used without an emit statement." https://github.com/ethereum/solidity/releases/tag/v0.8.4


# Instance of Issues not highlighted in 4NALYZER
The following issue type is reported in https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae but this occurence has not been reported
[NC-1] Missing checks for address(0) when assigning values to address state variables
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L72 and 73
Please consider to validate want and vault addresses, in fact in deployed optimism contracts this are 0 and there are not methods to change them

