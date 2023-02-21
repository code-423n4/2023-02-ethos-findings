Using vulnerable dependency of OpenZeppelin ! 

The package.json at `Ethos-Core/package.json` configuration file says that the project is using 3.3.0 of OZ which has a not last update version and vulnerable to so many security issues !
```
  "devDependencies": {
    "@nomiclabs/hardhat-ethers": "^2.0.2",
    "@nomiclabs/hardhat-etherscan": "^2.1.2",
    "@nomiclabs/hardhat-truffle5": "^2.0.0",
    "@nomiclabs/hardhat-web3": "^2.0.0",
    "@openzeppelin/contracts": "^3.3.0",
```
ref
https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts