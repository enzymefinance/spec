# End users

There are two categories of end users of the Enzyme Protocol: fund managers and investors.

## Fund managers

There are three primary fund management roles in this release, all of which are stored on the `VaultProxy` and thus will persist to subsequent releases (if those releases decide to use them):

* Owner
* Migrator
* Asset Manager

### Owner

Each fund has one owner, who can perform any administrative action on the fund.

Ownership can be changed via a pair of nominate-claim transactions.

Owners are considered fully-trusted.

### **Migrator**

Each fund can have a single migrator, who can call any migration or reconfiguration action on the `FundDeployer`.

Only the owner can set or unset a migrator.

### **Asset Manager**

Each fund can have many asset managers, who can call any action on the `IntegrationManager` or `ExternalPositionManager` , and also buyback protocol fee shares. Allowed calls to the `IntegrationManager` and `ExternalPositionManager` can be further narrowed per-asset manager via policies.

Only the owner can add or remove asset managers.

## Investors

A fund can theoretically have unlimited investors, who get exposure to a fund's performance by buying, redeeming, or receiving a transfer of fund shares.

## Relationship between owners, asset managers, and investors

Different funds have different needs and trust assumptions. The core contracts are left as unrestrictive as possible by default, with configuration options, policies, and peripheral tooling available to help craft bespoke levels of trust.

Fund owners are considered fully-trusted by all contracts created and configured by the Enzyme team. Any trust limitations must be done externally via peripheral contracts (e.g., timelocked owner contracts).

Fund owners delegate portfolio management actions to asset managers, whose available actions and action results can be restricted by fund owners. Ultimately, it is the responsibility of the fund owner to adequately restrict asset manager permissions to achieve their desired trust threshold.

Similarly, roles in peripheral contracts (e.g., executors of deposit and redemption queues) are assigned by the fund owner, who must assess whether the role is safe to assign to a given account.

It is assumed that investors will continuously assess their own trust of the fund owner and setup, as-needed.
