# [GAS-1] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each read (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

Saves 100 gas per instance

yieldingPercentageDrift:

https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L264
https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L269