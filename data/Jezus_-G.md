#gas optimization
#Ethos-Vault/contracts/ReaperVaultERC4626.sol Line no 273

using ++it instead of it++ will save gas 


        if (x % y != 0) q++;
  


        if (x % y != 0) ++q;
  

