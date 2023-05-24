# Policies

Policies are intended to provide particular guarantees that help establish trust between current investors and fund managers:

* actions of fund managers
* actions of current investors
* actions of potential investors

Policy rules and behavior:

* Implement one or many "policy hooks", e.g., policies that implement the `PostBuyShares` hook are called to validate state immediately following the minting of new shares by deposit
* Can define whether or not it is disableable
* Can define whether or not it is updatable
* Can be added at any time, _if the hook cannot aversely limit the actions of current investors_ (i.e., policies that implement `PreTransferShares` or `RedeemSharesForSpecificAssets` hooks cannot be added outside of fund setup / migration / reconfiguration)

## Policies: Potential investor actions

These policies limit by whom and on what terms new shares can be received

### AllowedDepositRecipientsPolicy

* Hook: `PostBuyShares`
* Disableable: Yes
* Updatable: Yes (but the list defines if list items are updatable)
* Description: Limits the recipients of new deposits to a list of addresses

### AllowedSharesTransferRecipientsPolicy

* Hook: `PreTransferShares`
* Disableable: Yes
* Updatable: Yes (but the list defines if list items are updatable)
* Description: Limits the recipients of shares transfers to a list of addresses

### MinMaxInvestmentPolicy

* Hook: `PostBuyShares`
* Disableable: Yes
* Updatable: Yes
* Description: Sets bounds on the investment amount of a single deposit
* A max amount of 0 can be used to disable all new deposits

E.g., minimum of 100 USDC and max 10,000 USDC

E.g., minimum of 100 USDC and no max

## Policies: Fund manager actions

These policies limit fund manager actions that might be used to drain, hide value, or act outside of the fund's mandate

### AllowedAdapterIncomingAssetsPolicy

* Hook: `PostCallOnIntegration`
* Disableable: No
* Updatable: No (but the list defines if list items are updatable)
* Description: Limits the assets that can be received via an adapter action

### AllowedAdaptersPolicy

* Hook: `PostCallOnIntegration`
* Disableable: No
* Updatable: No (but the list defines if list items are updatable)
* Description: Limits the `IntegrationManager` adapters that can be used
* Intended purpose: Prevent fund managers from using arbitrary adapters. Most funds should elect to use the Council-maintained list of known adapters.

### AllowedAdaptersPerManagerPolicy

* Hook: `PostCallOnIntegration`
* Disableable: Yes
* Updatable: Yes
* Description: Limits the `IntegrationManager` adapters that can be used, per manager
* Intended purpose: Limit the adapters that each asset manager can use (the owner can use any). Intended for cases where the any asset manager is not fully trusted by the owner.

### AllowedExternalPositionTypesPolicy

* Hooks:
  * `CreateExternalPosition`
  * `ReactivateExternalPosition`
* Disableable: No
* Updatable: No
* Description: Limits the external position "types" (e.g., Compound CDP) that can be used, by blocking adding external positions to the vault.
* Intended purpose: The primary purpose is to prevent a manager from using external positions at all, though it can also be used to limit the kinds of external positions allowed

E.g., No external positions are allowed

E.g., Only Compound CDPs are allowed

### AllowedExternalPositionTypePerManagerPolicy

* Hooks:
  * `CreateExternalPosition`
  * `PostCallOnExternalPosition`
  * `ReactivateExternalPosition`
  * `RemoveExternalPosition`
* Disableable: Yes
* Updatable: Yes
* Description: Limits the external position "types" (e.g., Compound CDP) that can be used, per manager
* Intended purpose: Limit the external position types that each asset manager can use (the owner can use any). Intended for cases where the any asset manager is not fully trusted by the owner.

### CumulativeSlippageTolerancePolicy

* Hook: `PostCallOnIntegration`
* Disableable: No
* Updatable: No
* Description: Limits value loss (i.e., slippage) that can occur via adapter actions over a "tolerance period" (7 days). Funds define their own tolerance amount (e.g., 5%, 10%, etc). When an adapter action results in slippage, that slippage amount is added to a cumulative slippage total. The accumulated slippage then diminishes over the "tolerance period duration" at a constant rate based on the fund's chosen tolerance. This policy allows bypassing the slippage checks entirely if the adapter being called is in a Council-maintained list on the `AddressListRegistry` (adapters that cannot be manipulated by asset managers to steal fund value).
* Intended purpose: Slow the rate at which a malicious manager can drain a fund enough to allow alerting and exiting

E.g., a fund with 10% tolerance can suffer a maximum slippage in any one trade of 10%, and then a maximum slippage of the replenished amount (10% \* secondsPassed / oneWeekInSeconds) until the end of the 7 day tolerance period.

### OnlyRemoveDustExternalPositionPolicy

* Hook: `RemoveExternalPosition`
* Disableable: No
* Updatable: N/A (no settings)
* Description: Allows removing an external position from the vault's `activeExternalPositions` only if its value can be considered negligible (i.e., dust). The dust threshold is maintained by the Council. This policy allows properly-signaled underlying assets of the external position without a valid price to be valued as `0`
* Intended purpose: Prevent a manager from hiding significant value in untracked external positions, while allowing the removal of negligible-valued positions that count towards the vault's `POSITIONS_LIMIT` and add significant gas costs to fund functionality

### OnlyUntrackDustOrPricelessAssetsPolicy

* Hook: `RemoveTrackedAssets`
* Disableable: No
* Updatable: N/A (no settings)
* Description: Allows removing an asset from the vault's `trackedAssets` only if a) it does not have a valid price or b) its value can be considered negligible (i.e., dust). The dust threshold is maintained by the Council.
* Intended purpose: Prevent a manager from hiding significant value in the vault as untracked assets, while allowing the removal of negligible-valued positions that count towards the vault's `POSITIONS_LIMIT` and add significant gas costs to fund functionality, and also allowing the removal of assets with invalid prices that block deposits and other functionality.

## Policies: Current investor actions

These policies prevent investors from intentionally or unintentionally disrupting fund strategies and processes

### AllowedAssetsForRedemptionPolicy

* Hook: `RedeemSharesForSpecificAssets`
* Disableable: Yes
* Updatable: No (but the list defines if list items are updatable)
* Description: Manager defines the assets that are allowed to be included in specific asset redemption

E.g., do not allow any assets (i.e., do not allow specific asset redemption)

E.g., allow only WETH and MLN

### MinAssetBalancesPostRedemptionPolicy

* Hook: `RedeemSharesForSpecificAssets`
* Disableable: Yes
* Updatable: No
* Description: Manager defines the minimum asset balances that must remain in the vault after a specific asset redemption
* Intended purpose: Guaranteed continued functionality of gas relayer (WETH) and auto-shares buyback (MLN) by maintaining min balances

E.g., At least 1 WETH and 10 MLN must remain in the vault after each redemption

###

###

###

###

###

###







