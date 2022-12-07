# Staking Wrappers

Some smart contracts that facilitate the staking of TokenA (e.g., OHM) issue an ERC20 TokenB (e.g., sOHM or gOHM) as a receipt for staking deposits. As ERC20s, these staking receipt assets are easily held in fund vaults and transferred to investors upon redeeming their shares.

Others do not natively issue an ERC20 receipt, e.g., Convex. In these cases, there needs to be a method for accounting for the staked position, i.e., some of TokenA has left the vault (it is in the staking protocol), and we need to know how much. While this could be accomplished with an "external position," these positions still have the downside of non-fungibility (non-transferability, etc), and will not be available to funds adhering to stricter trust models.

Instead, the Enzyme Council deploys and maintains "staking wrappers":

Staking wrappers are:

* ERC20-compliant tokens
* state-handlers for multiple depositors, including the checkpointing of staking rewards owed to each depositor
* completely open to any user, i.e., deposits, withdrawals, transfers, and reward-claiming are available outside of the Enzyme protocol
* limitedly pauseable (by the Enzyme Council; only deposits and harvesting new rewards are pauseable; withdrawals and reward claims are not)
* upgradable proxies (by the Enzyme Council; for emergency use only)

## ConvexCurveLpStakingWrapper

A wrapper for staked Curve LP token positions in [Convex Finance](https://docs.convexfinance.com/).

Notes and considerations:

* one wrapper deployment represents one specific staked Curve LP token, e.g., 3pool
* wrappers can be deployed by any party via the `ConvexCurveLpStakingWrapperFactory` (though the Enzyme Council will still need to add them to the asset universe)
* there is only one official wrapper deployment per Convex pool (enforced by the `ConvexCurveLpStakingWrapperFactory`)
