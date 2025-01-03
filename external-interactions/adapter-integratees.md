# Adapters

In order to exchange some of a fund's assets for other assets, an adapter generally integrates with one or more "integratees," i.e., endpoints at which to interact with a defi protocol such as Uniswap, Compound, ParaSawp, etc.

## AaveV2Adapter

Integrates with Aave v2 lending via aTokens.&#x20;

Functions and considerations:

* `lend()` - None
* `redeem()` - None

## AaveV3Adapter

Integrates with Aave v3 lending via aTokens.

Functions and considerations:

* `lend()` - None
* `redeem()` - None

## BalancerV2LiquidityAdapter

Integrates with Balancer v2 to swap, and to lend/redeem/stake/unstake Balancer Pool Tokens (BPTs).

Functions and considerations:

* `lend()` - None
* `lendAndStake()` - None
* `redeem()` - None
* `takeOrder()` - Uses Balancer Vault's `batchSwap()`. Use for LPing with nested composable stable pools.
* `unstake()` - None
* `unstakeAndRedeem()` - None

## CompoundAdapter

Integrates with Compound Finance's v2 cTokens. Each cToken is its own integratee.

Functions and considerations:

* `lend()` - fund receives `cToken` , which triggers the `VaultProxy` to start accumulating `COMP` based on the amount lent.
* `redeem()` - None
* `claimRewards()` - None

Note that `COMP` is also claimable natively on Compound on behalf of the fund (by any user).

## CompoundV3Adapter

Integrates with Compound Finance's v3 cTokens. Each cToken is its own integratee.

Functions and considerations:

* `lend()` - fund receives `cToken` , which triggers the `VaultProxy` to start accumulating the reward token, if provisioned by the pool for the lent asset
* `redeem()` - None
* `claimRewards()` - None

Note that `COMP` is also claimable natively on Compound on behalf of the fund (by any user).

## CurveLiquidityAdapter

Integrates with Curve pools that adhere to Curve's [pool templates](https://github.com/curvefi/curve-contract/tree/master/contracts/pool-templates) to allow liquidity provision-related actions, including staking to tokenized liquidity gauges (i.e., `LiquidityGaugeV2` and later).

Functions and considerations:

* `claimRewards()` - none
* `lend()` - none
* `redeem()` - redemption can be made for either an equal balance of underlying pool tokens (relative to pool proportions), or for a single asset in the pool
* `stake()` - fund starts accruing $CRV and pool rewards (if applicable) after action
* `unstake()` - none
* `lendAndStake()` - fund starts accruing $CRV and pool rewards (if applicable) after action
* `unstakeAndRedeem()` - redemption can be made for either an equal balance of underlying pool tokens (relative to pool proportions), or for a single asset in the pool

Claiming accrued rewards can also be accomplished outside of the adapter through two mechanisms:

* $CRV - must be claimed either through a call from the `VaultProxy` or by an account nominated via a call from the `VaultProxy` . We register all of these as approved vault calls.
* pool rewards - can be claimed by any party at any time on behalf of the `VaultProxy`

Note on sidechains/L2s: earned $CRV is not paid via a `Minter` , but rather via the pool rewards mechanism described above.

## ERC4626Adapter

Integrates with any tokenized vault that implements the ERC4626 standard, unless there is some non-standard behavior (e.g., redemption queue, special rewards claiming, etc).

Functions and considerations:

* `lend()` : calls `ERC4626.deposit()`
* `redeem()`: calls `ERC4626.redeem()`
* no further options for interacting are currently supported

## OneInchV5Adapter

Integrates with 1Inch (v5) swaps.

Functions and considerations:

* `takeMultipleOrders()` - None
* `takeOrder()` - None

## ParaSwapV5Adapter

Integrates with ParaSwap (v5) via the `AugustusSwapper`. Incorporates asset approvals via the `TokenTransferProxy`.

Functions and considerations:

* `takeMultipleOrders()` - None
* `takeOrder()` - None

## UniswapV2ExchangeAdapter

Integrates with UniswapV2 for trading.

Functions and considerations:

* `takeOrder()` - None

## UniswapV2LiquidityAdapter

Integrates with UniswapV2 for liquidity provision and redemption.

Functions and considerations:

* `lend()` - None
* `redeem()` - None

## UniswapV3Adapter

Integrates with UniswapV3 for trading (not LP)

Functions and considerations:

* `takeOrder()` - None

## YearnVaultV2Adapter

Integrates with Yearn vault v2 instances

Functions and considerations:

* `lend()` - none
* `redeem()` - none

Note that while all Yearn vault v2 instances adhere to the same interface, each individual instance uses one of many particular versioned implementations (see `v2.registry.ychad.eth`). Only the adapter interactions with expected behaviors of interface functions can be realistically audited.

## ZeroExV4Adapter

Integrates with the 0x Protocol v4.

This adapter limits orders to a list of makers, defined on the adapter. There can be multiple deployments of this adapter to facilitate different lists of allowed makers (or any maker).

Functions and considerations:

* `takeOrder()` -  none
