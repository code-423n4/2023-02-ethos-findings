
 ==== External Call To User-Supplied Address ====

SWC ID: 107

Severity: Low

Contract: Migrations

Function name: upgrade(address)

PC address: 477

Estimated Gas Usage: 3468 - 38269

A call to a user-supplied address is executed.

An external message call to an address specified by the caller is executed. Note that the callee account might contain arbitrary code and could re-enter any function within this contract. Reentering the contract in an intermediate state may lead to unexpected behaviour. Make sure that no state modifications are executed after this call and/or reentrancy guards are in place.

--------------------

In file: Migrations.sol:23



upgraded.setCompleted(last_completed_migration)



--------------------

Initial State:



Account: [CREATOR], balance: 0x0, nonce:0, storage:{}

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}



Transaction Sequence:



Caller: [CREATOR], calldata: , decoded_data: , value: 0x0

Caller: [CREATOR], function: upgrade(address), txdata: 0x0900f010efefefefefefefefefefefefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeef, value: 0x0

==== Multiple Calls in a Single Transaction ====

SWC ID: 113

Severity: Low

Contract: BorrowerOperations

Function name: getEntireSystemDebt(address)

PC address: 3840

Estimated Gas Usage: 5476 - 75636

Multiple calls are executed in the same transaction.

This call is executed following another call within the same transaction. It is possible that the call never gets executed if a prior call fails permanently. This might be caused intentionally by a malicious callee. If possible, refactor the code such that each transaction only executes one external call or make sure that all callees can be trusted (i.e. they’re part of your own codebase).

--------------------

In file: Dependencies/LiquityBase.sol:63



defaultPool.getLUSDDebt(_collateral)



--------------------

Initial State:



Account: [CREATOR], balance: 0x0, nonce:0, storage:{}

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}



Transaction Sequence:



Caller: [CREATOR], calldata: , decoded_data: , value: 0x0

Caller: [CREATOR], function: setAddresses(address,address,address,address,address,address,address,address,address,address,address), txdata: 0x7985c5e401010101010101010101010101010101010101040104010102010201010101010101010101010101010101010101010101010101010100010101100101402001010101010101010101010101400201800201010101010101010201010100012001010101010101010101010101010101010110100101010420101001408002020101010101010101010101010101010101010101010102010201010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010104040208000101010101010101800401044002100101010101010101010101010101010101108001010101020104100802018001010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101, value: 0x0

Caller: [CREATOR], function: getEntireSystemDebt(address), txdata: 0x27d04b350101010101010101010101010101010201010101010101010101011001010101, value: 0x0



==== External Call To User-Supplied Address ====

SWC ID: 107

Severity: Low

Contract: BorrowerOperations

Function name: claimCollateral(address)

PC address: 10559

Estimated Gas Usage: 2798 - 37694

A call to a user-supplied address is executed.

An external message call to an address specified by the caller is executed. Note that the callee account might contain arbitrary code and could re-enter any function within this contract. Reentering the contract in an intermediate state may lead to unexpected behaviour. Make sure that no state modifications are executed after this call and/or reentrancy guards are in place.

--------------------

In file: BorrowerOperations.sol:414



collSurplusPool.claimColl(msg.sender, _collateral)



--------------------

Initial State:



Account: [CREATOR], balance: 0x0, nonce:0, storage:{}

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}



Transaction Sequence:



Caller: [CREATOR], calldata: , decoded_data: , value: 0x0

Caller: [CREATOR], function: setAddresses(address,address,address,address,address,address,address,address,address,address,address), txdata: 0x7985c5e4010101010101010101010101010101010101010440010101010101010101010101010101010101010101010101010101010101010101010101010102100101010101010101010101010101010101040420021010010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010102010101010101010101010101010101010201028004100202020001010101010101010101010101010101010101010101deadbeefdeadbeefdeadbeefdeadbeefdeadbeef0101010101010101010101010110100101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101, value: 0x0

Caller: [CREATOR], function: claimCollateral(address), txdata: 0x8235b2848484848484848484848484848484848484848484848484848484848484848484, value: 0x0



==== Multiple Calls in a Single Transaction ====

SWC ID: 113

Severity: Low

Contract: BorrowerOperations

Function name: getEntireSystemColl(address)

PC address: 11125

Estimated Gas Usage: 5518 - 75678

Multiple calls are executed in the same transaction.

This call is executed following another call within the same transaction. It is possible that the call never gets executed if a prior call fails permanently. This might be caused intentionally by a malicious callee. If possible, refactor the code such that each transaction only executes one external call or make sure that all callees can be trusted (i.e. they’re part of your own codebase).

--------------------

In file: Dependencies/LiquityBase.sol:56



defaultPool.getCollateral(_collateral)



--------------------

Initial State:



Account: [CREATOR], balance: 0x0, nonce:0, storage:{}

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}



Transaction Sequence:



Caller: [CREATOR], calldata: , decoded_data: , value: 0x0

Caller: [CREATOR], function: setAddresses(address,address,address,address,address,address,address,address,address,address,address), txdata: 0x7985c5e401010101010101010101010102080101010002010140802004020102020401040101010101010101010101010101010101010101010801010280080108080101010101010101010101010101000104010101010101801001010101010101010101010101010101010101010101010101010110100102010040012001100204020101010101010101010101010101010101010101010101010101010110020101010101010101010101010101010101010101018001010101048001010101010101010101010101010101010102208001010101010101018001021002400410020101010101010101010101010101010101404001010104404008801004040201010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101010101, value: 0x0

Caller: [CREATOR], function: getEntireSystemColl(address), txdata: 0x9e86d0c40101010101010101010101010101010201010101010102010101018001010101, value: 0x0











