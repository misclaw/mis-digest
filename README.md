# MIS Daily Digest

The email-subscription front door for the misclaw IS-literature project. One
static page with a subscribe form, plus two Cloudflare Pages Functions that do
**double opt-in via Resend, no database**. The daily digest email itself is
sent by a separate cron (`~/research/is-crawler/digest.py`) to the same Resend
segment — this site only handles sign-up + confirmation.

Lives at **https://mis-digest.misclaw.app**. Sibling of
[mis-lit-reviewer](https://github.com/misclaw/mis-lit-reviewer) (the corpus
browser / semantic search), which links here from its header.

## How subscribe works

1. `POST /api/subscribe {email}` creates the contact in Resend with
   `unsubscribed=true` (pending) and emails an HMAC-signed confirmation link.
2. `GET /api/confirm?email=…&t=…` verifies the token and flips the contact to
   `unsubscribed=false`. Only confirmed contacts receive the daily broadcast.

A hidden honeypot field (`website`) silently drops bots; the email is the only
data stored, and every digest carries a one-click unsubscribe.

## Required Pages env vars (production)

Set as encrypted secrets on the `mis-digest` Pages project — **not** in the repo:

| var | what |
| --- | --- |
| `RESEND_API_KEY` | Resend API key (full access) |
| `RESEND_SEGMENT_ID` | segment the daily broadcast targets (`49de5f26-…`) |
| `CONFIRM_SECRET` | HMAC key for the double-opt-in links |
| `DIGEST_FROM` | *(optional)* sender, defaults to `MIS Daily Digest <digest@misclaw.app>` |

```sh
# set/rotate a secret
printf '%s' "<value>" | npx wrangler pages secret put RESEND_API_KEY --project-name mis-digest
```

These reuse the same segment + confirm secret the subscription used while it
lived in `mis-lit-reviewer`, so existing subscribers and the daily broadcast
are unaffected by the move.

## Deploy

Static page + a `functions/` dir, deployed by GitHub Actions
(`cloudflare/wrangler-action`, `pages deploy .`) on every push to `main`.
No build step.
