# Issue + epic templates

Use these verbatim. Sections are mandatory. Order is mandatory. Voice is direct, warm, ZA-aligned if the project is ZA-aligned, plain otherwise.

---

## Child issue template — the 8 mandatory sections

```markdown
# Why this matters

<2–3 sentences max. Business / product framing. Why this issue exists at all,
in plain language. No padding. Skip the "context" preamble — go straight to
the stakes.>

## The user

**"<Concrete persona, full name + role + scenario>"** — <one sentence
describing what they're trying to do and where the friction is today>.

**Adversarial case** (if applicable): <a malicious / careless / mistaken
user tries X — what does the system do?>

## What exists today

- <file path>:<line range> — <what's there now>
- <file path>:<line range> — <what's there now>
- Endpoints already mounted (verified by tests in `<test file>`):
  - `GET /api/v1/...` — `{ schema }`
  - `POST /api/v1/...` — `{ schema }`
- Anything else the agent needs to know about the current state — schema
  facts, RLS predicates, seeded users, env vars

## Acceptance criteria

- [ ] <One concrete, testable behaviour an agent can verify pass/fail>
- [ ] <Another>

### <Optional sub-group like "Data fetch">

- [ ] <Sub-grouped criterion>

### <Optional sub-group like "Empty + new-tenant handling">

- [ ] <Sub-grouped criterion>

### <Optional sub-group like "Audit + permissions">

- [ ] <Sub-grouped criterion>

## Implementation notes

- <Gotcha #1 — something an agent reading the diff for the first time would
  trip over>
- <Pattern to mirror from elsewhere in the codebase, with file:line
  reference>
- <Security / RLS / auth detail that's easy to miss>
- <One-sentence "why" for any non-obvious design decision>

## Test plan

- [ ] **Playwright** `e2e/<spec-name>.spec.ts`
  - As <seed user>: <action> → <assertion>
  - As <other seed user, often adversarial>: <action> → <assertion>
- [ ] **Vitest** `tests/<area>/<file>.test.ts`
  - <assertion>
  - <cross-tenant / cross-role check>
- [ ] Manual smoke (if applicable): <one-line description>

## Out of scope

- <Explicit deferral #1 — something an agent might be tempted to include
  but shouldn't>
- <Deferral #2>
- <Deferral #3>
- <Deferral #4>

## Dependencies

- **Blocked by**: #<n> (<one-line summary>) — or "None."
- **Blocks**: #<n>, #<n> — list children that wait on this one.
- **Pairs with**: #<n> — same-sprint companion if any.
```

---

## Epic template

```markdown
# <Area> — <one-line why>

<Intro paragraph: 3–5 sentences framing the problem. What's currently broken /
missing / unscaled. Ground it in a concrete observation (an audit finding,
a real user moment, a measured gap).>

## The mantra

> **<One sentence the agent should hold in mind across every child.>**

<E.g. "Honest empty → real data → real action.">

## The customer / the stakes

<2–4 short paragraphs explaining the business context. Who's affected, when
this becomes a blocker, what doesn't ship until this is done. Cite the
specific persona who feels the pain.>

## What ships in this epic (priority order)

| # | Issue | Priority | Why this priority |
|---|---|---|---|
| 1 | <child name> | <prio> | <one-line rationale tied to user payoff> |
| 2 | <child name> | <prio> | <rationale> |
| ... | ... | ... | ... |

## House style for every child

<Conventions the children all share, so each child issue can be shorter.
Examples: fetch pattern (no SWR), error handling (retry banner not full-
screen), empty state component to use, test discipline.>

- <Convention #1>
- <Convention #2>
- <Convention #3>

## Out of scope for this epic

- <Whole-epic deferral #1>
- <Whole-epic deferral #2>
- <Whole-epic deferral #3>

## How to pick this up

<One paragraph telling the agent how to read + execute a child issue cold.
Reference: "Each child issue is self-contained — file paths, acceptance
criteria, test plan. Read it, do it, ship it as a small PR. Don't read the
conversation that produced it. The audit that found these gaps is in the
git log if you want deeper context.">

## Child issues, in dependency order

(numbers fill in after creation — see linked issues below)
```

The placeholder line `(numbers fill in after creation — see linked issues below)` is **mandatory** because the workflow step 7 of `SKILL.md` does a string-replace on exactly that text.

---

## Voice + tone guide

- **Imperative + direct.** "Wire `/api/v1/today/digest`" not "We should consider wiring."
- **Concrete persona names.** "Dr Patel" beats "the user." Use the project's actual seed users by name.
- **Specific assertions in acceptance criteria.** "Returns 200 + an array of 7 items" beats "should respond correctly."
- **File paths + line numbers in every "What exists today" entry.** No file path = grep first, then write.
- **No exclamation marks.** Calm, warm, deliberate.
- **ZA spelling for ZA projects.** "colour", "organise", "behaviour". Match the project's existing convention — grep for "color" vs "colour" if unsure.

---

## Worked example — auth password-reset issue (real one shipped from Khayalo)

```markdown
# Why this matters

We have no password reset flow at all. A practitioner who forgets her password today has no path forward — she's locked out and has to message us. Every SaaS in the world has "Forgot password?" on the sign-in form; we are conspicuously missing it.

This is also the **instant-kill** mechanism for compromised accounts. Today, "Sign out everywhere" revokes refresh tokens but has up to a 1h tail on already-issued access tokens. The only thing that forces *immediate* re-auth across all devices is a password reset (Supabase invalidates all sessions when the password changes).

## The user

**"Dr Anika Patel"** — comes back to Khayalo three months later, doesn't remember the password she chose. She clicks "Forgot password?" on `/signin`, types her email, hits send. She opens the link in her inbox, picks a new password, lands on `/app/dental` already signed in. About 90 seconds, no support ticket.

**Adversarial case**: someone in the practice goes rogue. Owner clicks "Sign out everywhere", then "Reset my password" — the rogue user is locked out within the minute.

## What exists today

- `/signin` form has no "Forgot password?" link
- No `/api/v1/auth/forgot` endpoint
- No `/signin/reset` form
- Zero matches in the codebase for `resetPasswordForEmail`, `forgot`, `reset` (verified)
- `app/auth/callback/route.ts` already supports `?next=` so we can reuse it for the redirect

## Acceptance criteria

### Request side

- [ ] `/signin` form gets a "Forgot password?" link below the password field...
- [ ] `/signin/forgot/page.tsx` — minimal form: email input + "Send reset link" button...

### Reset side

- [ ] User clicks the email link → lands on `/signin/reset?code=...` (Supabase format)
- [ ] `app/signin/reset/page.tsx` — form: new password + confirm...

### Security + audit

- [ ] Every forgot-password request logs to a new audit_log row with `kind = 'attempt'`, `action = 'auth.password_reset.requested'`...
- [ ] **Sign out everywhere on password change**. The current `auth.updateUser({ password })` only revokes the current session...

## Implementation notes

- Supabase's reset email template can be customised in the dashboard — leave the default for now...
- The middleware in `lib/supabase/middleware.ts` will exchange the `code` for a session on the way in...
- Match the rate-limit pattern from `signin_locked` (RPC + insert into attempts table)...

## Test plan

- [ ] **Vitest** `tests/api/forgot-password.test.ts`
  - POST with a known seed email → 200, audit row written
  - POST with an unknown email → 200 (no enumeration), no audit row for unknown
- [ ] **Playwright** `e2e/password-reset.spec.ts`
  - Click "Forgot password?" on /signin → land on /signin/forgot...

## Out of scope

- Multi-factor / TOTP / passkey support (separate epic)
- "Security questions" or "trusted contact" recovery — passwords-via-email is the floor for SMB
- Customising the Supabase email template HTML — keep the default; brand the email properly when we move to Postmark (issue #161 — production hygiene)

## Dependencies

- None blocking. Pairs well with **issue #161 (production hygiene)** because the email-deliverability story is the same.
- Eventually depends on **issue #156 (resend verification)** for shared rate-limit infrastructure on `signin_attempts.kind`.
```

Note how every assertion is specific and testable. Note how "Out of scope" lists 3 explicit deferrals. Note the cross-references to other issue numbers in "Dependencies."

That's the standard. Match it.
