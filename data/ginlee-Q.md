[N-01] Use Require instead of Assert with error message

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

assert(_collWithdrawal <= vars.coll)

assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0))

require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay.
This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states “The Assert function generates an error of type Panic(uint256).Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. There’s a mistake”.

[N-02] Use uint256 instead of uint to save gas
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

function _requireTroveisActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {
        uint status = _troveManager.getTroveStatus(_borrower, _collateral);
        require(status == 1, "BorrowerOps: Trove does not exist or is closed");
    }

function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {
        uint status = _troveManager.getTroveStatus(_borrower, _collateral);
        require(status != 1, "BorrowerOps: Trove is active");
    }

[N-03] use latest solididy version to declare mapping with named params
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol
    mapping (address => mapping (address => Trove)) public Troves;
    mapping (address => uint) public totalStakes;
    mapping (address => uint) public totalStakesSnapshot;
    mapping (address => uint) public totalCollateralSnapshot;
    mapping (address => uint) public L_Collateral;
    mapping (address => uint) public L_LUSDDebt;
    mapping (address => mapping (address => RewardSnapshot)) public rewardSnapshots;
    mapping (address => address[]) public TroveOwners;
    mapping (address => uint) public lastCollateralError_Redistribution;
    mapping (address => uint) public lastLUSDDebtError_Redistribution;

[N-04] use constants instead of type(uintX).max
it uses more gas in the distribution process and also for each transaction than constant usage

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

IERC20Upgradeable(want).safeApprove(vault, type(uint256).max)

change this to 
uint256 constant MAX_VALUE = 2**256 - 1

[G-01] use "toFree = toFree + profit" instead of "toFree += profit" to save more gas

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

toFree += profit
roi -= int256(loss)


  