# Known Risks & Mitigations

Fund owners are fully-trusted.

Migrators (assigned by the fund owner) are also fully-trusted since they can arbitrarily change all fund configuration.

Both of these roles can drain funds by default.

Beyond those, there are primarily two risk categories (not including griefing) in terms of the behavior of various actors:

* Opportunistic investors
* Opportunistic asset managers

Different fund setups will have different levels of trust for these parties.

E.g., a DAO treasury might only have a single investor who is ostensibly the same entity as the owner. Or, they might delegate asset management to an EOA that should only be trusted to operate within specific parameters.

E.g., an individual fund owner might be a well-known party for whom reputation serves as a natural mitigation for investors. Or, they might be completely anonymous and untrusted.

In order to not sweepingly apply the strictest risk mitigations to all funds, the protocol is largely unopinionated about what constitutes a "safe" setup, but offers various configuration options and policies to craft bespoke setups that meet particular trust requirements.

## Opportunistic investors (arbitrage)

Investors can arbitrage temporarily mispriced shares or mispriced assets held by a fund by depositing and/or redeeming from a fund under opportune conditions.

_General mitigations for the below opportunities:_

* A `sharesActionTimelock` config option defines the seconds that must pass between a user's most recent deposit and their next transfer or redemption. Though 1 second is enough to prevent flash and sandwich exploits, the longer the `sharesActionTimelock`, the less of a guarantee that an arbitrage opportunity will remain open at the allowed time of redemption.
* Fees can run when depositing or redeeming shares that (conditionally) deduct an amount of shares, thus increasing their effective share price. E.g., for mitigating deposit arbitrage, funds that wish to use an additional arbitrage protection can use the `EntranceRateBurnFee` , which burns a % of the shares minted during deposit.

### **Opportunity: mispriced shares due to untracked value during deposit**

There may be value that "belongs" to a fund that is not included in its share price, i.e., "untracked assets" in the `VaultProxy` (e.g., an airdrop) and unclaimed "external rewards" (e.g., accrued `COMP` rewards) (see "Holdings and Shares").

_Extra mitigations_:

* New assets acquired to the VaultProxy via the protocol are automatically added as tracked assets
* Asset managers should track any untracked assets (e.g., via airdrops or publically-callable rewards claiming functions) as quickly as possible
* In the case of accruing external rewards (e.g., `COMP`), asset managers are advised to track any reward token that they expect to earn ASAP, e.g., track `COMP` as soon as you lend or borrow via Compound for the first time.

### **Opportunity: mispriced shares due to on-chain prices during deposit**

Due to the exclusive use of on-chain asset prices, there may be occasions where there is a deviation between on-chain and off-chain values of assets, and thus also share price.

Though the price feeds used in the protocol should be considered manipulation resistant, there are still occasions where prices are updated via transactions and can therefore be frontrun (e.g., Chainlink aggregator price updates). Furthermore, price updates might only occur when deviation thresholds are crossed (i.e., Chainlink aggregators), and even tight thresholds of <1% in a large enough fund could result in significant arbitrage opportunities.

If share price is "too low" (i.e., the total value of assets according to on-chain, internally-used prices is lower than the total value according to canonical prices), then new investors can deposit and essentially receive a discount.

### **Opportunity: mispriced assets due to on-chain prices during specific asset redemption**

Similarly, to the extent that internally-used asset values deviate from their canonical values, the `redeemSharesForSpecificAssets()` redemption option can be arbitraged, by withdrawing one or more assets that are priced "too low" relative to other assets in the fund.

Though the use of a `sharesActionTimelock` prevents a user who is yet to hold shares from exercising this arbitrage opportunity, current investors for whom the timelock has expired can redeem at any time.

_Extra mitigations:_

* An exit fee charged only on `FeeHook.RedeemSharesForSpecificAssets` (i.e., not on in-kind redemption) that burns a % of shares being redeemed.

## Opportunistic Asset Managers

Asset managers can outright steal value from a fund through bad configurations or bad actions with holdings.

A key concept with asset manager risk mitigation in the protocol is that it is extremely difficult to stop them from stealing value altogether without overly restricting their actions. The goal is to slow down the stealing of value in such a manner as to give fund owners and investors sufficient time to notice (or be notified) and remove the asset manager (owner) or exit a fund (investor) as necessary.

### **Opportunity: drain a fund via adapters**

It is possible via some adapters to trade in an opportunistic manner that results in value leaking from the fund into external accounts (i.e., to an asset manager).

For example, a multi-hop trade on Uniswap can be routed via an arbitrary intermediary pool in which the asset manager is the sole LP provider. Similar exploits would be possible through various routes on ParaSwap.

_Mitigation:_ Use a policy that limits share price value loss allowed over a given period, e.g., 5% over 24 hours

### **Opportunity: untrack assets**

An asset manager can untrack any tracked assets in the fund (other than the denomination asset), effectively exposing a shares arbitrage opportunity.

_Mitigations:_

* use a policy that limits removing tracked assets to negligible amounts



