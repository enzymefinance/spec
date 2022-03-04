# Asset pricing

Enzyme vaults and their shares (along with certain policies and fees) rely on trustlessly calculating the value of assets held.

This necessity creates a bounded "asset universe" for the protocol, where an asset must have a reasonable, reliable on-chain pricing mechanism, and then must be added to the protocol by the Enzyme Council.

Generally, if an asset has a Chainlink feed available, its price is directly queried from there (via the [`ChainlinkPriceFeed`](../external-interactions/price-feed-sources.md#chainlinkpricefeedmixin)).

Otherwise, a bespoke Enzyme price feed must be created in order to derive a pricing mechanism for the asset class. See the "Price Feeds" section for the available Enzyme feeds.

## Pricing risk

**Fund owners and asset managers must be aware of the pricing mechanism assumptions and vulnerabilities involved in the assets they hold, if their investors are unknown/untrusted entities.** This is because a fund is potentially exposed to share price arbitrage to the extent that Enzyme's understanding of the value of its holdings deviates in value from what can actually be acquired by trading or redeeming those assets. (see "[Known Risks & Mitigations](known-risks-and-mitigations.md)").

### Example: Wrapped or synthetic assets using a Chainlink price

With many wrapped assets (e.g., wBTC, cxDOGE, and Lido stETH) and potentially synthetic assets (none currently), the protocol assumes that the asset (e.g., wBTC) maintains a 1:1 price with its underlying (e.g., BTC).

In the case of wrapped assets, the underlyings are held in custody by a third party (whether an EOA or contract). If access to the assets is lost by the custodying entity (e.g., contract vulnerability or private key compromise), the protocol will continue to treat the wrapper as 1:1 with its underlying, even though its real value would be between 1 and 0.

The same would be true of a collateralized synthetic asset that became under collateralized.

\[A full list of assets where a 1:1 assumption is used will soon be available in documentation and/or the Enzyme app]

### Example: Assets that rely on external protocol assumptions

For example, the [`CurvePriceFeed`](../external-interactions/price-feed-sources.md#curvepricefeed) that is used for pricing staked and unstaked Curve pool tokens would become unstable should any of the assets in the pool lose its 1:1 peg with the pool invariant (which would lead to a "bank run" of sorts, imbalancing the Curve pool).

### Example: Assets that rely on external protocol security

For example, the [YearnVaultV2PriceFeed](../external-interactions/price-feed-sources.md#yearnvaultv2pricefeed) that is used for pricing yVault tokens relies on its yVault contract correctly reporting its value in a way that cannot be manipulated by price oracle manipulation attacks.
