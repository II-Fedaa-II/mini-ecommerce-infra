# Mini E-Commerce — Infra

Docker Compose orchestration that runs the whole Mini E-Commerce stack — MongoDB, the NestJS backend, the React client portal, and the React admin portal — with a single command.

## Related repos

| Repo | Role |
| --- | --- |
| [mini-ecommerce-backend](https://github.com/II-Fedaa-II/mini-ecommerce-backend) | NestJS + MongoDB API, shared by both frontends |
| [mini-ecommerce-client-portal](https://github.com/II-Fedaa-II/mini-ecommerce-client-portal) | Customer-facing storefront |
| [mini-ecommerce-admin-portal](https://github.com/II-Fedaa-II/mini-ecommerce-admin-portal) | Admin control panel (RBAC) |

## Prerequisites

- Docker Desktop (Compose v2)
- All repos cloned as **sibling folders** under one parent directory, because the
  compose file references them with relative build contexts (`../mini-ecommerce-backend`):

```
mini-ecommerce/
├── mini-ecommerce-backend/
├── mini-ecommerce-client-portal/
├── mini-ecommerce-admin-portal/
└── mini-ecommerce-infra/     ← run compose from here
```

## Running the stack

```bash
cp .env.example .env          # adjust secrets/ports if you like
docker compose up -d --build
```

Then seed the database on first run (idempotent — safe to re-run):

```bash
docker compose exec backend node dist/seed/seed.js
```

| Service | URL |
| --- | --- |
| Client portal | http://localhost:5173 |
| Admin portal | http://localhost:5174 |
| Backend API | http://localhost:4000 |
| MongoDB | mongodb://localhost:27017 |

**Demo accounts**

| Role | Email | Password |
| --- | --- | --- |
| Customer | `demo@mini-ecommerce.test` | `Password123!` |
| Admin | `admin@mini-ecommerce.test` | `Admin123!` |

## Useful commands

```bash
docker compose ps                 # service status
docker compose logs -f backend    # tail backend logs
docker compose down               # stop (keeps the mongo_data volume)
docker compose down -v            # stop and wipe the database
```

## Notes

- `JWT_ACCESS_SECRET` / `JWT_REFRESH_SECRET` in `.env.example` are development
  placeholders. Generate real secrets before running this anywhere but locally.
- The backend accepts credentialed CORS requests only from `CLIENT_ORIGIN` and
  `ADMIN_ORIGIN`, because the refresh token travels as an httpOnly cookie.
- `VITE_API_URL` is baked into the frontend bundles at **build** time, so changing it
  requires `docker compose up --build`, not just a restart.
- The admin portal is a **separate app**, not an `/admin` route inside the storefront.
  No admin code, routes, or permission strings ship in the public bundle, and the two
  frontends can be deployed and scaled independently.

## Scope note

Everything under the storefront (login, product listing/detail, cart, wishlist, checkout)
is the assessment's required scope. The **admin portal and RBAC are an intentional
addition** beyond the brief's page list, built after the required flows were complete
and tested.

Architecture rationale (why separate repos, why a modular monolith, why this data model)
is documented in each repo's own docs.
