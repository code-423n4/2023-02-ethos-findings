# Ethos Reserve

## [G-01] INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS
### Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.
#### Instances 47
-[BorrowerOperations line 438](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L438)
-[BorrowerOperations line 455](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L455)
-[BorrowerOperations line 476](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L476)
-[BorrowerOperations line 524](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L524) 
-[BorrowerOperations line 533](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L533) 
-[BorrowerOperations line 537](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L537)
-[BorrowerOperations line 546](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L546)
-[BorrowerOperations line 551](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L551)
-[BorrowerOperations line 555](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L555)
-[BorrowerOperations line 567](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L567)
-[BorrowerOperations line 571](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L571)
-[BorrowerOperations line 624](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L624)
-[BorrowerOperations line 636](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L636)
-[BorrowerOperations line 640](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L640)
-[BorrowerOperations line 661](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L661)
-[BorrowerOperations line 682](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L682)
-[TroveManager line 478](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L478)
-[TroveManager line 582](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L582)
-[TroveManager line 671](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L671) 
-[TroveManager line 785](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L785) 
-[TroveManager line 864](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L864) 
-[TroveManager line 1213](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1213)
-[TroveManager line 1321](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1321) 
-[TroveManager line 1339](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1339)
-[TroveManager line 1516](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1516) 
-[TroveManager line 1538](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1538) 
-[ActivePool line 320](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L320) 
-[SortedTroves line 338](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L338) 
-[SortedTroves line 402](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L402) 
-[LQTYStaking line 251](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L251) 
-[LQTYStaking line 261](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L261) 
-[LQTYStaking line 265](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L265) 
-[LQTYStaking line 269](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L269) 
-[LUSDToken line 320](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L320) 
-[LUSDToken line 328](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L328) 
-[LUSDToken line 360](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L360) 
-[LUSDToken line 365](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L365) 
-[LUSDToken line 375](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L375) 
-[LUSDToken line 380](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L380) 
-[LUSDToken line 393](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L393) 
-[ReaperVaultV2 line 462](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L462) 
-[ReaperStrategyGranarySupplyOnly line 86](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L86) 
-[ReaperStrategyGranarySupplyOnly line 176](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176) 
-[ReaperStrategyGranarySupplyOnly line 206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L206) 

## [G-02] `<X> += <Y>` COSTS MORE GAS THAN `<X> = <X> + <Y>` FOR STATE VARIABLES
### Using the addition operator instead of plus-equals saves gas
#### Instances 19

-[ReaperVaultV2 line 168](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168) 
-[ReaperVaultV2 line 194](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194)
-[ReaperVaultV2 line 196](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L196)
-[ReaperVaultV2 line 214](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L214)
-[ReaperVaultV2 line 390](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L390)
-[ReaperVaultV2 line 391](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L391)
-[ReaperVaultV2 line 395](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L395)
-[ReaperVaultV2 line 396](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L396)
-[ReaperVaultV2 line 444](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L444)
-[ReaperVaultV2 line 445](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L445)
-[ReaperVaultV2 line 450](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L450)
-[ReaperVaultV2 line 451](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L451)
-[ReaperVaultV2 line 452](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L452)
-[ReaperVaultV2 line 505](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L505)
-[ReaperVaultV2 line 514](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L514)
-[ReaperVaultV2 line 515](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L515)
-[ReaperVaultV2 line 516](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L516)
-[ReaperVaultV2 line 520](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L520)
-[ReaperVaultV2 line 521](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L521) 

## [G-03] CAN DECLARE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS
### Gas is not wasted on redeclaration of the variable for every iteration.
#### Instances 18

-[CollateralConfig line 57](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L57)
-[CollateralConfig line 63](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L63)
-[TroverManager line 610](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L610)
-[ActivePool line 109](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L109)
-[ActivePool line 110](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L110)
-[LQTYStaking line 207](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L207)
-[LQTYStaking line 208](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L208) 
-[LQTYStaking line 229](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L229) 
-[LQTYStaking line 242](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L242) 
-[ReaperVaultV2 line 265](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L265)
-[ReaperVaultV2 line 378](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L378) 
-[ReaperVaultV2 line 379](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L379) 
-[ReaperVaultV2 line 384](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L384) 
-[ReaperVaultV2 line 385](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L385) 
-[ReaperVaultV2 line 386](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L386) 
-[ReaperStrategyGranarySupplyOnly line 118](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L118) 
-[ReaperStrategyGranarySupplyOnly line 120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L120) 
-[ReaperStrategyGranarySupplyOnly line 166](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L166) 

## [G-04] EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT
#### Instances 18

-[ReaperStrategyGranarySupplyOnly line 25](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L25) 
-[ReaperStrategyGranarySupplyOnly line 27](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L27) 
-[ReaperStrategyGranarySupplyOnly line 29](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L29) 
-[ReaperVaultV2 line 40](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40) 
-[ReaperVaultV2 line 73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73) 
-[ReaperVaultV2 line 74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74) 
-[ReaperVaultV2 line 75](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L75) 
-[ReaperVaultV2 line 76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L76) 
-[TroveManager line 58](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L58) 
-[TroveManager line 60](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L60) 
-[CollateralConfig line 21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L21) 
-[CollateralConfig line 25](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L25) 
-[ReaperBaseStrategyv4 line 24](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L24) 
-[ReaperBaseStrategyv4 line 25](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L25) 
-[ReaperBaseStrategyv4 line 49](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49) 
-[ReaperBaseStrategyv4 line 50](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50) 
-[ReaperBaseStrategyv4 line 51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51) 
-[ReaperBaseStrategyv4 line 52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52) 

## [G-05] ADD UNCHECKED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT
### Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.
### `require(a <= b); x = b - a` => `require(a <= b); unchecked { x = b - a }`
#### Instances 16

-[TroveManager line 494](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L494)
-[TroveManager line 496](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L496)
-[ReaperVaultV2 line 235](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L235)
-[ReaperVaultV2 line 244](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L244)
-[ReaperVaultV2 line 245](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L245)
-[ReaperVaultV2 line 384](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L384)
-[ReaperVaultV2 line 451](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L451)
-[ReaperVaultV2 line 526](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L526)
-[ReaperVaultV2 line 528](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L528)
-[ReaperVaultERC4626 line 82](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L82)
-[ReaperVaultERC4626 line 125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L125)
-[ReaperBaseStrategyv4 line 121](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L121)
-[ReaperBaseStrategyv4 line 123](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L123)
-[ReaperStrategyGranarySupplyOnly line 81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L81)
-[ReaperStrategyGranarySupplyOnly line 93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L93)
-[ReaperStrategyGranarySupplyOnly line 100](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L100)
-[ReaperStrategyGranarySupplyOnly line 132](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L132)

## [G-06] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
### Do not use return at the end of the function
#### Instances 16

-[ReaperStrategyGranarySupplyOnly line 104-106](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L104-L106) 
-[ReaperVaultERC4626 line 29-30](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L29-L30) 
-[ReaperVaultERC4626 line 37-38](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L37-L38) 
-[ReaperVaultERC4626 line 51-53](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51-L53) 
-[ReaperVaultERC4626 line 66-68](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66-L68) 
-[ReaperVaultERC4626 line 79-82](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79-L82) 
-[ReaperVaultERC4626 line 96-97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L96-L97) 
-[ReaperVaultERC4626 line 122-125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122-L125) 
-[ReaperVaultERC4626 line 165-166](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L165-L166) 
-[ReaperVaultERC4626 line 220-221](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L220-L221) 
-[ReaperVaultERC4626 line 240-241](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L240-L241) 
-[TroveManager line 383-437](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L383-L437) 
-[TroveManager line 451-605](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L451-L605) 
-[TroveManager line 1196-1226](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1196-L1226) 
-[TroveManager line 1859-1862](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1859-L1862) 
-[TroveManager line 1867-1879](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1867-L1879) 

## [G-07] REQUIRE() OR REVERT() STATEMENTS SHOULD BE USED SORTED FROM CHEAPEST TO MOST EXPENSIVE
### Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.
#### Instances 13

In instances given below you need either to swap 2 lines or split 1 require (left statement and right statement relative to &&) into 2 require statements and put them in sorted order in order to save gas

-[CollateralConfig line 51-54](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L51-L54) 
-[BorrowerOperations line 653](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653) 
-[TroveManager line 1279](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L1279) 
-[TroveManager line 1342](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1342) 
-[TroveManager line 1539](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539) 
-[SortedTroves line 111-115](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L111-L115) 
-[SortedTroves line 214-216](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L214-L216) 
-[LUSDToken line 347](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347) 
-[LUSDToken line 352](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L352) 
-[ReaperVaultV2 line 150-156](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L150-L156) 
-[ReaperVaultV2 line 180-181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L180-L181) 
-[ReaperVaultV2 line 321-322](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L321-L322) 
-[ReaperBaseStrategyv4 line 96-98](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L96-L98) 

## [G-08] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE
### Before transfer, we should check for amount being 0 so the function doesn't run when it's not going to do anything
#### Instances 13

-[ActivePool line 186](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L186) 
-[ActivePool line 210](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L210) 
-[BorrowerOperations line 231](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L231)
-[CommunityIssuance line 127](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127)
-[LQTYStaking line 171](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L171)
-[LUSDToken line 198](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L198)
-[LUSDToken line 203](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L203) 
-[LUSDToken line 222](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L222) 
-[LUSDToken line 237](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L237) 
-[LUSDToken line 238](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L238) 
-[LUSDToken line 410](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L410) 
-[LUSDToken line 526](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L526) 
-[LUSDToken line 528](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L528) 

## [G-09] OPTIMIZE NAMES TO SAVE GAS
### `public`/`external` function names and public member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted.
#### Instances 12

-[IActivePool line 8](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Interfaces/IActivePool.sol#L8) 
-[IBorrowerOperations line 6](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Interfaces/IBorrowerOperations.sol#L6) 
-[ICollateralConfig line 5](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Interfaces/ICollateralConfig.sol#L5) 
-[ICommunityIssuance line 5](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Interfaces/ICommunityIssuance.sol#L5) 
-[ILQTYStaking line 5](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Interfaces/ILQTYStaking.sol#L5) 
-[ILUSDToken line 8](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Interfaces/ILUSDToken.sol#L8) 
-[ISortedTroves line 6](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Interfaces/ISortedTroves.sol#L6) 
-[ITroveManager line 14](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Interfaces/ITroveManager.sol#L14) 
-[ReaperVaultV2 line 21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L21) 
-[ReaperVaultERC4626 line 12](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L12) 
-[ReaperBaseStrategyv4 line 14](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L14) 
-[ReaperStrategyGranarySupplyOnly line 19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L19) 

## [G-10] USE A MORE RECENT VERSION OF SOLIDITY
### Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining. 
### Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads. 
### Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.
#### Instances 12

-[ActivePool line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3) 
-[BorrowerOperations line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3) 
-[CollateralConfig line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3) 
-[CommunityIssuance line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3) 
-[LQTYStaking line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3) 
-[LUSDToken line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3) 
-[SortedTroves line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L3) 
-[TroveManager line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3) 
-[ReaperVaultV2 line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3) 
-[ReaperVaultERC4626 line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3) 
-[ReaperBaseStrategyv4 line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3) 
-[ReaperStrategyGranarySupplyOnly line 3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3) 

## [G-11] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE
### Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.
#### Instances 6

-[TroverManager line 94-91](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L91-L94) 
-[TroverManager line 104-105](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L104-L105) 
-[TroverManager line 119-120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L119-L120) 
-[ActivePool line 44-45](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L44-L45) 
-[LQTYStaking line 25, 28, 32](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25-L32) 
-[LUSDToken line 62-64](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L62-L64) 

## [G-12] STACK VARIABLE USED AS A CHEAPER CACHE FOR A STATE VARIABLE IS ONLY USED ONCE
#### Instances 6

-[TroveManager line 1557](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1557) 
-[TroveManager line 1570](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1570) 
-[TroveManager line 1602](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1602) 
-[TroveManager line 1694](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1694) 
-[CommunityIssuance line 107](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L107) 
-[LQTYStaking line 219](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L219) 

## [G-13] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD
### The following functions could be set external to save gas and improve code quality. External call cost is less expensive than public functions.
#### Instances 4

-[ReaperStrategyGranarySupplyOnly line 62](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62) 
-[ReaperVaultV2 line 295](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L295) 
-[TroverManager line 1045](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1045) 
-[TroverManager line 1440](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1440) 

## [G-14] PACK MULTIPLE VARIABLES INTO FEWER SLOTS
### If the sum of the sizes of several variables is less than or equal to 32 bytes they can be packed in one storage slot. To pack them you need to place them next to each other. 
### Move closer the lines indicated for each contract
#### Instances 2

-[ReaperBaseStrategyv4 line 28<->58](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L28-L58) 
-[ReaperVaultV2 line 49<->78](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L49-L78) 

## [G-15] USAGE OF UINTS/INTS SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD
### When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. 

[link](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html ) 

### Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

#### Instances 1

-[TroveManager line 87](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L87) 

## [G-16] SHOULD USE ARGUMENTS INSTEAD OF STATE VARIABLE
### This will save about 97 gas
#### Instances 1

-[ReaperVaultV2 line 581](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L581) 

## [G-17] SUPERFLUOUS EVENT FIELDS
### `block.number` and `block.timestamp` are added to the event information by default, so adding them manually will waste additional gas.
#### Instances 1

-[TroveManager line 2126](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L2126) 
