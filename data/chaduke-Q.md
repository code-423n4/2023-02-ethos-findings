QA1.  The ``updateDistributionPeriod()`` function has no range check for the input ``_newDistributionPeriod``. As a result, it might cause an overflow for L112 of function ``fund()``: 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101-L117](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101-L117)

In addition, as ``_newDistributionPeriod`` is an important configuration parameter, a timelock should be introduced  help a smooth transition. 

Mitigation: 1) introduce a range check for ``_newDistributionPeriod``; 2) introduce a timelock delay; 


QA2. ``issueOath()`` and ``fund()`` have a division-followed-by-multiplication rounding error issue, because the ``fund()`` performs a division first to calculate ``rewardPerSecond`` and then   ``issueOath()`` uses a multiplication to calculate the amount ``issuance`` to issue. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L84-L110](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L84-L110)

Mitigation: we multiply ``rewardPerSecond`` by WAD and then divide ``issuance`` by WAD afterwards, where WAD = 1e18:

```diff
 function issueOath() external override returns (uint issuance) {
        _requireCallerIsStabilityPool();
        if (lastIssuanceTimestamp < lastDistributionTime) {
            uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;
            uint256 timePassed = endTimestamp.sub(lastIssuanceTimestamp);
-            issuance = timePassed.mul(rewardPerSecond);
+            issuance = timePassed.mul(rewardPerSecond)/WAD;
            totalOATHIssued = totalOATHIssued.add(issuance);
        }

        lastIssuanceTimestamp = block.timestamp;
        emit TotalOATHIssuedUpdated(totalOATHIssued);
    }

    /*
      @dev funds the contract and updates the distribution
      @param amount: amount of $OATH to send to the contract
    */
    function fund(uint amount) external onlyOwner {
        require(amount != 0, "cannot fund 0");
        OathToken.transferFrom(msg.sender, address(this), amount);

        // roll any unissued OATH into new distribution
        if (lastIssuanceTimestamp < lastDistributionTime) {
            uint timeLeft = lastDistributionTime.sub(lastIssuanceTimestamp);
            uint notIssued = timeLeft.mul(rewardPerSecond);
            amount = amount.add(notIssued);
        }


-        rewardPerSecond = amount.div(distributionPeriod);
+       rewardPerSecond = amount.div(distributionPeriod)*WAD;
        lastDistributionTime = block.timestamp.add(distributionPeriod);
        lastIssuanceTimestamp = block.timestamp;

        emit LogRewardPerSecond(rewardPerSecond);
}
```

QA3. The ``permit()`` function fails to check v ∈ {27, 28}, so it might be subject to signature malleability.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L262-L290

Mitigation: 
Use Zeppelin's ECDSA: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol

QA4. The ``BorrowerOperations`` contract might not work for fees-on-transfer collateral tokens. 

For example, the function ``_moveTokensAndCollateralFromAdjust calls ``safeTransferFrom()`` at L 497 to try to move ``_collchange`` amount of collateral tokens to the ``BorrowerOperations`` contract, however, the contract might receive less amount due to fees-on-transfer, as a result, the next statement that pull ``_collChange``, the same amount, to the ``_activePool`` will fail due to insufficient balance of collateral tokens. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L476-L502](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L476-L502)

Mitigation: always measure the  balance before and after transfer to decide the amount that has been transferred for fees-on-transfer tokens. 

QA5: ``_computeNominalCR()`` assumes the debt token has a 18 decimals. Need to make this explicit or adjust the debit tokens to 18 decimals as well. 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/Dependencies/LiquityMath.sol#L97-L106