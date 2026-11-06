# GitHub Issue Draft

**Target repository:** https://github.com/EVVM-org/testnet-Contracts/issues/new

**Suggested title:** [Feature Proposal] ERC-3009 gas-sponsored deposits with Fisher/protocol fee split

---

Hi EVVM maintainers,

I reviewed the current Treasury/Core flow and would like to propose an **optional ERC-3009 gas-sponsored ERC-20 deposit path with an explicit relayer/protocol fee split**.

## Proposal

The proposed flow is:

1. The user signs an ERC-3009 `receiveWithAuthorization` for the deposit amount plus a disclosed fee.
2. An active Fisher submits the authorization and pays the host-chain gas.
3. The Treasury credits the net deposit to the user's internal EVVM balance.
4. The fee is split between the executing Fisher and a configured protocol-fee recipient, as internal balances.
5. The conventional deposit route remains available.

The user does not need a prior ERC-20 approval transaction or native gas tokens to submit the deposit, but still pays the explicitly disclosed fee in the supported ERC-3009 token.

## Reference implementation

I prepared an independent, unaudited proof of concept with Foundry tests:

**Repository:** https://github.com/0xSix9/evvm-erc3009-gasless-deposit-poc

**Contract:** https://github.com/0xSix9/evvm-erc3009-gasless-deposit-poc/blob/main/src/EVVMERC3009Treasury.sol

**Tests:** https://github.com/0xSix9/evvm-erc3009-gasless-deposit-poc/blob/main/test/EVVMERC3009Treasury.t.sol

**CI results:** https://github.com/0xSix9/evvm-erc3009-gasless-deposit-poc/actions/runs/42301177592

The Foundry suite passed 5 tests with 0 failures using Solidity 0.8.24. The tests cover fee splitting, replay protection, executor binding, amount tampering, non-staker rejection, unsupported-token rejection, and expired authorizations.

These are isolated unit tests using a mock Core and mock ERC-3009 token. They do not establish end-to-end compatibility with a deployed EVVM instance, compatibility with every ERC-3009 token, or production readiness. The PoC has not been audited.

## Integration constraint

EVVM's `Core.addAmountToUser()` only accepts calls from the configured `treasuryAddress`. Therefore, a standalone sidecar Treasury cannot directly credit balances in an already initialized EVVM instance. The approach appears to require integration into the configured Treasury for a new EVVM instance, or an intentional migration/Core authorization design for existing instances.

Also, the Fisher's reward would be an internal balance in the deposited token, not native gas. The relayer economics must account for withdrawal/conversion liquidity and host-chain gas volatility.

Would the team consider this direction? If so, would you prefer it as an extension of the configured Treasury, a separate Treasury implementation for new deployments, or a migration/authorized-credit path for existing instances?

Thank you for your time.
