# Disclosure email — BIP 360 authors

Send from: `ashwingoyal2006@gmail.com` (the address on the repo and the grant application)

**To:** hunter@surmount.systems, ethan.r.heilman@gmail.com, isabel.duke@gmail.com
**Subject:** BIP 360 ref-impl (p2mr.py): two conformance/robustness gaps, with reproducers and a tested patch
**Attachment:** `bip360-ref-impl-fix.patch`

---

Hello Hunter, Ethan, Isabel,

I run an independent conformance-testing project for BIP 360 (P2MR): a from-scratch implementation differential-tested against the reference at `bip-0360/ref-impl/python/p2mr.py`. The two implementations agree byte-for-byte on all 16 official construction vectors (9 construction + 7 PQC) and on 2,000 seeded random script trees — so I believe the two divergences below are signal rather than reimplementation noise. Both reproduce against current master (`7fe0b03`) as well as the commit my suite pins (`ed4ffcb6`).

**1) Missing depth bound: the ref-impl emits consensus-invalid control blocks.**
BIP 360's Script Validation requires the control block length to be `1 + 32 * m` with `m` between 0 and 128 inclusive. `compute_merkle_root` / `compute_control_block` apply no depth bound, so a 129-deep tree yields a valid-looking merkle root and a 4,129-byte control block (m = 129) — an output consensus must reject as unspendable. Reproducer, run from `ref-impl/python`:

    import p2mr
    leaf = lambda s: {"leafVersion": 0xC0, "script": s}
    tree = leaf("51")
    for _ in range(129):
        tree = [tree, leaf("52")]
    print(len(p2mr.compute_control_block(0, tree)))   # 4129 -> m = 129

Severity is low (129-deep trees are pathological), but example code that can mint an unspendable output seems worth a guard before wallet authors copy the pattern.

**2) Structural validation via `assert` — stripped under `python -O`, silently dropping a leaf.**
`compute_merkle_root` enforces the binary-tree invariant with `assert len(tree) == 2`. Under `python -O` the assert is stripped, so a malformed ternary branch does not raise — it returns the root of its first two leaves, i.e. the third script leaf silently vanishes from the commitment:

    $ python3 -O
    >>> import p2mr
    >>> l = lambda s: {"leafVersion": 0xC0, "script": s}
    >>> p2mr.compute_merkle_root([l("51"), l("52"), l("53")]).hex()
    # identical to compute_merkle_root([l("51"), l("52")]).hex() — no error

The usual fix is explicit raises for input validation, keeping `assert` for true internal invariants.

The attached patch does both: adds the `m <= 128` guard (depth 128 still accepted, 129 refused), converts the validation asserts in `compute_merkle_root` / `compute_control_block` to `ValueError`, and adds three negative regression tests wired into `BIP360_tests()`. The existing 9/9 vector suite passes with and without `-O`. Happy to open this as a PR on bitcoin/bips instead if you prefer — or to close the thread with "intended behavior" if that's your read; I'll record either outcome.

Separately, two divergences I've graded as **questions, not bugs**, where I'd value your intent:

- `tapleaf_hash` raises on an empty script. Is the empty-script leaf meant to be unrepresentable at construction, or is that defensive?
- Construction silently masks odd leaf versions (`0xc1` -> `0xc0`) per `v = c[0] & 0xfe`. Intended at construction time, or should surprising input be refused rather than rewritten?

Full graded write-up (method, control results, and all four items): https://github.com/let-the-dreamers-rise/p2mr-assurance-lab/blob/main/FINDINGS.md — everything reproduces from a fresh clone, Python only, no dependencies.

Thanks for BIP 360. Happy to run any revision of the vectors or ref-impl through the suite as the draft evolves.

Ashwin Goyal
P2MR Assurance Lab — https://github.com/let-the-dreamers-rise/p2mr-assurance-lab
