# Release

## Core

### FundDeployer

The `FundDeployer` can be considered the top-level contract of the release. It serves two roles.

Its primary purpose is the gateway to creating, migrating, and reconfiguring funds.

This is the contract that the `Dispatcher` considers as the `currentFundDeployer`, thus allowing it to deploy and migrate `VaultProxy` instances.

It also handles intra-release migration (completely changing fund config by swapping out a new ComptrollerProxy, referred to as "reconfiguration"), mimicking the `Dispatcher` 's paradigm for signaling and executing timelocked "reconfiguration requests."

The `FundDeployer` deploys configuration contracts ( `ComptrollerProxy` ) per-fund that are then attached to `VaultProxy` instances (more in the next section).

The `FundDeployer` is also used as a release-wide registry for limiting allowed values for permissioned calls in a uniform way for all funds.

There is 1 shared `FundDeployer` for the release.

### ComptrollerProxy

A `ComptrollerProxy` is deployed per-fund, and it is the canonical contract for interacting with a fund in this release. It stores core release-level configuration and is attached to a `VaultProxy` via the latter's `accessor` role.

All state-changing calls to the `VaultProxy` related to the fund's holdings and shares balances must thus pass through the `ComptrollerProxy`, making it a critically important bottleneck of access control.

The storage and logic of the `ComptrollerProxy` are defined by the `ComptrollerLib` and its associated libraries. It is not upgradable, but a fund can be "reconfigured" by deploying a new `ComptrollerProxy` instance to replace the previous one, via the `FundDeployer`.

### VaultLib

The `VaultLib` contract contains the storage layout, event signatures, and logic for `VaultProxy` instances that are attached to this release.

There is 1 shared `VaultLib` for the release.

## Extensions

Extensions extend the logic of the core contracts by adding additional kinds of functionality.

They are semi-trusted, in that they are selectively granted access to state-changing calls to `VaultProxy` instances.

In order to make such a state-changing call, two conditions must be met:

1. The Extension function must have been called by a `ComptrollerProxy` via a function with the `allowsPermissionedVaultAction` modifier, which opens the calling `ComptrollerProxy` to `VaultProxy` state changes.
2. The state-changing call must pass back through the `ComptrollerProxy`, and is delegated to the `PermissionedVaultActionLib` to determine whether the calling Extension is allowed to perform such an action.

This paradigm assures that an Extension can only perform a state-changing action to a `VaultProxy` if it was called by that `VaultProxy`'s corresponding `ComptrollerProxy` and if the Extension is permitted to make such a change at all.

Each extension manages a particular type of plugin:

* `IntegrationManager` - adapters
* `PolicyManager` - policies
* `FeeManager` - fees
* `ExternalPositionManager` - external positions

Plugins do not have authority to act on a vault's state, unless explicitly granted via the core system. e.g., a `SynthetixAdapter` can trade on Synthetix on behalf of a vault by enabling that action via a `ComptrollerLib.vaultCallOnContract()`

In this release, there are four Extensions. All funds share one contract per Extension.

### PolicyManager

The `PolicyManager` allows a fund owner to set up and manage a stack of "policies," which are used to perform bespoke validations during various function calls.

The protocol invokes "policy hooks" during actions where it is deemed important to give the fund owner customizability of what should be allowed/disallowed according to their particular needs.

Policies define which hooks they implement. When a hook is reached, it loops over all policies that a fund has enabled that run on that particular hook and validate whether the particular call is allowed.

See `IPolicyManager.PolicyHook` for the available hooks.

Policies themselves can only fail or pass, so the `PolicyManager` has no need or access to state-changing vault actions.

All policies are addable during fund creation, migration, or reconfiguration.

A policy defines for itself whether or not it is updatable or removable.

A policy can be added at any time, unless it runs on a policy hook that restricts current investors (i.e., shares redemption and shares transfer).

### FeeManager

The `FeeManager` allows for "fees" to dictate the minting, burning, or transferal of fund shares, according to their internal logics.

Like policies, fees implement "fee hooks," which are invoked during particular actions.

Fees can either settle and payout immediately, or they can accrue upon settlement as "shares outstanding," and only unlock for payout when specific conditions (defined by the fee) are met.

Fees can only be added or removed during fund setup (creation / migration / reconfiguration).

### IntegrationManager

The `IntegrationManager` allows:

* exchanging a fund's assets for other assets via "adapters" to DeFi protocols (e.g., Uniswap, Kyber, Compound, Chai)
* tracking assets
* untracking assets&#x20;

Each of these actions contains a policy hook.

### ExternalPositionManager

The `ExternalPositionManager` allows creating and managing "external positions," proxy contracts that represent non-ERC20 holdings of the fund, e.g., a Compound CDP.

The `ExternalPositionManager` maintains a registry of "libs" and "parsers" per external position, which serves as the "beacon" in this release for the `ExternalPositionProxy` 's specific beacon proxy pattern.

Each of the available actions for interacting with an external position contains a policy hook.

See "External Positions" section.

## Plugins

Each of the Extensions above make use of plugins. The `IntegrationManager` uses "adapters", the `PolicyManager` uses "policies", and the `FeeManager` uses "fees", and the `ExternalPositionManager` uses "external positions."

Arbitrary (third party) plugins are allowed for fees, policies, and integrations. Fund managers can decide whether or not to use third party plugins, and investors will be able to determine if fund configurations are safe for their trust threshold. A setup that only uses official plugins would entail:

* No arbitrary policies that restrict investor actions (can only be added at fund setup)
* No arbitrary fees (can only be added at fund setup)
* No arbitrary adapters (unremovable policy will be provided with a registry of official, Council-verified adapters)

## Infrastructure

In addition to "core" and "extension" release-level contracts, there is a broad category of "infrastructure" contracts, which are misc dependencies of the release-level protocol. Unlike extensions, they do not receive any permissions to alter fund state.

### AssetFinalityResolver

The `AssetFinalityResolver` settles Synths in an efficient manner for operations that depend on accurate Synth balances.

### **Gas Relayer**

This release allows for funds to optionally use Gas Station Network relayers to pay for calls to specific contracts and functions.

See "Gas Relayer" section.

### **ProtocolFeeTracker**

The `ProtocolFeeTracker` tracks protocol fee payments and is queried to determine the amount of shares that a fund should mint to the `ProtocolFeeReserve` to bring its fees up-to-date.

See "Protocol Fees" section.

### ValueInterpreter

The `ValueInterpreter` is the single point of aggregation of various "price feeds" (an additional type of "plugin" that is only managed by the Enzyme Technical Committee) that are used to calculate the value of one or many input asset amounts in terms of an output asset.

There are two categories of assets in this release:

* "primitives" - assets for which we have rates via Chainlink-like aggregators that are either quoted in ETH or USD
* "derivatives" - assets for which we have rates via custom price feeds that are quoted in one or multiple underlying assets (e.g., Compound cTokens, Uniswap v2 pool tokens, etc)

The `ValueInterpreter` determines whether an asset is a primitive or derivative, and executes logic to use corresponding price feeds to determine the value in the output asset.

## Interfaces

All interfaces to external contracts are contained in the `release/interfaces/` directory.

Interfaces for internal contracts (e.g., `IFundDeployer` ) are kept beside the contracts to which they refer. These are narrow interfaces that only contain the functions required by other non-plugin, release-level contracts (i.e., those in the "core" and "extensions" sections above).
