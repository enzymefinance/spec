# ManagementFee

## Some definitions

Management fee rate (annual, in percent):`x`

Effective management fee rate (annual, in percent, after dilution): `k`

Since management fee is not paid out (as a percentage of assets), but is allocated as newly minted shares in the fund, we need to use the effective management fee rate. This ensures that the manager receives the correct ratio of shares.

The two fee rates are related as follows:

$$x = \frac{k}{1+k}$$

or, alternatively

$$k = \frac{x}{1-x}$$

## Continuous compounding

Management fee accrual happens at irregular and unknown intervals, so we have to resort to continuous compounding. The continuous management fee rate `z` is related to the annual effective management fee rate `k` as follows:

$$e^{z} = 1 + k$$

or, alternatively

$$z = ln(1+k)$$

Substituting for the effective management fee rate `k` yields the relation between the continuous management fee rate and the annual management fee rate:

$$e^{z} = \frac{1}{1-x}$$

or, alternatively

$$z=-ln(1-x)$$

## Management fee allocation

Whenever management fee is due after a time period `t` (expressed as a fraction of a year), the number of shares changes as follows

$$S' = e^{z\cdot t} S$$

`S` is the total supply of shares before the allocation of the management fee shares, and `S'` is the total supply of shares after the allocation of the management fee shares.

The share allocation to the manager is `S_{manager} = S'-S`, and it is calculated as follows:

$$S_{manager} = \left( \frac{1}{(1-x)^{t}} -1\right) S$$

or

$$S_{manager} = \left( (1+k)^{t} -1\right) S$$

Using `t = \Delta t / N`, we can rewrite this as

$$S_{manager} = ( f^{\Delta t} -1) S$$

where

$$f = (1+k)^{1/N}$$

`f` is calculated off-chain when configuring the fee, and it is stored on-chain as `scaledPerSecondRate` . The on-chain computation is then

`sharesDue = (rpow(scaledPerSecondRate, numberOfSeconds, 10**27) - 10**27) * totalSupply / 10**27`
