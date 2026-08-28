# Disclosure email — BIP 360 authors

STATUS: already loaded as a Gmail draft in ashwingenius2006@gmail.com with
bip360-ref-impl-fix.patch attached (verified byte-identical to the copy in
this folder). Open Gmail -> Drafts -> hit Send. This file is the record of
what the draft contains.

**To:** hunter@surmount.systems, ethan.r.heilman@gmail.com, isabel.duke@gmail.com
**Subject:** BIP 360 ref-impl: missing depth bound + validation asserts stripped under -O

---

Hi Hunter, Ethan, Isabel,

I've been differential-testing the BIP 360 ref-impl (bip-0360/ref-impl/python/p2mr.py)
against an independent implementation I wrote from scratch. The two agree byte-for-byte
on all 16 official construction vectors and on 2000 seeded random script trees, so I'm
fairly confident the divergences below are real and not my own bugs. Both reproduce on
current master (7fe0b03).

1. No depth bound, so the ref-impl can emit consensus-invalid control blocks.

The BIP's script validation says a control block "must have length 1 + 32 * m, for a
value of m that is an integer between 0 and 128, inclusive". compute_merkle_root and
compute_control_block never check depth: give them a 129-deep tree and you get a
normal-looking root plus a 4129-byte control block (m = 129), i.e. an output consensus
has to reject as unspendable. Repro from ref-impl/python:

    import p2mr
    leaf = lambda s: {"leafVersion": 0xC0, "script": s}
    tree = leaf("51")
    for _ in range(129):
        tree = [tree, leaf("52")]
    print(len(p2mr.compute_control_block(0, tree)))   # 4129

129-deep trees are pathological, so this is low severity, but it seems worth a guard
before wallet code copies the pattern.

2. The binary-tree check is an assert, which python -O strips.

compute_merkle_root line 102: assert len(tree) == 2. Under -O that's gone, and a
malformed 3-child branch doesn't raise - it returns the root of the first two leaves,
so the third script leaf silently disappears from the commitment:

    $ python3 -O
    >>> import p2mr
    >>> l = lambda s: {"leafVersion": 0xC0, "script": s}
    >>> p2mr.compute_merkle_root([l("51"), l("52"), l("53")]).hex()
    # same value as for [l("51"), l("52")], no error

Standard fix is an explicit raise for input validation, keeping assert for internal
invariants.

I've attached a patch that does both: m <= 128 guard (128 still accepted, 129 refused),
validation asserts turned into ValueError, and three negative tests wired into
BIP360_tests(). Your existing 9/9 vectors pass with and without -O. Happy to open it as
a PR on bitcoin/bips instead if that's easier - and if either behavior is actually
intended, tell me and I'll just record that.

Two smaller things I'd call questions rather than bugs:

- tapleaf_hash refuses empty scripts. Deliberate, or just defensive?
- construction silently masks odd leaf versions (0xc1 -> 0xc0, per "v = c[0] & 0xfe").
  Would you rather refuse odd versions than rewrite them?

Full graded writeup with method and repro instructions is in FINDINGS.md at the root of
the repo, let-the-dreamers-rise/p2mr-assurance-lab on GitHub (runs from a fresh clone,
stdlib only, no deps).

If it's useful I'll keep running the suite against future revisions of the vectors and
ref-impl as the draft evolves.

Ashwin Goyal
