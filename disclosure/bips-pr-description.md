# PR for bitcoin/bips (open only after the email, or if the authors ask for it)

Branch name suggestion: `bip360-ref-impl-depth-bound-and-raises`
Apply the patch with: `git apply bip360-ref-impl-fix.patch` on a fresh fork of bitcoin/bips.

**Title:** `bip360: bound merkle depth at 128 and replace validation asserts in ref-impl`

---

Two small conformance/robustness fixes to `bip-0360/ref-impl/python/p2mr.py`, found by differential testing against an independent implementation (byte-for-byte agreement on all 16 official vectors and 2,000 random trees before divergence analysis; graded write-up: https://github.com/let-the-dreamers-rise/p2mr-assurance-lab/blob/main/FINDINGS.md).

**1. Missing depth bound (consensus-invalid control blocks).** BIP 360 Script Validation: the control block "must have length 1 + 32 * m, for a value of m that is an integer between 0 and 128, inclusive." `compute_merkle_root` / `compute_control_block` had no bound, so a 129-deep tree produced a valid-looking root and a 4,129-byte control block (m = 129) — an output consensus must reject as unspendable. Now refused with `ValueError`; the m = 128 boundary is still accepted.

**2. Validation asserts stripped under `python -O`.** `assert len(tree) == 2` vanishes under `-O`, so a malformed ternary branch silently returned the root of its first two leaves — the third leaf dropped from the commitment with no error. Validation in the construction path now uses explicit raises, which survive `-O`.

Adds `negative_structure_tests()` (ternary branch refused; m = 128 accepted; m = 129 refused for both root and control block) and runs it from `BIP360_tests()`. The existing 9/9 vectors pass, with and without `-O`.

cc @notmike-5 @conduition
