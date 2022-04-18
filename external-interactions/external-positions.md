# External Positions

External positions interact with external protocols, just like adapters. Interactions via an `ExternalPositionProxy` instance and its library affect its value, while interactions via parser contracts only affect validations and the parsing of user input.

## External Position Types

Each external position is associated with a "type" that is written to `ExternalPositionProxy.EXTERNAL_POSITION_TYPE` upon its deployment. This type (e.g., a Compound debt position) instructs the system as to which library and parser contracts to use with proxy interactions.

A vault can have many active external positions of the same type.

There are no globally-enforced limitations on the asset counts that can be managed within any external position. If the need for a such a limitation arises for particular external position types, then a limitation should be imposed by the parser contract for that type.

### AaveDebtPosition

Handles creating (borrowing), managing, and repaying debt positions on Aave.

Aave aTokens are held as collateral directly within the `ExternalPositionProxy` instance, which acts as the owner of the CDP.

Currently only handles variable debt, not stable debt.

Docs: [https://docs.aave.com/](https://docs.aave.com)

Mainnet contracts:

* each aToken
* `LendingPoolAddressesProvider`: `0xB53C1a33016B2DC2fF3653530bfF1848a515c8c5`&#x20;
* `ProtocolDataProvider` : `0x057835Ad21a177dbdd3090bB1CAE03EaCF78Fc6d`&#x20;

Actions and considerations:

* `AddCollateral` - None
* `RemoveCollateral` - None
* `Borrow` - Borrowing accrues stkAAVE to the `ExternalPositionProxy`
* `RepayBorrow` - None
* `ClaimStkAave` - None

### CompoundDebtPosition

Handles creating (borrowing), managing, and repaying debt positions on Compound Finance.

Compound cTokens are held as collateral directly within the `ExternalPositionProxy` instance, which acts as the owner of the CDP.

Docs: [https://compound.finance/docs](https://compound.finance/docs)

Mainnet contracts:

* each cToken supported by the `CompoundPriceFeed`
* `CompoundComptroller` - `0x3d9819210A31b4961b30EF54bE2aeD79B9c9Cd3B`

&#x20;Actions and considerations:

* `AddCollateral` - Receives cTokens from the vault which accrues COMP to the `ExternalPositionProxy`
* `RemoveCollateral` - None
* `Borrow` - Borrowing accrues COMP to the `ExternalPositionProxy`
* `RepayBorrow` - None
* `ClaimComp` - None

### MapleLiquidityPosition

Handles lending on Maple Finance.

Docs: [https://maplefinance.gitbook.io/maple/](https://maplefinance.gitbook.io/maple/)

Mainnet contracts:

* each Maple pool
* `MplRewardsFactory`: `0x0155729EbCd47Cb1fBa02bF5a8DA20FaF3860535`
* `PoolFactory` : `0x2Cd79F7f8b38B9c0D80EA6B230441841A31537eC`&#x20;

Actions and considerations:

* `Lend` - None
* `LendAndStake` - Starts accrual of MPL rewards
* `IntendToRedeem` - None
* `Redeem` - Also claims all interest due
* `Stake` - Starts accrual of MPL rewards
* `Unstake` - Stops accrual of MPL rewards for the amount unstaked
* `UnstakeAndRedeem` - Stops accrual of MPL rewards for the amount unstaked
* `ClaimInterest` - None
* `ClaimRewards` - None

### UniswapV3LiquidityPosition

Handles minting, adding/removing liquidity, collecting fees, and burning UniswapV3 LP positions via Uniswap's `NonfungiblePositionManager`.

A minted nft representing a particular LP position (distinct fee, ticks, and token amounts) is owned by an `ExternalPositionProxy` of this type.&#x20;

One such `ExternalPositionProxy` instance can own and manage multiple nfts of any token pairs (i.e., one fund can manage all of its NFTs in a single contract).

Docs: [https://docs.uniswap.org/protocol/reference/periphery/NonfungiblePositionManager](https://docs.uniswap.org/protocol/reference/periphery/NonfungiblePositionManager)

Mainnet contracts:

* `NonfungiblePositionManager` - `0xC36442b4a4522E871399CD717aBDD847Ab11FE88`

&#x20;Actions and considerations:

* `Mint` - None
* `AddLiquidity` - None
* `RemoveLiquidity` - None
* `Collect` - None
* `Purge` - None

