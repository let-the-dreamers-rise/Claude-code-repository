# Galaxy update — send from ashwingoyal2006@gmail.com, reply in their thread

Subject (if the thread needs one): BIP 360 disclosure - update

---

Following up on the disclosure question with where it actually landed.

The two findings went to all three BIP 360 authors on 29 August. Ethan Heilman
replied the next day asking me to move it onto GitHub rather than email, since
email gets lost, and again on 2 September to say open a PR. That is done:

https://github.com/bitcoin/bips/pull/2273

It carries both findings with runnable reproducers, the fix itself, and three
regression tests. The reference implementation's existing 9/9 vector suite passes
with the patch applied, with and without python -O. bitcoin/bips has issues
disabled, so a pull request is the trackable equivalent on that repository.

Hunter Beast, the lead author, replied on 8 September, PGP-signed, to say thanks,
to offer his own BIPs fork for issue filing, and to suggest moving further
security discussion to Signal. I have agreed to a private channel for anything
genuinely sensitive in future, and kept the two current findings public, since
they have been in FINDINGS.md since the repository went up and are now in the PR.

The PR is still unreviewed, which is unremarkable for a repository carrying that
large an open-PR backlog.

One correction to my earlier note. I said the bips issue tracker had no prior
report of these findings. bitcoin/bips has no issue tracker at all, so there was
nothing for me to have searched. What is accurate is that no open pull request
touches that file and its last commit was in June, so the findings were
unreported upstream. I would rather correct my own evidence than leave a claim
standing that does not survive checking.

So the sequence is complete end to end: published in FINDINGS.md, disclosed
privately to the authors, and now upstream as a pull request at a co-author's
request. The two findings are the same two the repository has claimed since day
one.

Ashwin
