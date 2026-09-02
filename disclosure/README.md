# Disclosure pack — BIP 360 ref-impl findings F2 & F4

## Where things stand (2026-08-28)

- Disclosure email SENT to the three BIP 360 authors, with the tested patch attached.
- Ethan Heilman REPLIED (Aug 30, cc Hunter and Isabel): asked for it to be filed on GitHub rather than
  email, because email gets lost.
- Galaxy reply sent (`galaxy-reply.md`), including the delay apology.
- REMAINING: open the PR on bitcoin/bips, reply to Hunter with its URL, tell Galaxy
  the authors engaged, and fix 24/24 -> 27/27 in the lab README.

## Why a PR and not an issue

bitcoin/bips has issues DISABLED: no Issues tab on the repo, and /issues/new
returns 404. A PR is the equivalent persistent GitHub thread there, and it carries
the fix as well as the report. Verified the same day: no open PR touches p2mr.py
(#2232 is unrelated spend-path test vectors), last commit to the file is 75167ec,
Jun 28.

## Files

| File | What it is |
| --- | --- |
| `bips-pr-description.md` | PR title + body, and the no-clone steps to open it |
| `p2mr-patched-full-file.py` | The complete patched p2mr.py to paste in GitHub's editor |
| `bip360-ref-impl-fix.patch` | Same change as a patch (what was emailed) |
| `bips-issue.md` | Issue body, kept in case Hunter names a repo that accepts issues |
| `reply-to-hunter.md` | One-liner to send him once the PR is open |
| `galaxy-reply.md` | The reply already sent to Galaxy, plus the correction note |
| `2026-08-27-email-bip360-authors.md` | Record of what was emailed |

## Verification behind every claim

Both bugs still reproduce on bitcoin/bips master (7fe0b034): the assert is at
p2mr.py line 102, and no depth bound exists anywhere in the file against the BIP's
m <= 128 rule. The patch applies cleanly (`git apply --check`), and the upstream
9/9 vector suite passes with it, with and without `python -O`.

## Correction to log

An earlier draft of the Galaxy reply said "the bips issue tracker has no prior
report of either." That was wrong: bitcoin/bips has no issue tracker at all, so
there was nothing to search. The findings do appear to be unreported upstream (no
open PR touches the file), but the evidence was stated sloppily. If that line went
to Galaxy, send the correction in `galaxy-reply.md`.

## Status 2026-09-02 (actions taken from the session)

- CORRECTION: the reply came from **Ethan Heilman**, not Hunter Beast (Hunter and
  Isabel were cc'd). Earlier notes in this pack said Hunter; fixed.
- REPLIED to Ethan in the same thread (sent, reply-all): bitcoin/bips has issues
  disabled, so a PR is proposed instead, and he was asked to name another repo if
  he'd rather have a real issue.
- p2mr-assurance-lab README 24 -> 27 fixed: PR #2, needs a merge click.
- STILL BLOCKED: the bitcoin/bips PR itself. This session's GitHub access is scoped
  to let-the-dreamers-rise; forking bitcoin/bips was denied and it cannot be
  attached (cross-owner). Open it by hand with the steps in
  `bips-pr-description.md`, or start a fresh session with bitcoin/bips as the
  initial source.
- STILL PENDING: the Galaxy update. The grant thread lives in
  ashwingoyal2006@gmail.com, which is not the mailbox connected here
  (ashwingenius2006@gmail.com), so it has to be sent by hand from
  `galaxy-reply.md`.
