https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L627

Description: When the treasury address is updated, there is no event emitted to show that change of state

Remediation: Consider emitting an event for changes like this.
