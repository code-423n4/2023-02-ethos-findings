- `vars.yieldingAmount = yieldingAmount[_collateral];` can be read from `vars.currentAllocated` in ActivePool.rebalance
    
    ```solidity
    function _rebalance(address _collateral, uint256 _amountLeavingPool) internal {
      LocalVariables_rebalance memory vars;
    
      // how much has been allocated as per our internal records?
      vars.currentAllocated = yieldingAmount[_collateral];
      
    	...
    
    -- vars.yieldingAmount = yieldingAmount[_collateral];
    ++ vars.yieldingAmount = vars.currentAllocated
    ```
    
    [link](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol/#L263)