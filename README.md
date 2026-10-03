SurvivalGate does not ask whether a clause is valid, and it does not check a clause against a policy. Every clause here is live and invocable. It asks one thing: when the two sides close the agreement, does this clause reach past that moment or stop at it — and it lets a single closing transaction split the whole register along that line.

# SurvivalGate

GenLayer Intelligent Contract · StudioNet (chain 61999) · py-genlayer v0.2 (`# v0.2.16`)

| | |
|---|---|
| Contract | `contracts/SurvivalGate.py` |
| Address (StudioNet) | `<CONTRACT_ADDRESS>` |
| Deploy tx | `0xf3964da6536b5268f4e62b3995b4333c51542e34617d0c388906e7e604c243c8` |
| Source SHA-256 | `65249de251e03d4fda463979f99be8220bde0f5f0a2c955039315b751cd42c1d` |
| Tested on-chain | `RUNTIME_EVIDENCE.md` — 11 transactions, one hash per row, all as expected |
| Testing summary | `TESTING.md` |

## What it does

- A clause is recorded into an agreement between two wallets. Validators read it **once**:
  `SURVIVES` (it still holds after the agreement closes) or `DIES_WITH_IT` (it stops at the close).
  The label is frozen as a fate, `STANDING` or `LAPSED`; no method changes it.
- Until the agreement is closed, **every clause behaves the same**: either side may invoke any clause.
- `close_agreement` is one deterministic, irreversible transaction that either side may send. It never
  calls the model. After it, `STANDING` clauses stay invocable and `LAPSED` clauses revert with
  *"This clause lapsed when the agreement was closed"*.
- No clause can be recorded after the close, so nobody can wait for the event and then add the clause that
  would have paid off.
- The other side can contest any invocation, also after the close.

**The contract holds no funds.** It moves no money, enforces nothing off-chain and is not an escrow.

## Proven on StudioNet

| What | Transactions | Result |
|---|---|---|
| Same five words `while the work was/is live`, opposite fates | S3 `0x119c356b…ae37` · E4 `0xd06799eb…b4cb` | SURVIVES / DIES_WITH_IT |
| Two sweeping "what is owed at the close" sentences | S4 `0x165ecaaf…74ef` · E3 `0x2903a86e…ca2e` | SURVIVES / DIES_WITH_IT |
| Same wallet, same clause (E3), one close apart | `0xfbc00cd8…8e32` before · `0x1e9f6869…905c` after | SUCCESS / reverted *This clause lapsed when the agreement was closed* |
| Recording after the close | `0xf385f178…b539` | reverted *This agreement is closed; no further clauses can be recorded* |

## Methods

| Write | Who | Notes |
|---|---|---|
| `record_clause(other_wallet, other_label, text)` | the author | creates the agreement on first use; the only model call |
| `invoke_clause(clause_id_hex, note)` | either side | reverts on a `LAPSED` clause once the agreement is `CLOSED` |
| `close_agreement(other_wallet_or_author)` | either side | pass the **other** party's wallet; one way, no reopen |
| `contest_invocation(clause_id_hex, index, note)` | the side that did not invoke | once per invocation; works after the close |

Views return JSON strings; an unknown id returns `"{}"` and never reverts: `get_agreement`, `get_clause`
(includes `fate`, `outcome` and the computed `invocable`), `get_clause_at`, `get_clauses`, `get_invocation`,
`get_contest`, `get_rubric`, `get_limits`. There is no preview or dry-run view.

Ids:
`agreement_id = keccak256("SURVIVAL_GATE:AGREEMENT:V1|" + author + "|" + other)` with lower-case wallets;
`clause_id = keccak256("SURVIVAL_GATE:CLAUSE:V1|" + agreement_id + "|" + len(text) + "|" + text)`, where
`text` is the clause with all whitespace collapsed (Python `" ".join(text.split())`).

## Design choices

- **Fail-safe is `SURVIVES`.** Unclear or unparseable model output keeps a clause alive. A wrong
  `DIES_WITH_IT` would erase the other side's protection at the close and cannot be undone; a wrong
  `SURVIVES` only keeps the author bound longer.
- **No wording game.** The author cannot know which fate helps at the time of writing (a clause binding the
  author is worse for the author if it survives; one binding the other side is better), and the recording
  window closes before the consequence appears.
- **Prompt fence.** The model sees only the rubric, the other side's label (the one stored on the
  agreement) and the clause text, each in its own tag. Text or label containing a tag or an answer token is
  rejected before any model call.
- **Check order.** In `invoke_clause` and `contest_invocation` state checks come before the note check, so a
  lapsed clause always reports the lapse.

## How to try it in GenLayer Studio

Use **two wallets of your own** (one pair of wallets has exactly one agreement).

1. Wallet A: `record_clause(<wallet B>, "the other side", "<clause>")` for two or more clauses — for example
   one that outlives the agreement and one that ends with it. Read `get_clause` to see each fate.
2. Wallet B: `invoke_clause(<clause id>, "note")` on both — both succeed.
3. Either wallet: `close_agreement(<the other wallet>)`. **This cannot be undone.**
4. Invoke again: the `STANDING` clause succeeds, the `LAPSED` clause reverts.

Clause text and label may not contain the words `SURVIVES` or `DIES_WITH_IT` in any letter case: write
"continues after" or "stops at" instead of "survives". Notes are at most 60 characters; keep clause text
under about 160 characters so the calldata stays under 255 bytes.

## Honest limitation

1. **The contract holds no money and enforces nothing off-chain.** It only decides which clauses can still
   be invoked after the agreement closes, and keeps every invocation next to its contest.
2. **An invocation is a self-declared statement.** Nobody proves that what was invoked is true. The value is
   that a clause's fate is frozen **before** the event, so neither side can pick a fate after seeing what it
   would cost.
3. **A wrong `DIES_WITH_IT` is the main risk, and it cannot be repaired.** A clause wrongly labelled `LAPSED`
   stops being invocable at the first `close_agreement`, and no method restores it. Safety nets: the
   `SURVIVES` fail-safe; every fate is readable before anyone closes; contests keep working after the close.
4. **A wrong `SURVIVES` keeps the author bound longer than intended.** The cost falls on whoever wrote the
   unclear sentence, and the two sides can still settle it off-chain. This is deliberate.
5. **Either side can close alone.** That is the "either party may terminate" model, not every real contract.
   There is no two-signature close, and the contract does not check whether the closing side had the right
   to close under any off-chain law.
6. **`MAX_CLAUSES_PER_AGREEMENT` (20) and `MAX_INVOCATIONS` (20 per clause) are hard caps.** A full agreement
   stays `LIVE` but accepts no new clause; it does not close itself. One pair of wallets has exactly one
   agreement; if both wallets each author one naming the other, `close_agreement` from a wallet closes the
   one that wallet authored.

The contract accepts clause text up to 600 characters, but only texts up to about 160 characters fit the
255-byte calldata path that was measured; longer texts are not proven on StudioNet.

License: MIT.
