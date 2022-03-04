# External Positions

An "external position" is a type of fund holding introduced in v4 that:

* is not a fungible ERC20 token
* exists outside of the vault as a distinct contract (one per external position instance)
* reports its value to the fund in terms of its underlying holdings and liabilities
* cannot have value withdrawn during a shares redemption (investors in funds that use external positions should almost never redeem in-kind, as any value inside of external positions will effectively be forfeited)
* is a "beacon" proxy contract that uses the library dictated by the fund's current release (e.g., Sulu)

## Persistent architecture

An `ExternalPositionProxy` is deployed by a persistent `ExternalPositionFactory` , which serves as an enduring registry of external position "types" (e.g., Compound CDP) and all valid `ExternalPositionProxy` instances (i.e., those created via the factory).

The `ExternalPositionProxy` is deployed with a `VaultProxy` as its owner.

Upon each call, `ExternalPositionProxy` fetches its library via a `getExternalPositionType()` callback to its `VaultProxy`.

Protected / trusted calls to state-changing actions in the `ExternalPositionProxy` should pass through a protected `receiveCallFromVault()` , guaranteeing the call passed through the owning `VaultProxy` .

## Release architecture

The lifecycle of an external position is managed via `ExternalPositionManagerActions` on the `ExternalPositionManager` extension:

* `CreateExternalPosition` -- create (deploy) a new external position and activate it on the `VaultProxy`
* `CallOnExternalPosition` -- make allowed arbitrary calls to interact with the external position, and transfer assets to / from the `VaultProxy`
* `RemoveExternalPosition` -- deactivate an external position from the `VaultProxy`
* `ReactivateExternalPosition` -- reactivate an existing external position on the `VaultProxy`

Each of these actions has its own policy hook, to allow granular control of risk management.

The `ExternalPositionManager` stores two updatable reference contracts per external position type, which contain all logic specific to that type:

* `lib` is the target library of the beacon `ExternalPositionProxy`, and contains the business logic for its local accounting and interactions with external protocols
* `parser` is a contract used by the `ExternalPositionManager` to validate and parse call data for each specific action on its external position type

For example, VaultA has a Compound CDP external position already deployed. To add collateral to the CDP:

* an asset manager calls the `ExternalPositionManager` 's `CallOnExternalPosition` action via `ComptrollerProxy.callOnExtension()` with the desired payload to add 100 cDAI as collateral
* the `ExternalPositionManager` looks up its stored `CompoundDebtPositionParser` and forwards it the payload
* the `CompoundDebtPositionParser` validates the call, e.g., that cDAI is a valid cToken
* the `CompoundDebtPositionParser` formats the outgoing 100 cDAI into `assetsToTransfer` and `amountsToTransfer` arrays, which it returns to the `ExternalPositionManager`
* the `ExternalPositionManager` passes the payload along with the data for assets to transfer (100 cDAI) and to receive (none) to the `VaultProxy`
* the `VaultProxy` transfers 100 cDAI to the target `ExternalPositionProxy`
* the `VaultProxy` calls `ExternalPositionProxy.receiveCallFromVault()` , passing in the payload
* &#x20;`ExternalPositionProxy` calls back to the `VaultProxy` for the library to use for its type (Compound CDP), which it pulls from the `ExternalPositionManager`
* the `ExternalPositionProxy` uses the returned `CompoundDebtPositionLib` to parse and execute the `AddCollateralAssets` action, adding cDAI to its internal accounting of its managed assets and interacting with Compound to use cDAI as collateral as needed.

&#x20;
