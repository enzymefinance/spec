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

### **Migrator**

Each fund can have a single migrator, who can call any migration or reconfiguration action on the `FundDeployer`.

Only the owner can set or unset a migrator.

### **Asset Manager**

Each fund can have many asset managers, who can call any action on the `IntegrationManager` or `ExternalPositionManager` , and also buyback protocol fee shares. Allowed calls to the `IntegrationManager` and `ExternalPositionManager` can be further narrowed per-asset manager via policies.

Only the owner can add or remove asset managers.

## Investors

A fund can theoretically have unlimited investors, who get exposure to a fund's performance by buying, redeeming, or receiving a transfer of fund shares.

## Relationship between managers & Investors

Different funds have different needs and trust assumptions, and are thus as unrestrictive as possible by default, with configuration options, policies, and fees available to craft bespoke levels of trustedness between managers and investors.

It is assumed that investors will pay special attention to review the configuration in place before investing in a particular fund.
