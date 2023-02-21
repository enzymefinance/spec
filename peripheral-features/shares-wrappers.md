# Shares Wrappers

Sometimes, fund participation (depositing, redeeming, and transferring shares) requires rules that are different from the core logic. In such a case, a contract can be deployed that wraps the shares token with the desired logic.

A shares wrapper:

* is an ERC20 token
* is associated with one vault
* accepts end user requests to deposit and redeem to its associated vault (wrapper logic dictates whether these requests pass through atomically, or are queued for subsequent transactions)&#x20;
* holds all shares that are received through its deposits
* issues wrapped shares to the end user 1:1 for the shares received from their deposits&#x20;
* implements any arbitrary additional logic

Generally, a fund using a shares wrapper would use [#alloweddepositrecipientspolicy](../topics/policies.md#alloweddepositrecipientspolicy "mention") to only allow the shares wrapper and no other depositors, thus requiring that all deposit, redemption, and transfer interactions enter through the wrapper. (Note that shares created or transferred by the protocol fee or any fund-level fee would not be subject to the same restriction).

## GatedRedemptionQueueSharesWrapper

A shares wrapper that:

* is compatible with Enzyme v4 and future versions
* defines recurring windows for (user) requesting and (wrapper manager) executing redemptions
* holds redemption requests and allows their execution within the redemption window, up to a specified, collective, per-window relative cap (e.g., 25% of wrapped shares supply)
* optionally requires per-user pre-approvals for wrapped shares actions (deposit, redeem, transfer)
* allows the wrapper manager to force redemptions, i.e., kick depositors from the fund
