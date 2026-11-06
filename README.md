# EVVM ERC-3009 Gasless Deposit PoC

An independent proof of concept exploring an optional **ERC-3009 gas-sponsored ERC-20 deposit path** for EVVM, with a user-authorized fee split between the executing Fisher and a protocol fee recipient.

> **Status: experimental PoC.** This is not an official EVVM implementation, has not been audited, and is not production-ready. The tests use a mock Core and a mock ERC-3009 token.

## Concept

1. A user signs an ERC-3009 `receiveWithAuthorization` for the net deposit plus a disclosed fee.
2. An active Fisher submits the authorization and pays the host-chain gas.
3. The Treasury credits the net deposit to the user's internal EVVM balance.
4. The fee is credited internally to the Fisher and a configured protocol-fee recipient.
5. The existing, user-paid deposit path remains available.

The user does not need to submit the deposit transaction or hold the host chain's native gas token, but still pays the disclosed fee in the supported ERC-3009 token.

## Important architecture constraint

EVVM's `Core.addAmountToUser()` accepts balance credits only from the configured `treasuryAddress`. Therefore, this PoC is designed as a Treasury implementation for a new EVVM instance configured to use it. It cannot be deployed as an unrelated sidecar and credit balances in an already initialized instance without an intentional Core authorization or migration change.

## Fee model

- `protocolFeeBps` defines the protocol share of the user-authorized relayer fee.
- The remaining fee is credited to the Fisher who executes the deposit.
- Both fee shares are credited as internal EVVM balances in the deposited token.
- This is separate from EVVM Core's existing `priorityFee` mechanism. It does not add a fee to every `Core.pay()` call.
- The Fisher still needs liquidity and a path to convert or withdraw the received token to cover host-chain gas. Relayer quotes must account for gas volatility, conversion costs, and execution risk.

## Security and scope assumptions

- Only explicitly allowlisted ERC-3009 tokens are accepted.
- The caller must be an active EVVM staker.
- An optional executor binding can restrict the authorization to a specific Fisher.
- EVVM-specific parameters are bound into a derived authorization nonce.
- Exact token balance-delta checks reject fee-on-transfer behavior.
- The conventional deposit path remains available.
- The protocol fee recipient and fee split are immutable in this PoC; a production implementation would need carefully designed governance and timelock controls.
- Only tokens with correct ERC-3009 semantics and reliable `balanceOf` behavior should be supported.

## Project structure

```text
.
├── src/
│   └── EVVMERC3009Treasury.sol
├── test/
│   └── EVVMERC3009Treasury.t.sol
├── .github/
│   └── workflows/
│       └── evvm-erc3009-poc.yml
├── foundry.toml
├── ISSUE_DRAFT.md
├── LICENSE
└── .gitignore
```

## Run the tests

Install [Foundry](https://book.getfoundry.sh/getting-started/installation), then run:

```bash
forge test -vvv
```

GitHub Actions runs the Foundry suite on pushes to `main`. The workflow can also be started manually from the Actions tab.

**CI: PASS** The first run in this repository compiled with Solidity 0.8.24 and passed **5 tests, 0 failed**: [view GitHub Actions run](https://github.com/0xSix9/evvm-erc3009-gasless-deposit-poc/actions/runs/42301177592).

### Test coverage

1. Fee split and replay-protection behavior.
2. Executor binding and amount-tampering rejection.
3. Rejection of callers who are not active stakers.
4. Rejection of unsupported tokens.
5. Rejection of expired authorizations.

These are isolated unit tests against mocks. They do **not** establish end-to-end compatibility with a deployed EVVM instance, compatibility with every ERC-3009 token, production gas economics, or production security.

## References

- [EVVM testnet Treasury](https://github.com/EVVM-org/testnet-Contracts/blob/main/src/contracts/treasury/Treasury.sol)
- [EVVM Core](https://github.com/EVVM-org/testnet-Contracts/blob/main/src/contracts/core/Core.sol)
- [EVVM P2PSwap](https://github.com/EVVM-org/testnet-Contracts/blob/main/src/contracts/p2pSwap/P2PSwap.sol)
- [ERC-3009 specification](https://eips.ethereum.org/EIPS/eip-3009)

## License

See [LICENSE](./LICENSE). This PoC uses the EVVM Noncommercial License v1.0 terms from the original experiment. Commercial use requires a separate license from MATE Labs Inc.
