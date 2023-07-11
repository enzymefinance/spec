# External Positions

External positions interact with external protocols, just like adapters. Interactions via an `ExternalPositionProxy` instance and its library affect its value, while interactions via parser contracts only affect validations and the parsing of user input.

## External Position Types

Each external position is associated with a "type" that is written to `ExternalPositionProxy.EXTERNAL_POSITION_TYPE` upon its deployment. This type (e.g., a Compound debt position) instructs the system as to which library and parser contracts to use with proxy interactions.

A vault can have many active external positions of the same type.

There are no globally-enforced limitations on the asset counts that can be managed within any external position. If the need for a such a limitation arises for particular external position types, then a limitation should be imposed by the parser contract for that type.

### AaveDebtPosition (v2)

Handles creating (borrowing), managing, and repaying debt positions on Aave v2.

Aave aTokens are held as collateral directly within the `ExternalPositionProxy` instance, which acts as the owner of the CDP.

Currently only handles variable debt, not stable debt.

Actions and considerations:

* `AddCollateral` - None
* `RemoveCollateral` - None
* `Borrow` - Borrowing accrues stkAAVE to the `ExternalPositionProxy`
* `RepayBorrow` - None
* `ClaimStkAave` - None

### AaveV3DebtPosition

Handles creating (borrowing), managing, and repaying debt positions on Aave v3.

Aave aTokens are held as collateral directly within the `ExternalPositionProxy` instance, which acts as the owner of the CDP.

Currently only handles variable debt, not stable debt.

Actions and considerations:

* `AddCollateral` - can specify either aToken or its underlying (wraps to aToken)
* `RemoveCollateral` - can specify either aToken or its underlying (unwraps aToken collateral)
* `Borrow` - None
* `RepayBorrow` - None
* `SetEMode` - Used to toggle eMode (see Aave docs)
* `SetUseReserveAsCollateral` - Used to toggle isolation mode (see Aave docs)

### CompoundDebtPosition

Handles creating (borrowing), managing, and repaying debt positions on Compound Finance.

Compound cTokens are held as collateral directly within the `ExternalPositionProxy` instance, which acts as the owner of the CDP.

Actions and considerations:

* `AddCollateral` - Receives cTokens from the vault which accrues COMP to the `ExternalPositionProxy`
* `RemoveCollateral` - None
* `Borrow` - Borrowing accrues COMP to the `ExternalPositionProxy`
* `RepayBorrow` - None
* `ClaimComp` - None

### ConvexVotingPosition

Handles vote-locking CVX for vlCVX, delegating voting power to a 3rd party, and claiming rewards from Convex and Votium.

Does not currently support actually voting, which can be easily be accomplished by delegating to a 3rd party and voting directly via Snapshot.

Note that this uses `CvxLockerV2`, the second locker implementation.

Actions and considerations:

* `Lock` - Starts accrual of cvxCRV rewards and any extra rewards
* `Relock` - None
* `Withdraw` - Stops accrual of cvxCRV rewards and any extra rewards
* `ClaimRewards` - None
* `Delegate` - None

### KilnStakingPosition

Handles staking ETH on the Beacon Chain via Kiln

Actions and considerations:

* `ClaimFees` - Only execution layer fees for now. Consensus layer fee support will be added post-Shanghai
* `Stake` - None
* `WithdrawEth` - Transfers any ETH in the contract to the VaultProxy

### LiquityDebtPosition

Handles borrowing LUSD using ETH as collateral on Liquity.

Actions and considerations:

* `OpenTrove` - None
* `AddCollateral` - None
* `RemoveCollateral` - None
* `Borrow` - None
* `RepayBorrow` - None
* `CloseTrove` - None

### MapleLiquidityPosition

Handles lending on Maple Finance

Actions and considerations:

* `CancelRedeemV2` - None
* `LendV2` - None
* `RedeemV2` - None
* `RequestRedeemV2` - None
* `ClaimRewardsV1` - (legacy action for unclaimed MPL rewards)

### SolvV2BondBuyerPosition

Handles buying Solv v2 bonds at IVO (initial voucher offering) and claiming bond value at settlement.

Actions and considerations:

* `BuyOffering` - Once bought, any held voucher will result in an invalid (reverting) GAV/share price until that voucher's maturity
* `Claim` - None

### SolvV2BondIssuerPosition

Handles issuing Solv v2 bonds at IVO (initial voucher offering) and settling bond value at settlement.

Actions and considerations:

* `CreateOffer` - Once created, any issued voucher will result in an invalid (reverting) GAV/share price until that voucher's maturity
* `Reconcile` - None
* `Refund` - None
* `RemoveOffer` - None
* `Withdraw` - None

### TheGraphDelegationPosition

Handles delegating $GRT to indexers on The Graph.

Actions and considerations:

* `Delegate` - Starts accruing $GRT rewards, and charges a flat delegation tax (currently 0.5%) on the delegated $GRT amount&#x20;
* `Undelegate` - Stops accruing $GRT rewards
* `Withdraw` - None

### UniswapV3LiquidityPosition

Handles minting, adding/removing liquidity, collecting fees, and burning UniswapV3 LP positions via Uniswap's `NonfungiblePositionManager`.

A minted nft representing a particular LP position (distinct fee, ticks, and token amounts) is owned by an `ExternalPositionProxy` of this type.&#x20;

One such `ExternalPositionProxy` instance can own and manage multiple nfts of any token pairs (i.e., one fund can manage all of its NFTs in a single contract).

&#x20;Actions and considerations:

* `Mint` - None
* `AddLiquidity` - None
* `RemoveLiquidity` - None
* `Collect` - None
* `Purge` - None

