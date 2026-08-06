# zNSAnchor contract tests

```bash
cd test && npm install && npm test
```

`compile.js` builds `../zNSAnchor.sol` and `../MockZNSAnchor.sol` with solc 0.8.36 (the
pinned pragma) and fails on any error. `test.js` then exercises the contract's state
machine against an in-process EVM.

Tests run against **MockZNSAnchor**, which overrides `_verifyProof` to a no-op. That is
deliberate: it isolates the contract's own logic — root chaining, height monotonicity,
authorisation, pause, the two-step upgrade and rollback flows — from SP1 proof
generation, which is covered separately by the prover's own suite.

Some tests assert on the **event log** rather than only on resulting state, via the
`events()` helper. Privileged changes are meant to be auditable from logs alone, so
"the state ended up right" is not sufficient: a proposal cancelled without announcing it
would leave observers with no record of what happened to it.

## Two environment quirks worth knowing

* **Every state-changing call passes an explicit `gasLimit`.** Ganache's
  `eth_estimateGas` evaluates against stale state once an earlier estimate has reverted,
  which makes valid calls appear to fail. With a fixed limit a revert instead surfaces as
  receipt `status === 0`, which is what `reverts()` checks.
* **Deploy bytecode needs a `0x` prefix.** solc emits `evm.bytecode.object` without one,
  and ethers silently produces an empty deploy transaction if it is missing — every test
  then "passes" for the wrong reason. `compile.js` output is prefixed at the call site.
