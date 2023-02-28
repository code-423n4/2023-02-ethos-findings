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

# LQTY issued can not be obtained by later depositors and needs to be fixed by calling `fund`  

`issueOath` can return 0 witch will lead to [`_triggerLQTYIssuance`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L416) and [`_updateG`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L421) not working.

As they say in the Ethos contract, they already expect that in [Ethos-Core/contracts/StabilityPool.sol/L421](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L421)

        * When total deposits is 0, G is not updated. In this case, the LQTY issued can not be obtained by later
        * depositors - it is missed out on, and remains in the balanceof the CommunityIssuance contract.

But the issue here is that will remain in this stuck state until someone calls `fund` in [CommunityIssuance/L101](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101). The only one that can call this is the **owner**. 
In the function bellow [CommunityIssuance/L84](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L84-L95), if `lastIssuanceTimestamp` > `lastDistributionTime` (this can be done if the [fund](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101) is not called for more than 14 days) then the function will only emit an event and set the new `lastIssuanceTimestamp` to `block.timestamp` witch again will be larger than 14 days. In this case it will also not return any issuance since the `if` is preventing from `issuance` to be calculated.

    function issueOath() external override returns (uint issuance) {
        _requireCallerIsStabilityPool();
        if (lastIssuanceTimestamp < lastDistributionTime) {
            uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;
            uint256 timePassed = endTimestamp.sub(lastIssuanceTimestamp);
            issuance = timePassed.mul(rewardPerSecond);
            totalOATHIssued = totalOATHIssued.add(issuance);
        }

        lastIssuanceTimestamp = block.timestamp;
        emit TotalOATHIssuedUpdated(totalOATHIssued);
    }

The only was for this to be fixed is the owner to call [`fund`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101) witch will update `lastDistributionTime` and it will make it larger than `lastIssuanceTimestamp`.

        rewardPerSecond = amount.div(distributionPeriod);
        lastDistributionTime = block.timestamp.add(distributionPeriod);
        lastIssuanceTimestamp = block.timestamp;

My suggestion is to is to also update the `lastDistributionTime` in `issueOath` and in this way the function will remain working or at least reduce the likelihood of this happening.






