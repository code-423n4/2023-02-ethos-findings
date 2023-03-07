# No Maximum (or Minimum) on distribution period value

 In `CommunityIssuance.sol`, the `distributionPeriod` is not being having a minimum and maximum value (or range limitation). 

Recommendation is to have a range value (min - max) to ensure the distribution period is in valid and meaningful value.

```js
File: CommunityIssuance.sol
120:     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
121:         distributionPeriod = _newDistributionPeriod;
122:     }
```

# Usage of `ecrecover` allows signature malleability

The `permit` function of LUSDToken contract use the Solidity ecrecover function directly to verify the given signatures. However, the ecrecover EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible here since the nonce is increased each time, ensuring the signatures are not malleable is considered a best practice (and so is checking `recoveredAddress != address(0)`, where `address(0)` means an invalid signature).

It's recommended to use the `recover` function from [OpenZeppelin’s ECDSA library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol) for signature verification.

```js
File: LUSDToken.sol
262:     function permit
263:     (
264:         address owner, 
265:         address spender, 
266:         uint amount, 
267:         uint deadline, 
268:         uint8 v, 
269:         bytes32 r, 
270:         bytes32 s
271:     ) 
272:         external 
273:         override 
274:     {
275:         // EIP-2 still allows signature malleability for ecrecover(). Remove this possibility and make the signature
276:         // unique. Appendix F in the Ethereum Yellow paper (https://ethereum.github.io/yellowpaper/paper.pdf), defines
277:         // the valid range for s in (301): 0 < s < secp256k1n ÷ 2 + 1, and for v in (302): v ∈ {27, 28}. Most
278:         // signatures from current libraries generate a unique signature with an s-value in the lower half order.
279:         if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {
280:             revert('LUSD: Invalid s value');
281:         }
282:         require(deadline >= now, 'LUSD: expired deadline');
283:         bytes32 digest = keccak256(abi.encodePacked('\x19\x01', 
284:                          domainSeparator(), keccak256(abi.encode(
285:                          _PERMIT_TYPEHASH, owner, spender, amount, 
286:                          _nonces[owner]++, deadline))));
287:         address recoveredAddress = ecrecover(digest, v, r, s);
288:         require(recoveredAddress == owner, 'LUSD: invalid signature');
289:         _approve(owner, spender, amount);
290:     }
```


# No Transfer Accept Mechanism for Transfering Ownership

The project is lack of transfer-accept mechanism (two-step procedure) while transfering ownership which can lead into issues. It's best to provide ways to overcome this by adding function to accept the assignment of the ownership.

It’s possible that the governance role mistakenly transfers ownership to the wrong address (`updateGovernance()`), resulting in a loss of the governance role. The `updateGovernance()` function checks the new governance address is contract and proceeds to write the new governance’s address into the state variable. If the nominated contract is not a valid governance contract, it is possible the governance accidentally transfer it to an uncontrolled contract, breaking all functions with the only governance function. Lack of two-step procedure for critical operations leaves them error-prone if the address is incorrect, the new address will take on the functionality of the new role immediately.

```js
File: LUSDToken.sol
146:     function updateGovernance(address _newGovernanceAddress) external {
147:         _requireCallerIsGovernance();
148:         checkContract(_newGovernanceAddress); // must be a smart contract (multi-sig, timelock, etc.)
149:         governanceAddress = _newGovernanceAddress;
150:         emit GovernanceAddressChanged(_newGovernanceAddress);
151:     }
```

# No check for collateral duplicates in `CollateralConfig.sol`

Initialize function in `CollateralConfig.sol` doesn't check if duplicate collateral exist in collateral list.

The collaterals list will contains token addresses which is immutable, if the initialize contains duplicates of token, the function still accept it. It's best to check the uniqueness of collaterals array to make sure it doesnt accept duplicates in collaterals.

```js
File: CollateralConfig.sol
46:     function initialize(
47:         address[] calldata _collaterals,
48:         uint256[] calldata _MCRs,
49:         uint256[] calldata _CCRs
50:     ) external override onlyOwner {
51:         require(!initialized, "Can only initialize once");
52:         require(_collaterals.length != 0, "At least one collateral required");
53:         require(_MCRs.length == _collaterals.length, "Array lengths must match");
54:         require(_CCRs.length == _collaterals.length, "Array lenghts must match");
55:         
56:         for(uint256 i = 0; i < _collaterals.length; i++) {
57:             address collateral = _collaterals[i];
58:             checkContract(collateral);
59:             collaterals.push(collateral);
60: 
61:             Config storage config = collateralConfig[collateral];
62:             config.allowed = true;
63:             uint256 decimals = IERC20(collateral).decimals();
64:             config.decimals = decimals;
65: 
66:             require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
67:             config.MCR = _MCRs[i];
68: 
69:             require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
70:             config.CCR = _CCRs[i];
71: 
72:             emit CollateralWhitelisted(collateral, decimals, _MCRs[i], _CCRs[i]);
73:         }
74: 
75:         initialized = true;
76:     }
```

