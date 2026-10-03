# TESTING

```
COMPILE PASS ≠ RUNTIME PASS
SUBMITTED ≠ ACCEPTED ≠ FINALIZED ≠ EXECUTION SUCCESS ≠ POSTCONDITION PASS
```

## On-chain (GenLayer StudioNet, 2026-10-03)

One run of 11 transactions with two wallets, all in one agreement, in GenLayer Studio (Normal / Full
Consensus). Every row behaved as expected. Full table with inputs, tx hashes, results and read-back state:
`RUNTIME_EVIDENCE.md`.

| Check | Rows | Result |
|---|---|---|
| **MV-1** — two sweeping "what is owed at the close" sentences get opposite labels | S4 → `SURVIVES`, E3 → `DIES_WITH_IT` | PASS |
| **MV-2** — same five words `while the work was/is live`, opposite labels | S3 → `SURVIVES`, E4 → `DIES_WITH_IT` | PASS |
| **MV-3** — four cells around one close | before: S4 ✓, E3 ✓ · after: S4 ✓, E3 reverted | PASS |
| contest by the other side | `contest_invocation` on E3 by the author | PASS |
| the closing event | `close_agreement` by the other side → `CLOSED`, `closed_by`, `closed_at` set; `get_clause` S4 `invocable: true`, E3 `invocable: false` | PASS |
| no recording after the close | `record_clause` → reverted *This agreement is closed; no further clauses can be recorded* | PASS |

Test cases used (declared label for every case: `the other side`):

| Case | Text | Expected |
|---|---|---|
| S4 | Nothing in the closing of this arrangement releases either party from what is already owed. | SURVIVES |
| E3 | Once we close, neither party owes the other anything further. | DIES_WITH_IT |
| S3 | We remain answerable for anything done while the work was live. | SURVIVES |
| E4 | The exclusivity applies only while the work is live. | DIES_WITH_IT |
| S5 | The audit rights outlast the engagement itself. | (sent after the close; must revert) |

Consensus: E3 was accepted after one leader rotation and E4 with a 3–2 vote; both final labels are the
expected ones. A disagreement that does not reach quorum fails `record_clause` instead of recording a
label (fail-closed by design).

## Offline (before deploy)

- **genvm-linter** `lint`, `schema`, `typecheck`: pass; 12 methods (4 write, 8 view), no type errors.
- **Direct Mode tests** (`genlayer-test`, the real py-genlayer v0.2.16 SDK, model mocked): 58 passed —
  the four `_is_invocable` combinations, the four-cell invoke sequence, all 22 revert strings each with a
  dedicated test, check order, id formulas, fail-safe, validator function, prompt fence, views.
- **Rubric gate**: no word or word pair separates the two test classes, and the rubric shares no content
  word with any test case.
- **Calldata**: every call used on-chain is between 79 and 185 bytes, under the 255-byte RPC limit; the
  longest clause text that fits with the label `the other side` is 161 characters.

## What this run does NOT prove

- Only 4 semantic cases were labelled on-chain (the two hardest pairs), one run each; label stability
  across repeated runs or other validator sets is not measured.
- Clause texts longer than about 160 characters (the contract allows 600) were not sent.
- Prompt-injection resistance rests on the fence and the door check; no adversarial model run was done.
- `closed_at` is the node's datetime; nothing in the contract depends on it.
