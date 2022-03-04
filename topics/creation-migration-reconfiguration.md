# Fund Lifecycle

Globally, new funds can be created on the latest release according to the `Dispatcher`. Once created, funds can be migrated to the newest release. The global logic for these actions is located in the `Dispatcher` and additional release-level logic is located in the `FundDeployer`.

The `FundDeployer` operates as the top-level release contract, governing:

* **creation**: how new funds are created on this release
* **migration**: how funds on previous releases are upgraded to this release
* **reconfiguration**: how funds on this release can change all release-level configuration (core settings, policies, and fees)

A new `ComptrollerProxy` is deployed and attached to the `VaultProxy` as its `accessor` via each of these actions. It stores core config options, and the config relative to extensions and their plugins are stored in those particular contracts by reference to the `ComptrollerProxy`. The `ComptrollerProxy` thus serves as the primary configuration object of a fund.

See "Architecture: Release" for more detail on these contracts.

## Creation

In order to create a new fund, CallerA (any account) can call `FundDeployer.createNewFund()` and all components and configuration of the fund are created atomically. The steps taken by this function are:

1. The `FundDeployer` deploys a new `ComptrollerProxy` instance, which sets the caller-provided core and extension config.
2. `FundDeployer` calls to the `Dispatcher` to deploy a new `VaultProxy` with the CallerA-provided `fundOwner` and `fundName` (note that `fundOwner` does not need to be CallerA), along with the release's `VaultLib` and the newly-created `ComptrollerProxy` that will become the `VaultProxy` 's `accessor`.
3. The `FundDeployer` sets the newly-deployed `VaultProxy` on the `ComptrollerProxy` and calls`ComptrollerProxy.activate()` to perform the final setup logic and give extensions a final chance to validate and update state.
4. The fund is now live on this release.

## Migration

### **Migration to this release from a previous release**

1. &#x20;MigratorA calls `FundDeployer.createMigrationRequest()` , which deploys a new `ComptrollerProxy` instance, setting the caller-provided core and extension config, and the `VaultProxy` that will be migrated.
2. MigratorA waits for the `migrationTimelock` defined on the `Dispatcher` to pass.
3. MigratorA calls `FundDeployer.executeMigration()` , which calls up to `Dispatcher.executeMigration()`
4. The `Dispatcher` updates the `VaultProxy` 's `VaultLib` and assigns the newly-created `ComptrollerProxy` as its `accessor` .
5. The `FundDeployer` calls `ComptrollerProxy.activate()` to give extensions a final chance to validate and update state.
6. The fund is now live on this release.

### Migration from this release to a new release

The `Dispatcher` invokes hooks at each stage of the migration that call down to the outbound `FundDeployer` , giving this release the chance to execute arbitrary code, reacting to the migration.

This release implements only the `invokeMigrationOutHook` that runs immediately prior to executing the migration. It:

* pays the due protocol fee
* pays out any fee shares outstanding
* calls `selfdestruct()` on the `ComptrollerProxy`&#x20;

## Reconfiguration

A reconfiguration is very similar to a migration (on both a high and low level) in that a new `ComptrollerProxy` is created to replace the old `ComptrollerProxy` , within the same release. It can even be called an "intra-release migration."

The difference is mostly that rather than passing calls up to an authoritative `Dispatcher`, the `FundDeployer` itself stores a `reconfigurationRequest` and defines a local`reconfigurationTimelock` .

1. &#x20;MigratorA calls `FundDeployer.createReconfigurationRequest()` , which deploys a new `ComptrollerProxy` instance, setting the caller-provided core and extension config, and the `VaultProxy` that will be moved.
2. MigratorA waits for the `reconfigurationTimelock` to pass.
3. MigratorA calls `FundDeployer.executeReconfiguration()` , which assigns the newly-created `ComptrollerProxy` as its `accessor` .
4. The `FundDeployer` deactivates the old `ComptrollerProxy` as described in "Migration from this release to a new release".
5. The `FundDeployer` calls `ComptrollerProxy.activate()` to give extensions a final chance to validate and update state.
6. The fund now has a new `ComptrollerProxy` with new configuration live on this release.





