# Issue to file at https://github.com/cryptoquick/bips/issues/new

TITLE:

bip-0360 ref-impl (p2mr.py): no merkle depth bound + validation asserts stripped under python -O

BODY (everything below the line):
---------------------------------------------------------------------------
Filing here per your note, so it's trackable on your side. The fix is already
open upstream as bitcoin/bips#2273 - https://github.com/bitcoin/bips/pull/2273 -
so this issue is a tracking record rather than a duplicate of that work.

Found by differential-testing the reference implementation against an independent
implementation written from scratch. The two agree byte-for-byte on all 16 official
construction vectors and on 2000 seeded random script trees, which is what makes me
confident the divergences below are real rather than artifacts of my own code.

### 1. No depth bound - the ref-impl can emit consensus-invalid control blocks

BIP 360 Script Validation: the control block "must have length 1 + 32 * m, for a
value of m that is an integer between 0 and 128, inclusive". `compute_merkle_root`
and `compute_control_block` apply no depth bound, so a 129-deep tree yields a
normal-looking merkle root plus a 4129-byte control block (m = 129) - an output
consensus must reject as unspendable.

```python
import p2mr
leaf = lambda s: {"leafVersion": 0xC0, "script": s}
tree = leaf("51")
for _ in range(129):
    tree = [tree, leaf("52")]
print(len(p2mr.compute_control_block(0, tree)))   # 4129
```

129-deep trees are pathological, so severity is low, but example code that can mint
an unspendable output seems worth a guard before wallet code copies the pattern.

### 2. The binary-tree check is an assert, which python -O strips

`compute_merkle_root` line 102: `assert len(tree) == 2`. Under `-O` that assert is
gone, and a malformed 3-child branch does not raise - it returns the root of its
first two leaves, so the third script leaf silently disappears from the commitment:

```
$ python3 -O
>>> import p2mr
>>> l = lambda s: {"leafVersion": 0xC0, "script": s}
>>> p2mr.compute_merkle_root([l("51"), l("52"), l("53")]).hex()
# identical to the 2-leaf root, no error
```

The usual fix is an explicit raise for input validation, keeping `assert` for
genuine internal invariants.

### Fix and tests

The upstream PR adds the m <= 128 guard (128 still accepted, 129 refused), converts
the validation asserts to `ValueError`, and adds three negative tests wired into
`BIP360_tests()`. The existing 9/9 vector suite passes with the patch, with and
without `-O`.

### Two things I'd call questions rather than bugs

- `tapleaf_hash` refuses empty scripts. Deliberate, or defensive?
- Construction silently masks odd leaf versions (`0xc1` -> `0xc0`, per
  `v = c[0] & 0xfe`). Would you rather refuse odd versions than rewrite them?

Full graded writeup with method and repro instructions:
https://github.com/let-the-dreamers-rise/p2mr-assurance-lab/blob/main/FINDINGS.md
