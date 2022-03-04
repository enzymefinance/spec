# Protocol Fee

This release implements a "protocol fee," which essentially:

* is a tax on AUM
* is levied continuously relative to an annualized target percentage (initially 25 bps)
* results in burning a corresponding amount of $MLN

## Approach

The protocol fee is charged to a fund by minting new shares to an Enzyme Council-administered contract (`ProtocolFeeReserveProxy`). This occurs anytime that a fund:

* receives a new deposit
* has shares redeemed
* migrates to a new release or reconfigures to a new `ComptrollerProxy` (see "Fund Lifecycle" page)

This approach of minting shares rather than directly collecting $MLN or other assets was thoroughly vetted and resulted in the least user friction, leanest architecture, and highest reliability.

It does, however, come with the drawback that the Council-administered `ProtocolFeeReserveProxy` continuously receives shares from _n_ funds, which then need to somehow be converted to $MLN to be burned. Doing this manually via shares redemptions would be work-intensive, cost-inefficient (i.e., gas for redemptions and swaps), and in some cases not even possible (e.g., funds with all value locked in external positions).

To optimize the ease of fee collection and of passively receiving $MLN to burn, this release uses a shares buyback mechanism:

1. Protocol fee shares are minted at an inflated rate (e.g., 50 bps) above the effective target rate (e.g., 25 bps).
2. Funds can use $MLN (i.e., burn) to buy back (i.e., burn) the collected shares at a discount (e.g., 50%).
3. Funds that take advantage of shares buybacks ultimately pay the effective target rate (25 bps) while funds that do not (and leave the Enzyme Council and protocol with the burden of finding other mechanisms for converting shares to $MLN) pay the inflated rate (50 bps).&#x20;

A core configuration option is also provided that allows funds to automatically buyback protocol fee shares whenever they are collected. If `setAutoProtocolFeeSharesBuyback()` is toggled on in the fund's `ComptrollerProxy`, then it will attempt to use the $MLN available in the the `VaultProxy` to atomically buyback the full amount of protocol fee shares collected (during deposit and shares redemption actions).

## Contracts

In order to implement this mechanism, two decoupled contracts with different lifespans and separate concerns are used in tandem:

* `ProtocolFeeTracker` is a non-upgradable, release-level contract that handles state and logic related to tracking the amount of shares that each fund owes at any moment in time
* `ProtocolFeeReserveProxy` is an upgradable, persistent contract that serves as the repository to which protocol fee shares are minted, and its current `ProtocolFeeReserveLib` handles logic for how those shares can be acted upon, i.e., bought back at a discount by the fund in exchange for $MLN

Importantly, all state-changing actions upon `VaultProxy` $MLN holdings (i.e., burn) and shares (i.e., mint and burn) are executed by the `VaultProxy` rather than via these external contracts:

* the `ProtocolFeeTracker` does not have permission to call a minting function on the `VaultProxy`&#x20;
* the `ProtocolFeeReserve` does not have permission to call a burning function on the `VaultProxy`&#x20;
* the `ProtocolFeeReserve` does not actually receive and burn its own $MLN, this is handled inside of the `VaultProxy`

The `ProtocolFeeTracker` and `ProtocolFeeReserve` simply provide the `VaultProxy` with the data it needs handle minting and burning.

This works because the `ProtocolFeeTrakcer` and `ProtocolFeeReserve` operate in complete trust of the `VaultProxy`, and this pattern keeps logic simple and clean, does not expose `VaultProxy` functions to additional state-changing callers, and is gas efficient.
