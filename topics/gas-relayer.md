# Gas Relayer

This release supports relayed transactions via Open GSN.

Fund managers can leverage this support to pay for gas costs of allowed transactions with the `VaultProxy` 's WETH balance.

From a high level, this works by deploying a GSN-compliant "paymaster" contract per-fund that is allowed to withdraw WETH from its associated `VaultProxy` to top up a deposit balance for that fund.

## Architecture

The gas relayer architecture is specific to this release (i.e., it is not "persistent" between releases).

The main contracts involved are:

* `GasRelayPaymasterLib` - the canonical library contract for all "paymaster" instances, providing the logic for interacting with GSN contracts, maintaining a healthy WETH deposit, and defining the rules for calls that can be relayed
* `GasRelayPaymasterFactory` - deploys new "paymaster" (`BeaconProxy`) instances, and is the reference for the current beacon library (i.e., `GasRelayPaymasterLib`)
* `GasRelayRecipientMixin` - shared logic that is inherited by all gateway contracts for relayable transactions

## Usage

**To use gas relaying:**

1. There must be enough WETH in the `VaultProxy` to cover the deposit amount specified by the current `GasRelayPaymasterLib` .&#x20;
2. The fund owner calls `deployGasRelayPaymaster()` on the `ComptrollerProxy`
3. The `ComptrollerProxy` deploys a new "paymaster" ( `BeaconProxy` instance) via the `GasRelayPaymasterFactory` and deposits WETH into the newly-deployed paymaster.
4. The fund should maintain enough WETH balance in the `VaultProxy` to top up the deposit before it runs out.

**To turn off the gas relayer**, the fund owner can call `shutdownGasRelayPaymaster()` on the `ComptrollerProxy`, which withdraws the WETH deposit back to the `VaultProxy`.

**When a fund migrates to a new release**, the fund owner can call `withdrawBalance()` on the "paymaster" to withdraw the WETH deposit back to the `VaultProxy`.

## Allowed calls

Any account with a permissioned role (owner, migrator, or asset manager) on the fund can use the internal gas relayer architecture.

All calls related to fund management that pass through the following contracts are allowed:

* `ComptrollerProxy`
* `VaultProxy`
* `PolicyManager`&#x20;
* `FundDeployer` (reconfiguration functions only; migration functions are not possible as they are called on a different release than the paymaster)

Note: anybody can use an external "paymaster" to pay for relayed transactions, and there is intentionally support included to enable deposit and redeeming shares functions, while not allowing those particular calls in the protocol's "paymaster".

