# Injective fork

This is Choice's fork of a PancakeSwap Infinity repository, deployed to Injective EVM
(testnet `1439`, mainnet `1776`). Work branch: `injective`.

## The one rule

**`src/` is never edited.** Upstream's audits (Hexens, OtterSec, Zellic) describe the bytecode
we deploy, and that is only true while our `src/` is byte-identical to the pinned upstream
commit in [.injective-fork-base](.injective-fork-base). CI enforces it
([upstream-guard.yml](.github/workflows/upstream-guard.yml)) and will fail the PR otherwise.

Choice's own Solidity - fee controller, launchpad settler, aggregation router - lives in
`choice_v2_contracts`, never here.

## Diff vs upstream

| Path | Change |
| --- | --- |
| `script/config/injective-testnet.json` | New. Deploy config for chain 1439; addresses are filled in as each numbered script runs. |
| `.injective-fork-base` | New. The upstream commit `src/` is pinned to. |
| `.github/workflows/upstream-guard.yml` | New. Fails the PR if `src/` drifts from the pin; separately reports drift vs `upstream/main` without failing. |
| `.github/workflows/test.yml` | Trigger branch `main` → `injective`; path filters so the suite runs only when protocol code, tests, deps or the pin move; `ci_main` profile dropped (it could only ever fire on a branch named `main`, which this fork does not have). |

Nothing under `src/`.

## Deploy order on Injective

Scripts `01 → 03`, then `06 → 09`. **Upstream `04` and `05` are skipped**: those deploy the
stock protocol fee controllers. Choice deploys `ChoiceFeeController` for CL and Bin from
`choice_v2_contracts` instead and points `setProtocolFeeController` at them, because the fee
needs a treasury / burn-auction split the stock controller has no concept of.

## Moving the pin

Rebase `injective` onto the new upstream commit, update `.injective-fork-base` in the same
commit, run the full suite locally, and record the move in `choice_v2_contracts/deployments`.
The "Upstream drift" job reports how far behind the pin is on every PR, without failing it.

## Deploying

Injective RPCs can return `null` for a mined tx's receipt. Broadcast with `--slow`, **never
use `--resume`**, and confirm every contract with `eth_getCode` rather than with a receipt.
Full runbook: `choice_v2/CHOICE_V2_EVM_PLAN.md` §M1.
