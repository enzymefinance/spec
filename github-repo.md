# GitHub repo

All audited, deployed contracts are located in the `current` branch of our public repo: [https://github.com/enzymefinance/protocol](https://github.com/enzymefinance/protocol)

Unaudited code is located in the `next` branch of our private repo (must request access).

## Contracts

All contracts used for both production and testing are located in `contracts/`

`mocks/` - Mock contracts used only by tests

`persistent/` - Production contracts that will not be changed after their first deployment. These are contracts available to be used across multiple releases, e.g., the overarching structure that facilitates fund migration from an old release to the current release.

`release/` - Production contracts whose lifetime does not extend beyond this release.

`test/` - Other contracts used only by tests

## Tests

All tests are in the `tests/` directory.
