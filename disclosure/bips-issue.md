Title: bip-0360 ref-impl (p2mr.py): no merkle depth bound + validation asserts stripped under python -O

Copy everything BELOW this line into the issue body at https://github.com/bitcoin/bips/issues/new
---------------------------------------------------------------------------------------------------

I've been differential-testing the BIP 360 ref-impl (`bip-0360/ref-impl/python/p2mr.py`) against an independent implementation written from scratch. The two agree byte-for-byte on all 16 official construction vectors and on 2000 seeded random script trees, so I'm fairly confident the divergences below are real. Both reproduce on current master (7fe0b03). Originally sent to the BIP authors by email; filing here at their request so it's trackable.

### 1. No depth bound - the ref-impl can emit consensus-invalid control blocks

BIP 360's script validation says the control block "must have length 1 + 32 * m, for a value of m that is an integer between 0 and 128, inclusive". `compute_merkle_root` and `compute_control_block` never check depth: a 129-deep tree yields a normal-looking root plus a 4129-byte control block (m = 129), i.e. an output consensus has to reject as unspendable.

```python
import p2mr
leaf = lambda s: {"leafVersion": 0xC0, "script": s}
tree = leaf("51")
for _ in range(129):
    tree = [tree, leaf("52")]
print(len(p2mr.compute_control_block(0, tree)))   # 4129
```

129-deep trees are pathological, so low severity - but example code that can mint an unspendable output seems worth a guard before wallet code copies the pattern.

### 2. The binary-tree check is an assert, which python -O strips

`compute_merkle_root` line 102: `assert len(tree) == 2`. Under `-O` that's gone, and a malformed 3-child branch doesn't raise - it returns the root of the first two leaves, so the third script leaf silently disappears from the commitment:

```
$ python3 -O
>>> import p2mr
>>> l = lambda s: {"leafVersion": 0xC0, "script": s}
>>> p2mr.compute_merkle_root([l("51"), l("52"), l("53")]).hex()
# same value as for [l("51"), l("52")], no error
```

Standard fix is an explicit raise for input validation, keeping assert for internal invariants.

### Proposed fix

The patch below (same one sent by email) adds the m <= 128 guard (128 still accepted, 129 refused), turns the validation asserts into `ValueError`, and adds three negative tests wired into `BIP360_tests()`. The existing 9/9 vector suite passes with and without `-O`. Happy to open it as a PR if that's preferred.

<details>
<summary>bip360-ref-impl-fix.patch</summary>

```diff
--- a/bip-0360/ref-impl/python/p2mr.py
+++ b/bip-0360/ref-impl/python/p2mr.py
@@ -23,6 +23,11 @@
 BECH32M_CONST = 0x2BC830A3
 MAX_COMPACT_SIZE = 2**64 - 1
 
+# BIP 360, Script Validation: a control block "must have length 1 + 32 * m, for a
+# value of m that is an integer between 0 and 128, inclusive" -- so no script leaf
+# may sit deeper than 128 branches, or its control block is consensus-invalid.
+CONTROL_MAX_NODE_COUNT = 128
+
 # A script tree node is either a leaf (dict) or a branch (list of nodes)
 ScriptTree = Union[Dict[str, Any], List["ScriptTree"]]
 
@@ -88,7 +93,7 @@
     return tagged_hash("TapBranch", b"".join(sorted((left, right))))
 
 
-def compute_merkle_root(tree: ScriptTree) -> bytes:
+def compute_merkle_root(tree: ScriptTree, depth: int = 0) -> bytes:
     """Recursively compute script tree merkle root"""
     if isinstance(tree, dict):  # Leaf
         version = tree["leafVersion"]
@@ -99,8 +104,15 @@
         # Script trees are treated strictly as binary trees; each branch node should have
         # exactly 2 children. This isn't a general n-ary fold, and combining
         # more than 2 children sequentially would not produce a valid P2MR merkle root.
-        assert len(tree) == 2, f"expected binary branch, got {len(tree)} children"
-        left, right = compute_merkle_root(tree[0]), compute_merkle_root(tree[1])
+        if depth >= CONTROL_MAX_NODE_COUNT:
+            raise ValueError(
+                f"compute_merkle_root: leaf deeper than {CONTROL_MAX_NODE_COUNT}; "
+                "its control block would exceed the consensus length limit"
+            )
+        if len(tree) != 2:
+            raise ValueError(f"expected binary branch, got {len(tree)} children")
+        left = compute_merkle_root(tree[0], depth + 1)
+        right = compute_merkle_root(tree[1], depth + 1)
         return tapbranch_hash(left, right)
 
     else:  # badbadnotgood
@@ -118,18 +130,26 @@
     """
     if isinstance(tree, dict):
         return bytes([tree["leafVersion"] | 1])
-    assert isinstance(tree, list) and len(tree) == 2
+    if not (isinstance(tree, list) and len(tree) == 2):
+        raise ValueError("compute_control_block: invalid tree node")
 
     control_block = b""
 
     while isinstance(tree, list):
-        assert len(tree) == 2
+        if len(tree) != 2:
+            raise ValueError(f"expected binary branch, got {len(tree)} children")
         sibling = tree[(path & 1) ^ 1]
         tree = tree[(path & 1)]
         control_block = compute_merkle_root(sibling) + control_block
         path >>= 1
 
-    assert isinstance(tree, dict)
+    if not isinstance(tree, dict):
+        raise ValueError("compute_control_block: path does not terminate at a leaf")
+    if len(control_block) > 32 * CONTROL_MAX_NODE_COUNT:
+        raise ValueError(
+            f"compute_control_block: {1 + len(control_block)} bytes exceeds the "
+            f"consensus limit of 1 + 32 * {CONTROL_MAX_NODE_COUNT}"
+        )
     return bytes([tree["leafVersion"] | 1]) + control_block
 
 
@@ -412,6 +432,39 @@
         return False
 
 
+def negative_structure_tests() -> None:
+    """Malformed trees must raise -- not silently mis-hash -- including under python -O."""
+    print("\nRunning negative structure tests...")
+    leaf_a = {"leafVersion": 0xC0, "script": "51"}
+    leaf_b = {"leafVersion": 0xC0, "script": "52"}
+    leaf_c = {"leafVersion": 0xC0, "script": "53"}
+
+    # A ternary branch must be refused, never reduced to its first two children.
+    try:
+        compute_merkle_root([leaf_a, leaf_b, leaf_c])
+        raise SystemExit("FAILED: ternary branch accepted")
+    except ValueError:
+        pass
+
+    # A leaf may sit at depth 128 (m = 128) but never deeper.
+    tree: ScriptTree = leaf_a
+    for _ in range(CONTROL_MAX_NODE_COUNT):
+        tree = [tree, leaf_b]
+    cb = compute_control_block(0, tree)  # depth 128: still valid
+    if len(cb) != 1 + 32 * CONTROL_MAX_NODE_COUNT:
+        raise SystemExit("FAILED: wrong control block length at maximum depth")
+
+    tree = [tree, leaf_b]  # depth 129: consensus-invalid
+    for attempt in (lambda: compute_merkle_root(tree), lambda: compute_control_block(0, tree)):
+        try:
+            attempt()
+            raise SystemExit("FAILED: >128-deep tree accepted")
+        except ValueError:
+            pass
+
+    print("3/3 negative structure tests passed.")
+
+
 def BIP360_tests() -> None:
     """Run all BIP-360 Test Vectors."""
     print("\nRunning BIP-0360 Pay-to-Merkle-Root (P2MR) Tests...")
@@ -421,6 +474,7 @@
 
     passed = sum(run_single_test(v, i + 1) for i, v in enumerate(test_vectors))
     print(f"\n{passed}/{len(test_vectors)} BIP-360 tests passed successfully.")
+    negative_structure_tests()
 
 
 if __name__ == "__main__":
```

</details>

### Two things I'd call questions rather than bugs

- `tapleaf_hash` refuses empty scripts. Deliberate, or just defensive?
- construction silently masks odd leaf versions (`0xc1` -> `0xc0`, per `v = c[0] & 0xfe`). Would you rather refuse odd versions than rewrite them?

Full graded writeup with method and repro instructions: https://github.com/let-the-dreamers-rise/p2mr-assurance-lab/blob/main/FINDINGS.md (runs from a fresh clone, stdlib only, no deps).

cc @notmike-5 @conduition
