# Reply to Galaxy — "Have the writeups been disclosed to the maintainers? If so, what was the reaction?"

**Send the disclosure email FIRST (it is sitting ready in Gmail drafts of ashwingenius2006@gmail.com), then this reply the same day.** Variant A assumes the email
has gone out. If for some reason you must reply before sending, use Variant B — never claim
the disclosure happened before it has.

---

## Variant A (recommended — after the email is sent)

Short answer: not when you asked — they have been now, and your question is what fixed the sequencing.

The graded writeups have been public in FINDINGS.md since the repository went up, but direct notification to the BIP 360 authors was scoped into the funded program's coordinated-disclosure step (Q14). Your question exposed that as the wrong order — these two findings don't need funding to report — so today both went to the three BIP 360 authors (Hunter Beast, Ethan Heilman, Isabel Foxen Duke) with standalone reproducers that run against their own file, a tested minimal patch (the missing m ≤ 128 depth guard, plus explicit raises replacing the `assert`-based validation that `python -O` strips), and three negative regression tests — the reference's own 9/9 vector suite passes with the patch, with and without `-O`. I offered to open it as a PR on bitcoin/bips if they prefer.

Reaction: none yet — the mail went out today. I'll forward the first response as it arrives, whatever it says; if the maintainers conclude either behavior is intended, that verdict goes into FINDINGS.md too. Two things you can verify without trusting me in the meantime: both issues are still live on bitcoin/bips master (commit 7fe0b03, Aug 20 — the assert is at bip-0360/ref-impl/python/p2mr.py line 102, and no depth bound exists anywhere in the file against the BIP's m ≤ 128 rule), and the bips issue tracker has no prior report of either, so the findings remain novel as of this email.

## Variant B (only if replying before the email is sent)

Not yet — and the writeups themselves are already public in FINDINGS.md, so let me be precise about what "disclosed" means here. The graded findings, with pinned reproducers, have been publicly readable since the repository went up; what has not happened is direct notification to the BIP 360 authors, because coordinated disclosure was scoped into the funded program (Q14). Your question rightly exposes that as the wrong order for these two — they don't need funding to report. The notification goes to the three BIP 360 authors this week with standalone reproducers and a tested minimal patch; I'll forward their response as soon as it arrives, whatever it says. Both issues are still live on bitcoin/bips master (commit 7fe0b03), and the tracker has no prior report of either.

---

## When the authors respond (whatever they say)

Forward it to Galaxy with one line: what was accepted, what was disputed, and what changed
in FINDINGS.md as a result. A "this is intended behavior" verdict on F1/F3-style questions
is still a result — record it. Never editorialize their response.
