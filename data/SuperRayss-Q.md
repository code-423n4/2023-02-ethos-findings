# Issues Found
## Ethos-Core\contracts\LQTY\CommunityIssuance.sol
There is a loss of precision in calculating `rewardPerSecond`. This is due to `rewardPerSecond` being assigned the result of the lossy division of `distributionPeriod` from `amount` (line 112). This loss in precision results in a growing deviation between expected and actual issuance of $Oath tokens to the stability pool.
## Ethos-Core\test\LQTYIssuanceArithmeticTest.js
Test case on line 170 (“aggregates multiple funding rounds within a distribution period”) passes when it shouldn’t due to improper usage of arithmetic operators and type conversion on BigNumber variables, resulting in the test case comparing the integers 4 and 3 (line 181), rather than their intended values. When corrected, this test case does not pass with the current implementation of `fund()` in CommunityIssuance.sol due to loss of precision with calculating `rewardPerSecond`. Running the corrected test case fails with `AssertionError: expected 2611200 [Wei] to be at most 1000 [Wei]`. The corrected test case used is as follows (refer to Proposed Changes section for the fully overhauled test case).
```
  it("aggregates multiple funding rounds within a distribution period", async () => {
    await setupFunder(accounts[0], oathToken, communityIssuanceTester, million);
    await communityIssuanceTester.fund(thousand);
    const rps1 = await communityIssuanceTester.rewardPerSecond();
    await th.fastForwardTime(604800, web3.currentProvider); // 7 days pass
    await communityIssuanceTester.fund(thousand);
    const rps2 = await communityIssuanceTester.rewardPerSecond();
    await th.fastForwardTime(604800, web3.currentProvider);
    await communityIssuanceTester.fund(thousand);
    const rps3 = await communityIssuanceTester.rewardPerSecond();
    const final = rps1.mul(th.toBN(604800)).add(rps2.mul(th.toBN(604800))).add(rps3.mul(th.toBN(604800)));
    th.assertIsApproximatelyEqual(final, thousand.mul(th.toBN(3)), 1000);
  })
```
# Proposed Changes
## Ethos-Core\contracts\LQTY\CommunityIssuance.sol
### Use `DECIMAL_PRECISION` (inherited from BaseMath.sol contract) when assigning `rewardPerSecond` to minimize loss of precision
Line 44 (optional, but recommended): `uint internal rewardPerSecond;`
Line 112: `rewardPerSecond = amount.mul(DECIMAL_PRECISION).div(distributionPeriod);`

### Create new function `getRewardPerSecond` instead of accessing `rewardPerSecond` property directly
```
  function getRewardPerSecond() public view returns (uint) {
    return rewardPerSecond.div(DECIMAL_PRECISION);
  }
```
### Create new function `getRewardAmount` to calculate rewards generated over a provided number of seconds
```
  function getRewardAmount(uint seconds_) public view returns (uint) {
    return rewardPerSecond.mul(seconds_).div(DECIMAL_PRECISION);
  }
```
### Implement `getRewardPerSecond` and `getRewardAmount`
Line 89: `issuance = getRewardAmount(timePassed);`
Line 108: `uint notIssued = getRewardAmount(timeLeft);`
Line 116: `emit LogRewardPerSecond(getRewardPerSecond());`
## Ethos-Core\test\LQTYIssuanceArithmeticTest.js
### Overhaul test cases to work as intended and be compatible with CommunityIssuance.sol changes, using `getRewardAmount` for the highest possible precision, rather than multiplying time elapsed by `getRewardPerSecond`
```
  it("Issues a set amount of rewards per second", async () => {
    await setupFunder(accounts[0], oathToken, communityIssuanceTester, million);
    await communityIssuanceTester.fund(thousand);
    const blockTimestampOne = await th.getLatestBlockTimestamp(web3);
    await th.fastForwardTime(100, web3.currentProvider);
    await communityIssuanceTester.unprotectedIssueLQTY();
    const issuance = await communityIssuanceTester.totalOATHIssued();
    const blockTimestampTwo = await th.getLatestBlockTimestamp(web3);
    const difference = blockTimestampTwo - blockTimestampOne;
    const rewards = await communityIssuanceTester.getRewardAmount(difference);
    assert.isTrue(issuance.eq(rewards));
  })
```
```
  it("aggregates multiple funding rounds within a distribution period", async () => {
    await setupFunder(accounts[0], oathToken, communityIssuanceTester, million);
    await communityIssuanceTester.fund(thousand);
    await th.fastForwardTime(604800, web3.currentProvider); // 7 days pass
    await communityIssuanceTester.fund(thousand);
    await th.fastForwardTime(302400, web3.currentProvider); // 3.5 days pass
    await communityIssuanceTester.fund(thousand);
    const final = await communityIssuanceTester.getRewardAmount(1209600); // 14 days - a full distribution period
    const compare = thousand.mul(th.toBN(3));
    th.assertIsApproximatelyEqual(final, compare, 1000);
  })
```
### Implement `getRewardPerSecond` in other tests
Line 107: `const rps = await communityIssuanceTester.getRewardPerSecond();`
Line 189: `const rps = await communityIssuanceTester.getRewardPerSecond();`
## Ethos-Core\contracts\TestContracts\CommunityIssuanceTester.sol
### Implement `getRewardAmount`
Line 12: `issuance = getRewardAmount(timePassed);`