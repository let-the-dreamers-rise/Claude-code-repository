# The branch is pushed. One click left.

Open this link — it pre-opens the PR form against bitcoin/bips with the branch
already selected:

https://github.com/bitcoin/bips/compare/master...let-the-dreamers-rise:bips:bip360-refimpl-depth-bound-and-raises?expand=1

TITLE (paste):

bip360: bound merkle depth at 128 and replace validation asserts in ref-impl

BODY (paste everything below the line):
--------------------------------------------------------------------------
Two conformance/robustness fixes to `bip-0360/ref-impl/python/p2mr.py`, found by differential testing against an independent implementation (byte-for-byte agreement with the reference on all 16 official construction vectors and on 2000 seeded random script trees before divergence analysis). Reported to the BIP authors by email first; opening it here at @EthanHeilman's request so it is trackable — issues are disabled on this repo, so a PR is the closest equivalent.

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

Now refused with `ValueError`; m = 128 is still accepted.

### 2. Validation asserts are stripped under `python -O`

`assert len(tree) == 2` vanishes under `-O`, so a malformed 3-child branch silently returned the root of its first two leaves — the third script leaf dropped from the commitment with no error:

```
$ python3 -O
>>> import p2mr
>>> l = lambda s: {"leafVersion": 0xC0, "script": s}
>>> p2mr.compute_merkle_root([l("51"), l("52"), l("53")]).hex()
# identical to the 2-leaf root before this PR, no error
```

Validation in the construction path now raises explicitly, which survives `-O`. `assert` is left in place for genuine internal invariants.

### Tests

Adds `negative_structure_tests()` — ternary branch refused, m = 128 accepted, m = 129 refused for both the root and the control block — run from `BIP360_tests()`. The existing 9/9 vector suite passes with and without `-O`.

Two related things I'd rather ask than patch: `tapleaf_hash` refuses empty scripts (deliberate, or defensive?), and construction silently masks odd leaf versions `0xc1 -> 0xc0` per `v = c[0] & 0xfe` (would you rather refuse odd versions than rewrite them?). Happy to follow up either way.

Full graded writeup with method and repro instructions: https://github.com/let-the-dreamers-rise/p2mr-assurance-lab/blob/main/FINDINGS.md

cc @notmike-5 @conduition
--------------------------------------------------------------------------

Do NOT tick "draft" - a maintainer is waiting on this.

## Then, same day

1. Reply to Ethan in the email thread with the PR URL (one line is enough).
2. Send Galaxy the update in galaxy-reply.md with the PR URL filled in.
