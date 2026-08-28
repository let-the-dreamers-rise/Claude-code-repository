# Disclosure pack — BIP 360 ref-impl findings F2 & F4

Everything here was verified on 2026-08-27 against bitcoin/bips master
(commit `7fe0b034`, 2026-08-20): both bugs are still present, the bips issue
tracker has no prior report of either, and the patch applies cleanly
(`git apply --check` passes) with the reference's own 9/9 vector suite green
before and after, with and without `python -O`.

## Order of operations

1. **Send the email.** It is already sitting as a draft in Gmail for
   ashwingenius2006@gmail.com - subject "BIP 360 ref-impl: missing depth
   bound + validation asserts stripped under -O", patch attached and
   verified byte-for-byte against this folder's copy. Open Drafts, read it
   once, hit Send. (`2026-08-27-email-bip360-authors.md` is the record of
   its contents.)
2. **Reply to Galaxy the same day** with Variant A from `galaxy-reply.md`.
   Never send Variant A before step 1 has actually happened.
3. **When the authors respond**, forward the substance to Galaxy and record
   the verdict in FINDINGS.md — including "intended behavior" verdicts.
4. **PR on bitcoin/bips** (`bips-pr-description.md`): open it if the authors
   ask for it, or after roughly a week of silence. It is also fine to open it
   the same day as the email if you prefer motion — the findings are already
   public in FINDINGS.md, so there is no embargo logic to respect; the
   email-first step is about honoring the process FINDINGS.md promised.

## One repo fix before Galaxy runs the commands

`p2mr-assurance-lab/README.md` still says `24/24 adversarial mutation cases`
(two places: the "Verify" expected-output block and the "Adversarial
mutations (24)" heading). The corpus now has 27 and the runner prints
`27/27` — which is what the grant answers correctly claim. A reviewer
following the README literally sees expected output that does not match
actual output, in a repo whose pitch is "if the outputs don't match, reject."
Change both `24`s to `27` and push.
