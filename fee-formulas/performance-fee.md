# Performance Fee

### Principles

* Performance fee is paid after a period of constant share supply. Share supply changes on the following actions:
  * buy shares
  * redeem shares
  * claim fees
* Performance fee is only paid if the share price at the end of a share period is larger than the high watermark.
* Only the wealth created for the share price above high watermark is taken into account.
* Performance fee is paid out in shares, as all other fees.
* Order of fee registrations: Management Fee, Performance Fee, Entrance Fees, Exit Fees

### Formulas

* Call `totalSupply` = $$TS_i$$ i.e. totalSupply before minting or burning shares for the action)
* Read `highWatermark` from storage (this is the share price after the previous performance fee calculation, see below), $$hwm$$
* Current gross share price $$g_i = GAV_i / TS_i$$
* Wealth created during period: $$W_i = max(g_i - hwm, 0) \cdot TS_i$$
* Value of performance fee during period $$F_i = W_i \cdot x%$$, where $$x$$is the performance fee percentage
* Performance fee shares (dilute existing shares): $$f_i = \frac{F_i \cdot TS_i}{GAV_i - F_i}$$
* Calculate share price (after all fees have been minted or burnt): $$g_i^\prime = GAV_i / TS_i^\prime$$ where $$TS^\prime_i$$ is the new total supply after all fees have been settled. If $$g^\prime_i > hwm$$ (i.e. also $$W_i$$ and $$F_i$$ will be larger than zero), then update storage $$hwm = g^\prime_i$$.



{% hint style="info" %}
On Sulu(v4) we removed the crystallisation period. Without a "crystallisation period", the manager can potentially earn more performance fees through continuous accrual instead of quarterly or yearly accrual. Managers should, therefore, set the rate for the new simplified performance fee lower than the rate of the previously used performance fee.
{% endhint %}
