# PR on bitcoin/bips — the trackable artifact Hunter asked for

Issues are DISABLED on bitcoin/bips (no Issues tab; `/issues/new` 404s), so the
issue he asked for cannot be filed there. A pull request is the equivalent
persistent, notifying GitHub thread on that repo, and it carries the fix as well
as the report. Verified 2026-08-28: no open PR touches p2mr.py (#2232 is
jeanpablojp's spend-path test vectors, unrelated), and the file's last commit is
75167ec, Jun 28.

## Easiest way to open it (no local clone needed)

1. Open https://github.com/bitcoin/bips/blob/master/bip-0360/ref-impl/python/p2mr.py
2. Click the pencil (Edit). GitHub forks the repo to your account automatically.
3. Select all, delete, and paste the full contents of `p2mr-patched-full-file.py`
   from this folder (raw link:
   https://raw.githubusercontent.com/let-the-dreamers-rise/Claude-code-repository/claude/p2mr-assurance-lab-grant-1l48bb/disclosure/p2mr-patched-full-file.py)
4. Commit message: `bip360: bound merkle depth at 128 and replace validation asserts`
5. "Propose changes" -> opens the PR form. Title and body below.

---

**Title:** `bip360: bound merkle depth at 128 and replace validation asserts in ref-impl`

**Body:**

Two conformance/robustness fixes to `bip-0360/ref-impl/python/p2mr.py`, found by differential testing against an independent implementation (byte-for-byte agreement on all 16 official vectors and 2000 random trees before divergence analysis). Sent to the BIP authors by email first; opening it here at their request so it is trackable — issues are disabled on this repo, so a PR is the closest equivalent.

### 1. No depth bound — the ref-impl can emit consensus-invalid control blocks

BIP 360 Script Validation: the control block "must have length 1 + 32 * m, for a value of m that is an integer between 0 and 128, inclusive." `compute_merkle_root` / `compute_control_block` applied no bound, so a 129-deep tree produced a normal-looking root and a 4129-byte control block (m = 129) — an output consensus must reject as unspendable.

```python
import p2mr
leaf = lambda s: {"leafVersion": 0xC0, "script": s}
tree = leaf("51")
for _ in range(129):
    tree = [tree, leaf("52")]
print(len(p2mr.compute_control_block(0, tree)))   # 4129 before this PR
```

Now refused with `ValueError`; the m = 128 boundary is still accepted.

### 2. Validation asserts are stripped under `python -O`

`assert len(tree) == 2` vanishes under `-O`, so a malformed 3-child branch silently returned the root of its first two leaves — the third script leaf dropped from the commitment with no error:

```
$ python3 -O
>>> import p2mr
>>> l = lambda s: {"leafVersion": 0xC0, "script": s}
>>> p2mr.compute_merkle_root([l("51"), l("52"), l("53")]).hex()
# identical to the 2-leaf root before this PR, no error
```

Validation in the construction path now uses explicit raises, which survive `-O`. `assert` is left in place for genuine internal invariants.

### Tests

Adds `negative_structure_tests()` — ternary branch refused, m = 128 accepted, m = 129 refused for both root and control block — and runs it from `BIP360_tests()`. The existing 9/9 vector suite passes, with and without `-O`.

Two related questions I'd rather ask than patch: `tapleaf_hash` refuses empty scripts (deliberate or defensive?), and construction silently masks odd leaf versions `0xc1 -> 0xc0` per `v = c[0] & 0xfe` (would you rather refuse odd versions than rewrite them?). Happy to follow up either way.

Full graded writeup with method and repro instructions: https://github.com/let-the-dreamers-rise/p2mr-assurance-lab/blob/main/FINDINGS.md

cc @notmike-5 @conduition
