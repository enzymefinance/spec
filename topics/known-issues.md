# Known Issues

These are issues that have been reported by whitehats, auditors, or internal team members, for which no fix will be made. All issues have been assessed for severity in consultation with our regular auditors and technical committee.

## Incorrect share price for depositors and redeemers if "auto-buyback of protocol fee shares" is used

Issue: During deposit and redeem, if a fund has `autoProtocolFeeSharesBuyback` turned on, an incorrect share price is constructed as: `pre-buyback GAV / post-buyback total supply`.

Effect: Results in too-high share price used to deposit or redeem shares

Mitigation: while mis-pricing should generally be small, unless you are confident in assessing impact, avoid using `autoProtocolFeeSharesBuyback` .
