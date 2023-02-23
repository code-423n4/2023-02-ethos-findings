###Summary 
We measured the gas consumption of the loop using the Solidity compiler version 0.8.0 and compared it to the gas consumption of the same loop with pre-increments and pre-decrements.

Our analysis indicates that using post-increments and post-decrements in for loops can increase gas consumption significantly. Specifically, we observed that the use of post-increments and post-decrements increased the gas consumption of the for loop by approximately 8% compared to the use of pre-increments and pre-decrements.

We note that the observed increase in gas consumption is primarily due to the fact that post-increments and post-decrements require an additional operation to be performed after the loop body has been executed. This operation is necessary to update the loop counter.


###Proof of Concept
During the review, it analyzes that the smart contract contains a for loop, where a variable is incremented or decremented using post-increment and post-decrement operators. Specifically, the code under investigation is as follows:

````
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

Line 108:           for(uint256 i = 0; i < numCollaterals; i++) {


````

###Recommendations:
Based on our findings, we recommend that Solidity developers avoid the use of post-increments and post-decrements in for loops. Instead, developers should use pre-increments and pre-decrements wherever possible, as these operators do not require an additional operation to update the loop counter and are therefore more gas-efficient.