# Update imports

Dependencies  in every contract need to be expanded. In the current implementations we see from where the things are imported, but that does not tell us what is imported. For more clearer and simpler code I suggest that you can update the imports as follows.

    import "./Interfaces/ILUSDToken.sol";
    import "./Interfaces/ITroveManager.sol";
    import "./Dependencies/SafeMath.sol";
    import "./Dependencies/CheckContract.sol";
    import "./Dependencies/console.sol";

Change it to this:

    import {console} from "./Dependencies/console.sol";
    import {SafeMath} from "./Dependencies/SafeMath.sol";
    import {ILUSDToken} from "./Interfaces/ILUSDToken.sol";
    import {ITroveManager} from "./Interfaces/ITroveManager.sol";
    import {CheckContract} from "./Dependencies/CheckContract.sol";


# Use more modern version of solidity 

In some of the contract, especially the newer ones, we see the use of sol 0.8, but in other contract we see sol 0.6.11. Newer versions are more secure and come with enhanced features, like safeMath implemented in ^0.8  . It is really important solidity contracts to be up to date, so their functionality wont be impaired but their shortcomings. Suggestion is to upgrade the current versions to at least 0.8.


# No checks if the arguments are contracts


In the constructor there are no contracts checks if any of the given arguments are in fact full contract. Although it is really unlikely for those addresses not to be contract, since the developers are gonna be careful with the deployment, but at the end one mistake and the contract needs to be redeployed, thus costing enormous fees for gas.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111

There is a great implementation on how this can be cheeked and even ethos checks it but only in some contract.
in https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/CheckContract.sol#L11
    
    function checkContract(address _account) internal view {
        require(_account != address(0), "Account cannot be zero address");

        uint256 size;
        // solhint-disable-next-line no-inline-assembly
        assembly { size := extcodesize(_account) }
        require(size > 0, "Account code size cannot be zero");
    }

# Changing governance should always be 2 step procedure 
Although really unlikely  there could be a mistake that governance could be upgraded to a faulty  or not fully functioning  contract. There is a check for if the upgraded instance if it is a contract, but to be more sure that everything will turn out fine I suggest that there should be a function to `allow` for a new contract to claim the governance and a function in the new contract to `claim` it. This will prevent not fully optimized contracts to be falsely assigned to be the new governor.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L146-L157

    function updateGovernance(address _newGovernanceAddress) external {
        _requireCallerIsGovernance();
        checkContract(_newGovernanceAddress); // must be a smart contract (multi-sig, timelock, etc.)
        governanceAddress = _newGovernanceAddress;
        emit GovernanceAddressChanged(_newGovernanceAddress);
    }

    function updateGuardian(address _newGuardianAddress) external {
        _requireCallerIsGovernance();
        checkContract(_newGuardianAddress); // must be a smart contract (multi-sig, timelock, etc.)
        guardianAddress = _newGuardianAddress;
        emit GuardianAddressChanged(_newGuardianAddress);
    }



