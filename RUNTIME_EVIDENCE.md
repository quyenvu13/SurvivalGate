# RUNTIME_EVIDENCE

Network: GenLayer StudioNet, chain 61999. Source SHA-256: `65249de251e03d4fda463979f99be8220bde0f5f0a2c955039315b751cd42c1d`.
Every row: wallet, method, exact input, expected, tx hash, execution result read back, post-state from a view.
Run date: 2026-10-03, GenLayer Studio, mode Normal (Full Consensus). "EP" is the equivalence-principle output
(the label the validators agreed on). A screenshot or a FINALIZED status alone is not evidence of execution.

## The two pairs that carry the concept

| Pair | What it shows | Rows | Status |
|---|---|---|---|
| **#6 vs #10** | same method, same wallet, **same clause** — success before the close, revert after it; the only difference is the close in #8 | #6 `0xfbc00cd8…8e32` SUCCESS · #10 `0x1e9f6869…905c` reverted | PASS |
| **#9 vs #10** | two clauses of the same agreement, same moment, same wallet — opposite results; the only difference is the wording labelled in #1 and #2 | #9 `0x092dfa27…11dc` SUCCESS · #10 `0x1e9f6869…905c` reverted | PASS |

## Intelligent Contract — 11 transactions

Contract address: `0x294667415F31825ddC29f986B47c3dE747c1ff85` ([explorer](https://explorer-studio.genlayer.com/address/0x294667415F31825ddC29f986B47c3dE747c1ff85)) · deploy tx `0xf3964da6536b5268f4e62b3995b4333c51542e34617d0c388906e7e604c243c8` (FINALIZED · SUCCESS)
Author wallet `0x923a09d0D6e5C242e36C3c1D2071835917cC0bDF` · other wallet `0x76DD809f34e0B72d9339bc509e1E19FaFEB445c2` · agreement id `bdbcd199912aba2f0a8a37a2b0a0e558b918d3097b82ab81486d32aefe69cdde`
Label for every `record_clause`: `the other side`. All rows in **one** agreement.

| # | Wallet | Method and exact input | Expected | Tx hash | Execution result | Post-state read back | Status |
|---|---|---|---|---|---|---|---|
| 1 | author | `record_clause(other, "the other side", S4)` | SURVIVES → STANDING; agreement created LIVE (MV-1a) | `0x165ecaaf2369873ffa5774e7dfe899e4e488a50b2ede4374bc13d596fd6f74ef` | FINALIZED · SUCCESS · EP `SURVIVES` | `get_clause`: fate STANDING, invocable true | PASS |
| 2 | author | `record_clause(other, "the other side", E3)` | DIES_WITH_IT → LAPSED (MV-1b) | `0x2903a86e04bb4e347055020306798e0329c7c2f0cba5a617f43a9319f670ca2e` | FINALIZED · SUCCESS · EP `DIES_WITH_IT` (accepted after one leader rotation) | `get_clause`: fate LAPSED, invocable true | PASS |
| 3 | author | `record_clause(other, "the other side", S3)` | SURVIVES (MV-2a) | `0x119c356b3338d6b2400df7efbdc83eb13048b6b2e3ad0b781648c251a32eae37` | FINALIZED · SUCCESS · EP `SURVIVES` | `get_clause`: fate STANDING | PASS |
| 4 | author | `record_clause(other, "the other side", E4)` | DIES_WITH_IT (MV-2b) | `0xd06799eb9fb481d6107b091bc8786dd4db5e69675f64258aff72a8959dcbb4cb` | ACCEPTED · SUCCESS · EP `DIES_WITH_IT` (3 agree / 2 disagree) | `get_clause`: fate LAPSED; `get_agreement`: clause_count 4, standing 2, lapsed 2, LIVE | PASS |
| 5 | other | `invoke_clause(<S4 id>, "before close")` | success (MV-3a) | `0x8a8d8a6322c70725e6caebb8124fce5994d430d318b38ce6a9ba65da3fa4ce8a` | FINALIZED · SUCCESS | S4 invocation_count 1 | PASS |
| 6 | other | `invoke_clause(<E3 id>, "before close")` | **success** (MV-3b) | `0xfbc00cd8b673aec3ffdda64a12ccabe79bf0b5369f7352b46c036b19d87b8e32` | FINALIZED · SUCCESS | E3 invocation_count 1 (a LAPSED clause invoked while LIVE) | PASS |
| 7 | author | `contest_invocation(<E3 id>, 1, "disputed")` | success | `0x98a931d10f13c41717b32bcd835d6885e48c9834b59dc0860a68dd7dd13fda52` | FINALIZED · SUCCESS | accepted from the author, so invocation #1 on E3 was made by the other wallet | PASS |
| 8 | other | `close_agreement(<author wallet>)` | CLOSED, `closed_by=other`, `closed_at` set — **the event** | `0x41704c386329c6bd25e11da15e039722131805e7e6074cc3508862411f524244` | FINALIZED · SUCCESS | `get_agreement`: CLOSED, closed_by `0x76dd…45c2`, closed_at `2026-10-03T15:32:26.107825Z`, 4 / 2 / 2; `get_clause` S4 `invocable=true`, E3 `invocable=false` | PASS |
| 9 | other | `invoke_clause(<S4 id>, "after close")` | success (MV-3c) | `0x092dfa277ca655f28ec2cb25becf22ec863554843d9f0d84afa51a20ebb011dc` | FINALIZED · SUCCESS (5/5 agree) | | PASS |
| 10 | other | `invoke_clause(<E3 id>, "after close")` | revert *This clause lapsed when the agreement was closed* (MV-3d) | `0x1e9f6869a9027b29da941a6a0a9db2eef364c50d7b1d2e5a99c8107350e6905c` | ERROR · `[rollback] This clause lapsed when the agreement was closed` (validators agree) | | PASS (expected revert) |
| 11 | author | `record_clause(other, "the other side", S5)` | revert *This agreement is closed; no further clauses can be recorded* | `0xf385f178530987147a45beb04b189aea87a9c029ce33af0883324ba891dab539` | ERROR · `[rollback] This agreement is closed; no further clauses can be recorded` (validators agree) | | PASS (expected revert) |

S4 = `Nothing in the closing of this arrangement releases either party from what is already owed.`
E3 = `Once we close, neither party owes the other anything further.`
S3 = `We remain answerable for anything done while the work was live.`
E4 = `The exclusivity applies only while the work is live.`
S5 = `The audit rights outlast the engagement itself.`

Must-verify: MV-1 (#1 vs #2) PASS · MV-2 (#3 vs #4) PASS · MV-3 (#5, #6, #9, #10) PASS.

## Notes on consensus

- Row #2 (E3) was accepted after one leader rotation, and row #4 (E4) with three validators agreeing and two
  disagreeing. Both final labels are the expected ones. Disagreement on these two sentences is consistent
  with them being the hard half of their pairs; a disagreement that did not reach quorum would have failed the
  transaction rather than record a label (fail-closed).
- Rows #10 and #11 are expected reverts: the validators agree on the rollback reason shown.
- After row #8, `get_clause` was read for S4 (`invocable: true`) and E3 (`invocable: false`).
