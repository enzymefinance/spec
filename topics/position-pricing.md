# Position Pricing

Enzyme funds and their shares (along with certain policies and fees) rely on trustlessly calculating the values of [positions](fund-holdings.md#holdings) held. These values come from sources specific to each position that carry their own assumptions, idiosyncrasies, and risks. Fund owners and asset managers should be aware of which assets are appropriate for their fund setup.

## Pricing sources

Position value is derived from two sources:

1. External feeds registered on the [`ValueInterpreter`](../architecture/release.md#valueinterpreter)  contract
2. Internal feeds of individual [external positions](external-positions.md)

All Enzyme funds use the same `ValueInterpreter` , which is maintained by the Enzyme Technical Committee.

**Asset** **positions** (e.g., WETH, MLN, many LP tokens) are priced directly by `ValueInterpreter` only.

**External positions** (e.g., CDPs, non-fungible LP) contain an internal price feed, which reports its holdings in terms of virtual managed (positive value) and debt (negative value) asset positions. Generally - e.g., when calculating a fund's share price, these managed and debt assets are then aggregated into a target quote asset via `ValueInterpreter`.

See the [price feeds list](../external-interactions/price-feed-sources.md) section for the specific feeds used in `ValueInterpreter`, and the [external positions list](../external-interactions/external-positions.md) section for the available external position types, all of which have an internal pricing feed, as discussed above.

## Pricing risk

**Fund owners and asset managers must be aware of the pricing mechanism assumptions and vulnerabilities involved in the assets they hold, especially if their investors are unknown/untrusted entities.** This is because a fund over or under-prices its shares to the extent that Enzyme's understanding of the value of its holdings deviates from what can actually be acquired by trading or unwinding those positions. (see "[Known Risks & Mitigations](known-risks-and-mitigations.md)").

For asset positions, fund owners and asset managers must assess whether the feeds used by `ValueInterpreter` are safe for their fund setup.

For external positions, they must assess whether the internal price feeds are safe for the fund setup, in addition to whether the holdings of the external position result in virtual managed and debt asset positions that are themselves safe to price (by the same logic as standard asset positions).

For the most part, price feeds added to the `ValueInterpreter` are considered generally safe for use, though most carry risks of different natures (e.g., front-running, freshness, data sources quality, protocol risk, etc).

### Example: Wrapped or synthetic assets using a Chainlink price

With many wrapped assets (e.g., wBTC, stETH) and potentially synthetic assets (none currently), the protocol assumes that the asset (e.g., wBTC) maintains a 1:1 price with its underlying (e.g., BTC).

In the case of wrapped assets, the underlyings are held in custody by a third party (whether an EOA or contract). If access to the assets is lost by the custodying entity (e.g., contract vulnerability or private key compromise), the protocol will continue to treat the wrapper as 1:1 with its underlying, even though its real value would be between 1 and 0.

The same would be true of a collateralized synthetic asset that became under collateralized.

\[A full list of assets where a 1:1 assumption is used will soon be available in documentation and/or the Enzyme app]

### Example: Assets that rely on external protocol assumptions

For example, the [`CurvePriceFeed`](../external-interactions/price-feed-sources.md#curvepricefeed) that is used for pricing staked and unstaked Curve pool tokens would become unstable should any of the assets in the pool lose its 1:1 peg with the pool invariant (which would lead to a "bank run" of sorts, imbalancing the Curve pool).

### Example: Assets that rely on external protocol security

For example, the [YearnVaultV2PriceFeed](../external-interactions/price-feed-sources.md#yearnvaultv2pricefeed) that is used for pricing yVault tokens relies on its yVault contract correctly reporting its value in a way that cannot be manipulated by price oracle manipulation attacks.

### Caution: Riskier price feeds

The Enzyme Technical Committee routinely adds (and updates) price feeds to `ValueInterpreter` . Most of these feeds hold to some basic standards, such as:

* Uses Chainlink and Redstone aggregators (no internal assessment is made into freshness or data quality of specific aggregators)
* Reports fresh prices (or prices that have insignificant deviations between updates)
* Interacts only with well-audited external protocols
* Interacts with external protocol logic that only trusted entities can update&#x20;
* Has received a full audit or rigorous QA by an auditing firm

In order to facilitate a larger asset universe for funds to use, it is sometimes necessary for the Enzyme Technical Council to add price feeds that do not hold to a general standard of correctness or security, e.g.,

* Weighted-averages (or otherwise lagging prices)
* Discontinuous oracle updates
* Nacient / unaudited protocols
* Pegged prices (e.g., LRTs)

#### Mitigation: List of assets that use riskier price feeds

To assist in identifying assets that use price feeds with risks that exceed what is generally acceptable for default fund setups, the Enzyme Technical Committee maintains a list of riskier price feeds in its `AddressListRegistry`, with the following `listId`:

* Arbitrum: `16`
* Ethereum: `650`
* Polygon: `1383`

This address list can be referenced by asset managers (to avoid acquiring them) and/or within [policies](../architecture/release.md#policymanager) (to enforce not acquiring them).&#x20;

e.g., `DisallowedAdapterIncomingAssetsPolicy` can be used to prevent asset managers from acquiring such asset positions via properly-constructed adapters (meaning that they correctly report the incoming assets of the interaction).&#x20;
