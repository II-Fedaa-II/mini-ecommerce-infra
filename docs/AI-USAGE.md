# AI usage — development methodology

## Which tools I used

Claude Code (Anthropic's agentic CLI) end to end — not autocomplete-style
suggestions accepted line by line, but an agent I directed with instructions,
that read and wrote files, ran the test suites, ran the Docker stack, and
reported back what happened.

## When in the lifecycle I used it

At every stage: initial architecture scaffolding, feature implementation,
writing both unit and e2e tests, debugging failures, and building out the
Docker/infra setup. I did not write this project by hand and have Claude
assist at the margins — I directed Claude through the whole build, gave it
the constraints and decisions up front, and reviewed its output at each
step rather than accepting it blind.

## What I decided myself

The architecture calls were mine going in, stated as constraints before any
code was written: NestJS over plain Express, four separate repos over a
monorepo (client/admin/backend/infra), a modular monolith rather than
microservices, the repository pattern for persistence, JWT access-in-memory
plus httpOnly-refresh-cookie with rotation and reuse detection specifically
because I wanted the "highest security" tier rather than a single long-lived
token, a separate admin frontend rather than an `/admin` route bundled into
the storefront, dynamic RBAC (roles as data, not a hardcoded `isAdmin`
flag), trunk-based git flow with a commit after every finished unit of work,
and a flat, no-gradient visual design specifically so the UI wouldn't read
as visually generic/AI-templated.

Beyond the initial architecture, most of the substantive product direction
came from me reviewing what was built and pushing back or asking for more:
pointing out that login had no rate limiting and product edits had no
concurrency protection; asking for the admin and client portals to be
visually redesigned and for CRUD to move into modal dialogs; asking for
pagination/search/filters on product, order, and user listings; asking for
sign-up and admin-side user creation, since neither existed; and — the one
I pushed hardest on — noticing that a role limited to `products:write` could
still browse products and place orders on the storefront, which led to
tracing that permissions were checked on nothing storefront-side and that
`products:read` was defined but enforced nowhere, and asking for the
permission model to be rebuilt so every role's permission list actually
describes what that role can do, and for the frontend to hide UI a role
can't use instead of showing it and then failing.

## How I verified the output

Every change went through: TypeScript typecheck and lint (zero warnings
tolerated, not just zero errors), the unit test suites (Jest on the
backend, Vitest on both frontends), and the Playwright e2e suites against
the real containerized stack — not mocked. I had Claude rebuild the Docker
images and re-run both e2e suites after any change that touched a shared
contract (API response shapes, permission gates) before calling it done,
and I looked at real screenshots of the running app rather than trusting
that a passing test meant it looked or worked right. When I saw something
odd in a screenshot — a role with permission gaps still seeing pages it
shouldn't, an error page showing a canned message instead of the real
backend error — I asked about it and had Claude explain what was actually
happening before agreeing to a fix, rather than accepting the first
explanation.

## What I changed or rejected

- The permission model, as originally built, left several storefront actions
  (browse, cart, wishlist, checkout, view your own orders) ungated behind
  any permission at all, and shipped one permission (`products:read`) that
  no route actually checked. I rejected that as incomplete and had it
  rebuilt so every one of those actions maps to a real, enforced permission,
  and the customer role's permission list is an accurate description of
  what a customer account can do.
- I rejected having the frontend show navigation links or buttons for
  actions a role can't perform and only finding out via a failed request —
  I asked for the UI itself to reflect the permissions returned at login, the
  same way the admin portal already did, instead of that logic existing in
  only one of the two frontends.
- An early version of the checkout stock-compensation logic restored *all*
  requested stock on failure rather than only what had actually been
  claimed — Claude's own test caught this before I did, but I confirmed the
  fix and the reasoning behind it (restoring unclaimed stock would invent
  inventory that was never taken) before accepting it.
- I asked for the login-throttling and optimistic-concurrency gaps to be
  closed rather than accepting the initial "highest security" auth
  implementation as sufficient — those two specific gaps were mine to catch,
  not something flagged for me.
- I declined to have these four rationale documents (this one included)
  written and left as AI output — I asked Claude to give me the facts and
  trade-offs so I could write the explanation in my own words, on the
  reasoning that a document about how I used AI, or why I made an
  architecture decision, is exactly the kind of writing where the source
  should be me.
