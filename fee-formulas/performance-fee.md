# Performance Fee

Performanc Fee for pre-Sulu releases knows the concept of “Crystallisation Period”. While this concept is important in traditional finance, it is also complicated and gas-expensive to properly implement on-chain.

By removing the concept of “crystallisation period”, we can greatly simplify the implementation of the performance fee in the protocol.

This also implies that performance fee can be claimed at any time.&#x20;

Without "crystallisation period" the manager can potentially earn more performance fee through continuous accrual instead of quarterly or yearly accrual. Managers should therefore set the rate for the new simplified performance fee lower than the rate of the previously used performance fee.

### Principles

* Performance fee is paid after a period of constant share supply. Share supply changes on the following actions:
  * buy shares
  * redeem shares
  * claim fees
* Performance fee is only paid if the share price at the end of a share period is larger than the high watermark.
* Only the wealth created for the share price above high watermark is&#x20;
* Performance fee is paid out in shares, as all other fees.
* Performance fee needs to be registered after management fee (i.e. management fee needs to be calculated first and management fee shares need to be ), but be
* Order of fee registrations: Management Fee, Performance Fee, Entrance Fees, Exit Fees

### Formulas

* Call `totalSupply` = $$TS_i$$ i.e. totalSupply before minting or burning shares for the action)
* Read `highWatermark` from storage (this is the share price after the previous performance fee calculation, see below), $$hwm$$
* Current gross share price $$g_i = GAV_i / TS_i$$
* Wealth created during period: $$W_i = max(g_i - hwm, 0) \cdot TS_i$$
* Value of performance fee during period $$F_i = W_i \cdot x%$$, where $$x$$is the performance fee percentage
* Performance fee shares (dilute existing shares): $$f_i = \frac{F_i \cdot TS_i}{GAV_i - F_i}$$
* Calculate share price (after all fees have been minted or burnt): $$g_i^\prime = GAV_i / TS_i^\prime$$ where $$TS^\prime_i$$ is the new total supply after all fees have been settled. If $$g^\prime_i > hwm$$ (i.e. also $$W_i$$ and $$F_i$$ will be larger than zero), then update storage $$hwm = g^\prime_i$$.
