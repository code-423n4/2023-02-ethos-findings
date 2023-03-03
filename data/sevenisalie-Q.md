#DOS when `withdrawalQueue` length is sufficiently large

[ReaperVaultV2.sol#L258-L271](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L258)

##Description of the Vulnerability

the vulnerability allows a potentially compromised admin wallet to perform an critical denial-of-service (DoS) attack on the contract by exploiting a loop within a function. The loop is dependent on the length of the input parameter of the function, meaning that if an array of sufficient size is accidentally or purposely provided, the loop will iterate too many times and cause the contract to fail. This vulnerability can be exploited by an attacker to consume significant amounts of computational resources of the network.

##Technical Details

The vulnerable function contains a loop that iterates over each element of an input array, performing a certain action on each element. The code block below demonstrates a simplified example of the vulnerable function:

```solidity
function vulnerableFunction(uint256[] calldata inputArray) external {
    for (uint256 i = 0; i < inputArray.length; i++) {
        // Perform action on inputArray[i]
    }
}
```

The issue with this code is that an attacker can craft a malicious input array with a large number of elements, causing the loop to iterate many times and consume significant amounts of gas. Even more critically, the `_withdraw()` function core logic relies on the same `withdrawalQueue` length; meaning that if a sufficiently large enough array of calldata is passed through `setWithdrawalQueue` users would also lose access to their funds.


##Recommendation

While the feasibility of a compromised admin wallet is low, all risks can be taken off the table with a small fix. To mitigate this vulnerability, the vulnerable function should be modified to limit the number of iterations in the loop, regardless of the length of the input array; such as a require statement. In other implementations of fractionally reserved MCD the `withdrawalQueue` is limited to a length of 20 to avoid this problem. 