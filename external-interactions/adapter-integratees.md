# Adapters

In order to exchange some of a fund's assets for other assets, an adapter generally integrates with one or more "integratees," i.e., endpoints at which to interact with a defi protocol such as Uniswap, Compound, ParaSawp, etc.

## AaveV2Adapter

Integrates with Aave v2 lending via aTokens.

Docs: [https://docs.aave.com/](https://docs.aave.com/)

Mainnet contracts:

* `LendingPoolAddressesProvider`: `0xB53C1a33016B2DC2fF3653530bfF1848a515c8c5`&#x20;

Functions and considerations:

* `lend()` - None
* `redeem()` - None

## AaveV3Adapter

Integrates with Aave v3 lending via aTokens.

Docs: [https://docs.aave.com/](https://docs.aave.com/)

Mainnet contracts:

* `Pool`: `0x87870Bca3F3fD6335C3F4ce8392D69350B4fA4E2`&#x20;

Functions and considerations:

* `lend()` - None
* `redeem()` - None

## AuraBalancerV2LpStakingAdapter

Integrates with Aura to stake and unstake Balancer Pool Tokens (BPTs), via wrapped Aura staking tokens.

Can be used for combining Balancer LP/swap and Aura staking actions.

Docs: [https://docs.aura.finance/](https://docs.aura.finance/)

Mainnet contracts:

* `Vault` (Balancer): `0xBA12222222228d8Ba445958a75a0704d566BF2C8`

Functions and considerations:

* `lendAndStake()` - None
* `takeOrder()` - Uses Balancer Vault's `batchSwap()`. Use for LPing with nested composable stable pools.
* `unstake()` - None
* `unstakeAndRedeem()` - None

Above action implementations are the same as [#balancerv2liquidityadapter](adapter-integratees.md#balancerv2liquidityadapter "mention")

## BalancerV2LiquidityAdapter

Integrates with Balancer v2 to swap, and to lend/redeem/stake/unstake Balancer Pool Tokens (BPTs).

Docs: [https://docs.balancer.fi/](https://docs.balancer.fi/)

Mainnet contracts:

* `BalancerMinter`: `0x239e55F427D44C3cc793f49bFB507ebe76638a2b`
* `Vault`: `0xBA12222222228d8Ba445958a75a0704d566BF2C8`

Functions and considerations:

* `lend()` - None
* `lendAndStake()` - None
* `redeem()` - None
* `takeOrder()` - Uses Balancer Vault's `batchSwap()`. Use for LPing with nested composable stable pools.
* `unstake()` - None
* `unstakeAndRedeem()` - None

## CompoundAdapter

Integrates with Compound Finance's v2 cTokens. Each cToken is its own integratee.

Docs: [https://compound.finance/docs](https://compound.finance/docs)

Mainnet contracts:&#x20;

* all cTokens, other than those with underlyings not supported in the Enzyme asset universe

Functions and considerations:

* `lend()` - fund receives `cToken` , which triggers the `VaultProxy` to start accumulating `COMP` based on the amount lent.
* `redeem()` - None
* `claimRewards()` - None

Note that `COMP` is also claimable natively on Compound on behalf of the fund (by any user).

## CompoundV3Adapter

Integrates with Compound Finance's v3 cTokens. Each cToken is its own integratee.

Docs: [https://compound.finance/docs](https://compound.finance/docs)

Mainnet contracts:&#x20;

* all cTokens, other than those with underlyings not supported in the Enzyme asset universe
* `Rewards`: `0x1B0e765F6224C21223AeA2af16c1C46E38885a40`

Functions and considerations:

* `lend()` - fund receives `cToken` , which triggers the `VaultProxy` to start accumulating the reward token, if provisioned by the pool for the lent asset
* `redeem()` - None
* `claimRewards()` - None

Note that `COMP` is also claimable natively on Compound on behalf of the fund (by any user).

## ConvexCurveLpStakingAdapter

Integrates with [`ConvexCurveLpStakingWrapper`](../peripheral-features/staking-wrappers.md#convexcurvelpstakingwrapper) deployments to facilitate the staking of Curve LP tokens to Convex Finance.

Also provides convenience functions to LP on Curve and then stake on Convex in the same action, i.e., `lendAndStake()` and `lendAndRedeem()`. These actions use the same logic for LP'ing as the [`CurveLiquidityAdapter`](adapter-integratees.md#curveliquidityadapter-1) and thus have the same requirements and considerations.

Mainnet contracts:

* `AddressProvider` (Curve): `0x0000000022D53366457F9d5E68Ec105046FC4383`
* all deployed `ConvexCurveLpStakingWrapper` instances

Functions and considerations:

* `claimRewards()` - none
* `stake()` - fund starts accruing $CVX, $CRV and pool rewards (if applicable) after action
* `unstake()` - none
* `lendAndStake()` - fund starts accruing $CVX, $CRV and pool rewards (if applicable) after action
* `unstakeAndRedeem()` - redemption can be made for either an equal balance of underlying pool tokens (relative to pool proportions), or for a single asset in the pool

Claiming accrued rewards on behalf of any fund can also be accomplished outside of the adapter via the `ConvexCurveLpStakingWrapper` .

## CurveExchangeAdapter

Integrates with Curve's universal interface for swapping between any assets in a given pool.

Docs: [https://curve.readthedocs.io/](https://curve.readthedocs.io/)

Mainnet contracts:

* `AddressProvider`: `0x0000000022D53366457F9d5E68Ec105046FC4383`

Functions and considerations:

* `takeOrder()` : none

## CurveLiquidityAdapter

Integrates with Curve pools that adhere to Curve's [pool templates](https://github.com/curvefi/curve-contract/tree/master/contracts/pool-templates) to allow liquidity provision-related actions, including staking to tokenized liquidity gauges (i.e., `LiquidityGaugeV2` and later).

Docs: [https://curve.readthedocs.io/](https://curve.readthedocs.io/)

Mainnet contracts:

* `AddressProvider`: `0x0000000022D53366457F9d5E68Ec105046FC4383`
* `Minter` : `0xd061D61a4d941c39E5453435B6345Dc261C2fcE0`
* all LP tokens and liquidity gauge tokens in the asset universe

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

Docs: [https://ethereum.org/en/developers/docs/standards/tokens/erc-4626/](https://ethereum.org/en/developers/docs/standards/tokens/erc-4626/)

Functions and considerations:

* `lend()` : calls `ERC4626.deposit()`
* `redeem()`: calls `ERC4626.redeem()`
* no further options for interacting are currently supported

## IdleAdapter

Integrates with the Idle Finance's `IdleToken` contracts using the functions available in V4 of its protocol.

Docs: [https://developers.idle.finance/](https://developers.idle.finance/)

Mainnet contracts: each `IdleToken`

Functions and considerations:

* `approveAssets()` - can only be used for rewards tokens that are assets in the Enzyme asset universe
* `claimRewards()` - claimed rewards tokens are sent to the Vault, but are not reported as `incomingAssets`, and thus are not emitted in an event, not run through policy management, and are not added as tracked assets
* `lend()` - none
* `redeem()` - any call to redeem() will also claim all rewards tokens; these rewards tokens are sent to the vault, but are not reported as `incomingAssets`, and thus are not emitted in an event, not run through policy management, and are not added as tracked assets

Unclaimed rewards transfer to the recipient during ERC20 transfer calls, rather than auto-checkpointing as other protocols do.

## OneInchV5Adapter

Integrates with 1Inch (v5) swaps.

Docs: [https://docs.1inch.io/](https://docs.1inch.io/)

Mainnet contracts:

* `AggregationRouterV5`: `0x1111111254eeb25477b68fb85ed929f73a960582`

Functions and considerations:

* `takeMultipleOrders()` - None
* `takeOrder()` - None

## ParaSwapV5Adapter

Integrates with ParaSwap (v5) via the `AugustusSwapper`. Incorporates asset approvals via the `TokenTransferProxy`.

Docs: [https://doc.paraswap.network/](https://doc.paraswap.network/)

Mainnet contracts:

* `AugustusSwapper`: `0xDEF171Fe48CF0115B1d80b88dc8eAB59176FEe57`
* `TokenTransferProxy`: `0x216B4B4Ba9F3e719726886d34a177484278Bfcae`

Functions and considerations:

* `takeMultipleOrders()` - None
* `takeOrder()` - None

## PoolTogetherV4Adapter

Integrates with PoolTogether (v4).

Docs: [https://v4.docs.pooltogether.com/](https://v4.docs.pooltogether.com/)

Mainnet contracts:

* all ptTokens, other than those with underlyings not supported in the Enzyme asset universe

Functions and considerations:

* `claimRewards()` - none
* `lend()` - fund is entered in the PoolTogether drawings for the lent token, and must actively claim winnings. This must be monitored by managers, off-chain.
* `redeem()` - none

Notes:

* Claiming accrued winnings can also be accomplished via `claimRewards()` (required if the manager wants to use the gas relayer for the tx cost), or via the PoolTogether PrizeDistributor directly (open to any caller for any winner).
* When transferring ptTokens, the recipient's received amount is only entered into the lottery if they have previously specified a delegate (either themself or a third party). If the recipient has not yet specified a delegate, they must do so in a separate call via PoolTogether contracts. This should be taken into consideration by fund managers and investors, as investors that receive ptTokens from a shares redemption will need to choose a delegate to be entered into drawings, if they have not yet done so.

## SynthetixAdapter

\[Deprecated. Can currently only be used to purge already-held synths into sUSD]

Integrates with Synthetix via `SNX`.

Docs: [https://docs.synthetix.io/](https://docs.synthetix.io/)

Mainnet contracts:

* `SNX`: `0xC011a73ee8576Fb46F5E1c5751cA3B9Fe0af2a6F`

Functions and considerations:

* `takeOrder()` - None
* `redeem()` - None

## UniswapV2ExchangeAdapter

Integrates with UniswapV2 for trading.

Docs: [https://docs.uniswap.org/](https://docs.uniswap.org/)

Mainnet contracts:

* `UniswapV2Router2`: `0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D`

Functions and considerations:

* `takeOrder()` - None

## UniswapV2LiquidityAdapter

Integrates with UniswapV2 for liquidity provision and redemption.

Docs: [https://docs.uniswap.org/](https://docs.uniswap.org/)

Mainnet contracts:

* `UniswapV2Factory`: `0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f`
* `UniswapV2Router2`: `0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D`

Functions and considerations:

* `lend()` - None
* `redeem()` - None

## UniswapV3Adapter

Integrates with UniswapV3 for trading (not LP)

Docs: [https://docs.uniswap.org/](https://docs.uniswap.org/)

Mainnet contracts:

* `SwapRouter` : `0xE592427A0AEce92De3Edee1F18E0157C05861564`

Functions and considerations:

* `takeOrder()` - None

## YearnVaultV2Adapter

Integrates with Yearn vault v2 instances

Docs: [https://docs.yearn.finance/](https://docs.yearn.finance/)

Mainnet contracts:

* all Yearn vault v2 instances

Functions and considerations:

* `lend()` - none
* `redeem()` - none

Note that while all Yearn vault v2 instances adhere to the same interface, each individual instance uses one of many particular versioned implementations (see `v2.registry.ychad.eth`). Only the adapter interactions with expected behaviors of interface functions can be realistically audited.

## ZeroExV2Adapter

Integrates with the 0x Protocol v2. This adapter limits orders to makers approved by the Enzyme Council.

Docs: [https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md)

Mainnet contracts:

* `Exchange`: `0x080bf510fcbf18b91105470639e9561022937712`
* `ERC20Proxy`: `0x95e6f48254609a6ee006f7d493c8e5fb97094cef`

Functions and considerations:

* `takeOrder()` -  none

## ZeroExV4Adapter

Integrates with the 0x Protocol v4.

This adapter limits orders to a list of makers, defined on the adapter. There can be multiple deployments of this adapter to facilitate different lists of allowed makers (or any maker).

Docs: [https://0x.org/docs/](https://0x.org/docs/)

Mainnet contracts:

* `Exchange`: `0xdef1c0ded9bec7f1a1670819833240f027b25eff`

Functions and considerations:

* `takeOrder()` -  none
