QA1.  The ``updateDistributionPeriod()`` function has no range check for the input ``_newDistributionPeriod``. As a result, it might cause an overflow for L112 of function ``fund()``: 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101-L117](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101-L117)

In addition, as ``_newDistributionPeriod`` is an important configuration parameter, a timelock should be introduced  help a smooth transition. 

Mitigation: 1) introduce a range check for ``_newDistributionPeriod``; 2) introduce a timelock delay; 


QA2. ``issueOath()`` and ``fund()`` have a division-followed-by-multiplication rounding error issue, because the ``fund()`` performs a division first to calculate ``rewardPerSecond`` and then   ``issueOath()`` uses a multiplication to calculate the amount ``issuance`` to issue. For example, there will be around $871 lost for distributing 1M over the given 14 days period using this division-followed-by-multiplication approach. 

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

QA6. Nothing concrete regarding ``lqtystaking`` is implemented now in ``TroveManager.sol``, so state variable or event can be deleted regarding ``lqtystaking`` or complete the implementation if necessary.
```javascript
    address public lqtyStakingAddress;
```

QA7. ``updateGovernance()`` and  ``updateGuardian()`` use only one step to change address. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L146-L158](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L146-L158)

Changing adminS in one step is risky: if the new address is a wrong input by mistake, we will lose all the privileges of the owner. 

Recommendation:  Use OpenZeppelin's Ownable2Step. https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol

QA8. There is no way to revoke a permit of approval. It the approval permit was given to the wrong person or with the wrong amount, a revocation is necessary. 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L262-L290

Mitigation: One can revoke a permit by increasing the nonce number. 

QA9. ``permit()`` will bypass signature check when ``owner = 0x0``, allowing allowance approval from the zero address and possible other future exploits (such as transfer LUSD from zero address to the attacker's address)

For example, Bob can call ``permit(address(0), Bob, balanceOf(adress(0)), deadline, v, r, s)`` to get an allowance from zero address with the amount of the LUSD balance of the zero address. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L262-L294](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L262-L294)

He will pick a value ``s`` such that it will pass the check
```javascript
if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {
            revert('LUSD: Invalid s value');
        }
```

Pick a future deadline so that it will pass
```javascript
require(deadline >= now, 'LUSD: expired deadline');
```

Pick v = 27, and the value of r does not matter. 

Then ``recoveredAddress = 0`` since it will be an invalid signature. 

However, the following two lines will pass and as  a result, Bob gets the needed allowance from the Zero address.
```javascript
      require(recoveredAddress == owner, 'LUSD: invalid signature');
        _approve(owner, spender, amount);
```

Mitigation: 1) add a check that ``recoveredAddress != 0`` and 2) use Zeppelin's [ECDSA](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol).  