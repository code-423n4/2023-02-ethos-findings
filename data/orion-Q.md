Price is not checks when opening//closing trove

## Impact
Contract can return unintended results if collateral price is not configured yet

## Proof of Concept
The open/close trove functionalities takes the price of the collateral from the price feed contract :

```
 vars.price = priceFeed.fetchPrice(_collateral);
...

uint price = priceFeed.fetchPrice(_collateral);
```

However the prices are used direcctly with the logic with no checks that these are higher than 0, when opening the trove may allows the attacker to get the collateral higher than he is supposed to be when the collateral is not registeres on chainlink or tellor, the same when closing the trove if the price is moved to 0  

## Tools Used
Manual

## Recommended Mitigation Steps :
Once the price is fetched from the priceFeed it's better to check that it's valid (higher than 0)

