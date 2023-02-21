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

Docs: [https://docs.aave.com/](https://docs.aave.com/)

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

### ConvexVotingPosition

Handles vote-locking CVX for vlCVX, delegating voting power to a 3rd party, and claiming rewards from Convex and Votium.

Does not currently support actually voting, which can be easily be accomplished by delegating to a 3rd party and voting directly via Snapshot.

Note that this uses `CvxLockerV2`, the second locker implementation.

Docs: [https://docs.convexfinance.com/convexfinance/](https://docs.convexfinance.com/convexfinance/)

Mainnet contracts:

* `BaseRewardPool` (cvxCrv staking) :  `0x3Fe65692bfCD0e6CF84cB1E7d24108E434A7587e`
* `ConvexToken` : `0x4e3FBD56CD56c3e72c1403e103b45Db9da5B9D2B`
* `CvxLockerV2`: `0x72a19342e8F1838460eBFCCEf09F6585e32db86E`
* `MultiMerkleStash` (Votium): `0x378Ba9B73309bE80BF4C2c027aAD799766a7ED5A`
* `vlCvxExtraRewardDistribution`: `0x9B622f2c40b80EF5efb14c2B2239511FfBFaB702`

Actions and considerations:

* `Lock` - Starts accrual of cvxCRV rewards and any extra rewards
* `Relock` - None
* `Withdraw` - Stops accrual of cvxCRV rewards and any extra rewards
* `ClaimRewards` - None
* `Delegate` - None

### LiquityDebtPosition

Handles borrowing LUSD using ETH as collateral on Liquity.

Docs: [https://docs.liquity.org/](https://docs.liquity.org/)

Mainnet contracts:

* `BorrowerOperations`: `0x24179CD81c9e782A4096035f7eC97fB8B783e007`
* `TroveManager` : `0xA39739EF8b0231DbFA0DcdA07d7e29faAbCf4bb2`

Actions and considerations:

* `OpenTrove` - None
* `AddCollateral` - None
* `RemoveCollateral` - None
* `Borrow` - None
* `RepayBorrow` - None
* `CloseTrove` - None

### MapleLiquidityPosition

Handles lending on Maple Finance

Docs: [https://maplefinance.gitbook.io/maple/](https://maplefinance.gitbook.io/maple/)

Mainnet contracts:

* each Maple pool
* `MplRewardsFactory` (v1): `0x0155729EbCd47Cb1fBa02bF5a8DA20FaF3860535`
* `MapleGlobals` (v2): `0x804a6F5F667170F545Bf14e5DDB48C70B788390C`

Actions and considerations:

* `CancelRedeemV2` - None
* `LendV2` - None
* `RedeemV2` - None
* `RequestRedeemV2` - None
* `ClaimRewardsV1` - (legacy action for unclaimed MPL rewards)

### NotionalV2Position

Handles lending and borrowing on Notional Finance v2

Docs: [https://docs.notional.finance/](https://docs.notional.finance/)

Mainnet contracts:

* `Router`: `0x1344A36A1B56144C3Bc62E7757377D288fDE0369`

Actions and considerations:

* `AddCollateral` - None
* `Borrow` - None
* `Lend` - Repaying a `Borrow` is done by calling `Lend` with the desired amount to repay
* `Redeem` - None

### TheGraphDelegationPosition

Handles delegating $GRT to indexers on The Graph.

Docs: [https://thegraph.com/docs/en/](https://thegraph.com/docs/en/)

Mainnet contracts:

* `GraphProxy`: `0xf55041e37e12cd407ad00ce2910b8269b01263b9`

Actions and considerations:

* `Delegate` - Starts accruing $GRT rewards, and charges a flat delegation tax (currently 0.5%) on the delegated $GRT amount&#x20;
* `Undelegate` - Stops accruing $GRT rewards
* `Withdraw` - None

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

