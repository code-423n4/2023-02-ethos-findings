let assume that yielding percentage is 70%. total collateral is 5000. so yield vault smart contract will get 3500 and 1500 will remain in active pool address. so now owner change the yielding percentage to 80%.
if the first transaction(let assume withdraw) is 625 after yielding percentage change, the transaction will fail because finalyieldingpercentage is also 80%. some transactions will always fail after yielding percentage change if finalyieldingpercentage is same with yielding percentage.