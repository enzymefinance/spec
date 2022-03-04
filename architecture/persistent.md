# Persistent

## Core

### Philosophy

Enzyme has upgradable vaults, structured to give both managers and investors the opportunity to opt-in (managers) or opt-out (investors) of major updates.

For more on the philosophy and challenges of upgradability within the Enzyme Protocol, refer to this article: [https://medium.com/enzymefinance/fund-in-the-shell-e82c46a0a0fa](https://medium.com/enzymefinance/fund-in-the-shell-e82c46a0a0fa)

### Approach

Essential state lives in a `VaultProxy` , which is moved between releases by upgrading its `VaultLib` and permissioned `accessor` via a global `Dispatcher` contract.

The essential state for a fund is:

* holdings
* shares
* roles

### VaultProxy

The "essential state" described above lives in per-fund `VaultProxy` contract instances, which are upgradable contracts following the EIP-1822 proxy pattern.

The `VaultProxy` specifies a `VaultLib` as its target logic, and these logic contracts are deployed per release, and swapped out during a migration.

A `VaultLibBaseCore` contract defines the absolutely essential state and logic for every VaultProxy. This includes:

* a standard ERC20 implementation called `SharesTokenBase`&#x20;
* the functions required of a `IProxiableVault` called by the `Dispatcher`&#x20;
* core access control roles: `owner`, `accessor`, `creator`, and `migrator`&#x20;

The `owner` is the fund's owner.

The `migrator` is an optional role to allow a non-owner to migrate the fund.

The `creator` is the `Dispatcher` contract, which is allowed to update the `accessor` and `vaultLib` values.

The `accessor` is the primary account that can make state-changing calls to the `VaultProxy` . In practice, this is the release-level contract that interacts with a vault's assets, updates shares balances, etc.

This extremely abstract interface - in which a `VaultProxy` needs no knowledge about a release other than which caller can write state - allows for nearly limitless possibilities for release-level architecture.

### Dispatcher

&#x20;An overarching, non-upgradable `Dispatcher` contract is charged with:

* deploying new instances of `VaultProxy`&#x20;
* migrating a `VaultProxy` from an old release to the current release
* maintaining global state such as the current release, the global owner (i.e., the Enzyme Council) and the default `symbol` value for fund shares tokens

The `Dispatcher` stores the `currentFundDeployer` (a generic reference to the latest release's contract that is responsible for deploying and migrating funds), and only a `msg.sender` with that value is allowed to call functions to deploy or migrate a `VaultProxy` .&#x20;

This release-level `FundDeployer` can optionally implement migration hooks provided by `IMigrationHookHandler`, which give the release an opportunity to run arbitrary logic as the `Dispatcher` invokes those hooks at every step of the migration process.

As was the case described above with the `VaultLibBaseCore` , this abstracted notion of a `FundDeployer` - in which the `Dispatcher` only cares about its identity for access and for optional callbacks - is totally unrestrictive to the shape of the release-level protocol.

\---

Release-level contracts, then, are mostly arbitrary from the standpoint of these persistent contracts, offering maximum flexibility for future iterations and changes.

## Other

In addition to the core persistent architecture that endures across all releases, there are also persistent components whose lifetimes can span one or many releases.

Used in this release:

### **Protocol Fee Reserve**

The `ProtocolFeeReserveProxy` serves as a repository that stores collected fees, with upgradable logic for what can be done with the collected assets.

### External Position Factory

Deploys new `ExternalPositionProxy` instances, which are upgradable proxy contracts used to manage positions that live outside of the vault, e.g., CDPs.
