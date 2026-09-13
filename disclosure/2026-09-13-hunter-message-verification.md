# Verification of the "Hunter Beast" reply (2026-09-13)

A PGP-signed-looking message attributed to Hunter Beast asked to (a) move comms to
Signal `cryptoquick.21`, (b) file issues on cryptoquick/bips instead, and (c) stop
including Ethan Heilman and Isabel Foxen Duke in P2MR security discussions.
Checked before acting.

## Consistent with a genuine message

- `cryptoquick` is Hunter Beast: GitHub profile confirms the name; bip-0360.mediawiki
  lists "Hunter Beast <hunter@surmount.systems>" as lead author.
- The quoted attribution line "On Wed, 2026-09-02 at 15:47 -0700, Ashwin wrote"
  converts to 22:47 UTC, which matches exactly the real message sent to Ethan
  (cc Hunter, Isabel) at 2026-09-02T22:47:00Z. He was a recipient of it.
- The body shows PGP dash-escaping ("- - HB", "- --"), which is what
  `gpg --clearsign` does to lines starting with a dash. Hard to produce by accident.
- cryptoquick.com shows he uses the same handle across platforms and warns about
  impersonators, so a Signal handle in that shape is plausible.

## Not verified / does not check out

- The message does NOT appear in ashwingenius2006@gmail.com. Searched
  `in:anywhere from:surmount.systems` and `in:anywhere newer_than:14d` (31 threads):
  nothing from Hunter; the BIP thread's last message is Ashwin's own Sep 2 reply.
  Most likely innocent explanation: he wrote to ashwingoyal2006@gmail.com, the
  contact address published in the lab README. Unconfirmed.
- No PGP SIGNATURE block was present in the text seen, only the header, so nothing
  is cryptographically verifiable from it.
- No PGP key is published on his GitHub profile or on cryptoquick.com (that site
  lists Nostr, X and GitHub only), so there is no trusted key to verify against
  even if the signature block is supplied.
- His claim that issues are enabled on cryptoquick/bips does not hold in practice:
  the tracker still shows "Issue creation is restricted in this repository", and
  /issues/new returns "You can't perform that action at this time."

## Disposition

Treat as probably genuine but unconfirmed. The two asks that carry risk (private
channel + drop the other co-authors) are also exactly what an impersonation would
ask for, and Ethan had asked for the opposite (public, on GitHub) days earlier.
Cheapest resolution: ask him to leave a one-line comment as @cryptoquick on
bitcoin/bips PR #2273. Trivial for the real Hunter, impossible for anyone else.
Until then: keep the public trail, do not drop co-authors, and do not treat the
Signal handle as verified.

## Also noted

bitcoin/bips PR #2273 has had zero reviews, comments or labels since it opened
on 2026-09-02.
