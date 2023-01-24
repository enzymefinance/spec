# Holdings and Shares

The core functionality of a fund is:

* accepting **deposits** from investors
* using those deposits to build a portfolio of on-chain asset **holdings**
* facilitating the **redemption** of shares for access to portfolio holdings

Each fund is configured with a "denomination asset," which is the unit of account for calculating GAV **** and share price.

## **Holdings**

There are two types of holdings that are accounted for in GAV: "tracked assets" and "external positions."

### Tracked Assets

"Tracked assets" are fungible, ERC20-compliant assets that belong to the `VaultProxy` , which are stored in its state as `trackedAssets` .

E.g., WETH, MLN, Compound cTokens, Uniswap V2 LP tokens, or any other asset the comprises the asset universe.

An asset is added as a tracked asset whenever the protocol recognizes that a new asset has been transferred to the `VaultProxy`, e.g., via a trade or DeFi action in an `IntegrationManager` adapter or when withdrawing assets from an external position into the `VaultProxy`.

An asset manager can also explicitly add or remove tracked assets via dedicated `IntegrationManager` actions (each with its own policy hook), though the denomination asset of a fund is always a tracked asset.

Through Enzyme Protocol v3, a fund's holdings consisted only of tracked assets.

### **External Positions**

Starting with Enzyme Protocol v4, "external positions" are available as a second type of holding for cases where an action does not result in a simple exchange of ERC20 assets.

E.g., Compound CDPs, Uniswap v3 LP positions

External positions live outside of the `VaultProxy` as distinct `ExternalPositionProxy` instances, which are not ERC20-compliant and are not divisible. These positions can hold valued assets themselves (e.g., Compound cTokens that serve as collateral for a CDP) or simply represent ownership of a position where value is held outside of the protocol, e.g., Uniswap v3 LP positions or staked assets.

See the "External Positions" page.

### Not Included: Untracked Assets and External Rewards

It is important to note that some assets that "belong" to a fund are not included in its GAV and share price.

"Untracked assets" are those ERC20 assets belonging to the `VaultProxy` that are not included in "tracked assets" state.

"External rewards" are unclaimed assets accrued in external protocols for lending, staking or otherwise participating, for example unclaimed `COMP` accrued for lending and borrowing on Compound.

## Shares

Shares are fungible ERC20 tokens that represent a claim to fund holdings in proportion to the total shares supply.

The canonical value of any amount of shares is the total fund GAV multiplied by the proportion of shares / total supply.

Shares are normalized to 18 decimals.

### Deposits

To deposit, a user calls `ComptrollerProxy.buyShares()` with an amount of the denomination asset to deposit, and the `ComptrollerProxy` transfers the denomination asset amount into the `VaultProxy`, where it is absorbed into the holdings. Shares are then minted to the depositor relative to the current share price.

There is a second, access-controlled `ComptrollerProxy.buySharesOnBehalf()` used by peripheral contracts (i.e., the `DepositWrapper`) that wrap end-user actions during a deposit, such as trading from AssetA into the denomination asset and then depositing. It is important to carefully gate access to depositing on behalf of others, so as to not expose a griefing attack due to the `sharesActionTimelock` (see "Transfers" below).

There is one policy hook (`PolicyHook.PostBuyShares` ) and two fee hooks (`FeeHook.PreBuyShares` and `FeeHook.PostBuyShares`) that run during the common `__buyShares()` logic shared by these two functions, allowing policies to validate the buyer and investment amount and fees to be charged prior to and immediately after changes to the fund holdings and shares supply.

### **Redemptions**

There are two redemption mechanisms available.

In both cases, shares are burned in exchange for access to proportionate underlying holdings.

In both cases, fees can be run prior to the redemption via `FeeHook.PreRedeemShares`.

#### **`redeemSharesInKind()` **&#x20;

The designated `_recipient` receives a proportionate slice of the ERC20 assets in the `VaultProxy` , relative to the amount of shares being redeemed.&#x20;

By default, these assets are limited to the "tracked assets" of the `VaultProxy`, but the redeemer can specify tracked assets to ignore (i.e., forfeit) or untracked assets to include (i.e., ERC20 tokens that belong to the `VaultProxy` but are not "tracked assets").

E.g., FundA has 10 shares units issued and holds 20 WETH and 10 ZRX. UserA redeems 1 share uint (10% of total supply). UserA receives 2 WETH and 1 ZRX.

No policies can run on this function, as it should be continuously available as a guaranteed redemption option, though there are cases where redeeming for full shares value would not be possible:

1. An asset in the fund holdings is not transferable (e.g., due to a pause on the ERC20 asset itself, due to a Synth balance not yet being settleable after a trade on Synthetix, etc)
2. The fund holds value in "external positions," which are not divisible, ERC20 representations, and are not included

This latter point is critical: with rare exception, users in funds that are allowed to use external positions (enforced by policies) should not redeem shares in-kind, as they will only receive a proportion of the ERC20 assets held by the `VaultProxy`, and forfeit the claim to value held in external positions.

#### **`redeemSharesForSpecificAssets()`**

The redeemer specifies one or multiple of the `VaultProxy`'s ERC20 holdings along with the relative values of each to receive (for a total of 100%).

E.g., FundA is denominated in DAI, has 10 shares units issued, and has a total GAV of 1000 DAI. UserA redeems 1 share unit (10% of total supply, worth 100 DAI) and specifies to receive 75% of owed value in DAI and 25% in ZRX. UserA receives 75 DAI and 25 DAI worth of ZRX.

Policies can implement `PolicyHook.RedeemSharesForSpecificAssets`  to define, for example, limits on assets that can be redeemed for.

Importantly, unlike `redeemSharesInKind()`, this option pays the redemption `_recipient` their owed proportion of value _inclusive_ of value stored in external positions. This function should thus be used as the canonical method of redemption for any fund that uses external positions.

### **Shares Action Timelock**

Each fund configures its own `sharesActionTimelock`, which defines the number of seconds that must pass after UserA's last receipt of shares via deposit, before being allowed to either redeem or transfer any shares.

This is an arbitrage protection, and funds that have untrusted investors should use a non-zero value.

### Transfers

Shares are ERC20-compliant and are transferable by default, though there are a couple of validations that can block transfers:

1. UserA cannot transfer shares to any user until UserA's "shares action timelock" has expired
2. A `PolicyHook.PreTransferShares` enables policies to validate the conditions of a transfer (e.g., a whitelist of allowed recipients)

Because fund configuration (including policies) is changeable via a migration or a reconfiguration, this second point is particularly problematic for secondary markets or any other smart contract holder of shares tokens: if a fund were to add a policy that blocked transfers out of a Uniswap pool, for example, LP providers would be stuck in un-withdrawable positions.

It would further be impractical to liquidity providers or builders if there were no core guarantees that once their contracts receive shares via a transfer in, they would always be able to transfer them out.

For those funds that would to provide such a guarantee to builders or users of secondary applications, there is thus a persistent `freelyTransferableShares` config option on the `VaultProxy` . This configuration option will not run `PolicyHook.PreTransferShares` upon shares transfer, and will persist between migrations and reconfigurations. Once set, it cannot be unset.
