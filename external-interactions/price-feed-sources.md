# Price Feeds

Just as adapters and external positions interact with external "integratees," so do price feeds interact with sources that provide them with the data they need to provide rates.

## AavePriceFeed

Prices Aave `aTokens` , which are pegged 1:1 with the value of their underlying, e.g., 1 `aAAVE` is equal to 1 `AAVE` . `aTokens` accrue interest via a formulaic rebasing mechanism.

Docs: [https://docs.aave.com/](https://docs.aave.com/)

Mainnet contracts:

`ProtocolDataProvider` : `0x057835Ad21a177dbdd3090bB1CAE03EaCF78Fc6d`&#x20;

Considerations: Always directly returns the input amount as the output amount (i.e., 1:1 pegging)

## ChainlinkPriceFeedMixin

Each rate pair (we use those quoted in either USD or ETH) is provided by a Chainlink "aggregator," and we interact with the proxy contracts for those aggregators.

Docs: [https://docs.chain.link/](https://docs.chain.link/)

Mainnet contracts: each aggregator

Considerations:

We do not check timestamps on every price lookup, rather we provide a function to allow quickly removing an aggregator that is considered to be stale. If an aggregator is removed, the corresponding primitive will cease to produce a price from the feed and will revert, causing any function that relies on that price to fail until it is re-instituted. This is the desired behavior.

## CompoundPriceFeed

Queries each Compound Token (cToken) directly for its rate.

Docs: [https://compound.finance/docs](https://compound.finance/docs)

Mainnet contracts: each cToken

Considerations:

We query the cached rate instead of the live rate for gas savings. cTokens rates change negligibly for long periods of time.

## ConvexCurveLpStakingWrapperPriceFeed

Converts an amount of a [`ConvexCurveLpStakingWrapper`](../topics/staking-wrappers.md#convexcurvelpstakingwrapper) token into its underlying Curve LP token (rate is always 1:1)

Mainnet contracts:

* each wrapper contract

## CurvePriceFeed

Provides prices for all Curve LP tokens, both staked and unstaked. Staking LP tokens to the pool's "liquidity gauge" (or to a liquidity gauge wrapper contract) returns an equal amount of tokens representing the stake in that liquidity gauge. i.e., 1 LP token is equivalent to 1 liquidity gauge token in value.

Remembering that deriving the underlying value for a derivative in Enzyme consists of returning underlying asset(s) and amount(s), there are two components for returning a value:

1. Underlying asset: each LP token (staked or unstaked) is arbitrarily assigned an "invariant proxy asset," a supported asset in the protocol that is chosen to be the proxy (i.e., representative) of the pool invariant (e.g., WETH in the case of the stETH pool).
2. Underlying amount: the amount of the invariant proxy asset is calculated via the pool's "virtual price" (provided natively in all Curve pools).

Considerations:

Since an arbitrary asset is chosen for the single underlying asset returned by the price feed (rather than considering all values and balances of the pool's underlyings), the live value of an LP token within Enzyme will always deviate from the redeemable value by some small percentage. This deviation is insignificant for a couple reasons:

* Assets that temporarily lose their peg will generally lose it in the downward direction, which would cause the redeemable value of an LP token to be less than its virtual price-derived Enzyme value. This condition is not susceptible to investor-side arbitrage in Enzyme funds, where the concern is preventing the purchase of discounted shares (whereas using the virtual price results in temporarily inflated share price). Further, as long as the loss of peg is assumed to be temporary, using the virtual price rather than this ephemeral imbalance protects the fund from discounted shares.
* In the more extreme case of permanent loss of peg, Curve only works if all pooled assets generally hold their peg to the invariant. If any one asset were to permanently lose its peg to the invariant, the redeemable value of the LP token itself would capitulate, as arbitragers drain the pool of all but the fallen asset.

## FiduPriceFeed

Provides a value for Goldfinch's FIDU token.

Docs: [https://docs.goldfinch.finance/goldfinch/](https://docs.goldfinch.finance/goldfinch/)

Mainnet contracts:

* FIDU token: `0x6a445E9F40e0b97c92d0b8a3366cEF1d67F700BF`

Considerations:

Goldfinch's withdrawal fee is included in the calculations.

## FusePriceFeed

Queries each Fuse Token (fToken) directly for its rate.

Docs: [https://docs.rari.capital/fuse/](https://docs.rari.capital/fuse/)

Mainnet contracts: each fToken

Considerations:

We query the cached rate instead of the live rate for gas savings. fTokens rates change negligibly for long periods of time.

## IdlePriceFeed

Provides a value for each `IdleToken` to its underlying asset.

Docs: [https://developers.idle.finance/](https://developers.idle.finance/)

Mainnet contracts: each `IdleToken`

Considerations: Does not take into account user-specific fees (i.e., relative to the vault) that are charged upon redeeming `IdleToken` for its underlying

## PoolTogetherV4PriceFeed

Prices PoolTogether `ptTokens` , which are pegged 1:1 with the value of their underlying, e.g., 1 `ptUSDC` is equal to 1 `USDC` .

Docs: [https://v4.docs.pooltogether.com/](https://v4.docs.pooltogether.com/)

Mainnet contracts:

* each `ptToken`

## RevertingPriceFeed

Immediately reverts upon any price lookup.

The purpose of the `RevertingPriceFeed` is to be able to keep an asset in our asset universe, while disabling any actions that rely on accurate GAV calculation. It is only to-be-used as a temporary stop gap to allow the continued functionality of integrations, at the cost of limited functionality in the core system.

Tracking an asset that uses the RevertingPriceFeed causes an action that depends on GAV to revert, affecting, for example:

* deposits
* fees that depend on GAV
* policies that depend on GAV

Note that reverting fees are skipped during shares redemption, which would affect a fund that has GAV-dependent fees that settle upon redemption (e.g., `PerformanceFee`) and that tracks any asset that uses the RevertingPriceFeed.

## UniswapV2PoolPriceFeed

Uses a special pool manipulation-resistant formula that takes into consideration the current underlying asset balances and pool token balance in the given Uniswap pool (`UniswapV2Pair`) along with a trusted rate between the two underlying tokens.

Docs: [https://docs.uniswap.org/](https://docs.uniswap.org/)

Mainnet contracts:

* `UniswapV2Factory`: `0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f`
* instances of `UniswapV2Pair`

See also the sample implementation we based this on: [https://github.com/Uniswap/uniswap-v2-periphery/blob/267ba44471f3357071a2fe2573fe4da42d5ad969/contracts/libraries/UniswapV2LiquidityMathLibrary.sol](https://github.com/Uniswap/uniswap-v2-periphery/blob/267ba44471f3357071a2fe2573fe4da42d5ad969/contracts/libraries/UniswapV2LiquidityMathLibrary.sol)

Considerations: Live prices



## YearnVaultV2PriceFeed

Provides a value for each `yVault` (Yearn vault v2) in terms of its underlying asset

Docs: [https://docs.yearn.finance/](https://docs.yearn.finance/)

Mainnet contracts:

* Registry/Deployer: `0x50c1a2eA0a861A967D9d0FFE2AE4012c2E053804`

Considerations:

* Live prices
* Note that while all Yearn vault v2 instances adhere to the same interface, each individual instance uses one of many particular versioned implementations (see registry contract above). Only the price feed interactions with expected behaviors of interface functions can be realistically audited.
