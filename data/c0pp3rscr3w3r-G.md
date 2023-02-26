### Gas optimization in Ethos-core/contracts/Migrations.sol

#### Location:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Migrations.sol#L9-L10

modifier restricted() {
    if (msg.sender == owner) _;
  }


***can be changed to***

modifier restricted() {
    if (msg.sender == owner) _;
    revert()
  }

to save gas