# Access Control Handoff

## Dispatcher ownership

The owner of the `Dispatcher` is the canonical global admin for the protocol, persisting across releases (how the release implements that authority is up to the release).

The owner is already the Enzyme Council.

The Enzyme Council is able to transfer ownership (e.g., to a new multisig wallet, as has taken place) via a nomination-claim procedure.

## FundDeployer ownership

For this release, the owner of the `FundDeployer` is taken to be the admin of release-level protocol contracts.

The owner of `FundDeployer` is set dynamically:

* when `isLive` is `false`, the owner is the contract's deployer, i.e., Avantgarde Core
* when `isLive` is set to `true` (by Avantgarde Core), the contract then defers ownership to the owner of `Dispatcher`

This setup allows for easily setting up all contract configuration prior to freezing ownership for verification.

## Extensions and plugins ownership

Extensions (`FeeManager`, `PolicyManager`, `IntegrationManager`) and plugins (fees, policies, integration adapters) that require ownership for access control defer ownership to the current `FundDeployer` owner. This is accomplished by inheriting a `FundDeployerOwnerMixin` contract.

Thus when the owner of the `FundDeployer` becomes the ETC, so does the owner of all contracts that implement this mixin.

\---

These patterns of handing-off access control gives maximum flexibility for deployment and configuration, while assuring that the ETC will end up with complete admin privileges once the protocol is taken liven.
