# Reply to Galaxy — disclosure question (final, matches reality: authors emailed, replying 2 days late)

Paste as the reply in the same thread as their question. No greeting placeholder —
starts mid-thread on purpose.

---

Thanks for the follow-up, and sorry for the two-day delay in answering - I wanted to
reply with the thing done rather than with a promise, and doing it properly took the
extra day.

The straight answer: no, they had not been disclosed when you asked, and that was my
mistake in sequencing. The writeups have been public in FINDINGS.md since the
repository went up
(https://github.com/let-the-dreamers-rise/p2mr-assurance-lab/blob/main/FINDINGS.md),
but I had scoped direct notification to the authors into the funded
program's disclosure step (Q14). Two weeks between publishing a finding and telling the
people who maintain the code is longer than it should have been, and your question is
what flagged it. Reporting these two costs nothing, so it is done: today I sent both
writeups to the three BIP 360 authors (Hunter Beast, Ethan Heilman, Isabel Foxen Duke)
with reproducers that run against their own file and a tested patch - the missing
m <= 128 depth guard, explicit raises replacing the asserts that python -O strips, and
three regression tests, with their existing 9/9 vector suite passing before and after.
I also offered to open it as a PR on bitcoin/bips.

Reaction: none yet - the mail went out today. I'll forward the first reply whatever it
says, and if the authors rule either behavior intended, that verdict goes into
FINDINGS.md too. In the meantime both issues are still checkable on bitcoin/bips master
(commit 7fe0b03): the assert is at bip-0360/ref-impl/python/p2mr.py line 102, there is
no depth bound anywhere in that file against the BIP's m <= 128 rule, and the tracker
has no prior report of either.

Ashwin

---

## After sending this

1. When the authors respond, forward the substance to Galaxy the same day -
   including an "intended behavior" verdict if that's what comes back.
2. Fix p2mr-assurance-lab README: 24/24 -> 27/27 in both places, before Galaxy
   runs the four commands.
3. bitcoin/bips PR (bips-pr-description.md): open if the authors ask, or after
   about a week of silence.
