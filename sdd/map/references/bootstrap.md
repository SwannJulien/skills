# Bootstrapping `.claude/map/conventions.md`

Runs once per project, the first time `/map` is used. It decides **which maps
exist**, **what each one is derived from**, and **which commands produce it
mechanically**. Every later run reads this file and asks nothing.

Get it right once: the source globs are what Stage 1 uses to decide whether a map
needs regenerating, so a glob that is too narrow means a map that silently goes
stale, and one that is too wide means paying for a full re-read on every PR.

## Detect first, ask last

| Field | Where to look |
|---|---|
| Project shape | root layout, `package.json` workspaces, `pnpm-workspace.yaml`, `turbo.json`, `settings.gradle`, `Cargo.toml` workspace, `go.work` |
| Database | ORM schema (`schema.prisma`, `models.py`, `*.entity.ts`, JPA entities), a migrations directory, `docker-compose.yml` |
| Schema dump command | the DB container name in `docker-compose.yml`, the ORM's own introspection (`prisma migrate diff`, `alembic`, `mvn flyway:info`) |
| API surface | a routes directory, controller annotations, an OpenAPI/Swagger document, a tRPC router, a GraphQL schema |
| Route listing command | `rails routes`, `php artisan route:list`, `nest`'s bootstrap log, an existing `openapi.json` generation script |
| Front end | a components directory, a router config, file-system routing conventions (`app/`, `pages/`), a state library (`store/`, context providers) |
| Deployment | `.github/workflows/`, `.gitlab-ci.yml`, `Dockerfile`, `docker-compose*.yml`, `terraform/`, `helm/`, `fly.toml`, `vercel.json`, `*.env.example` |
| Doc language | existing docs under `docs/`, the `README.md`, `CLAUDE.md` — projects usually state it |
| Where docs live | an existing `docs/` tree; otherwise propose `docs/map/` |

Then ask the user only about:

- **Maps you are unsure are worth having.** A project with three background-job
  systems may want a `jobs` map; one with a thin API may not want `api` separate
  from `back`. Propose, don't interrogate.
- **Any introspection command that touches a live system** — a database that must
  be running, a cloud CLI that needs credentials. Never run one of these without
  asking, and record whether it is allowed at all.
- **Nothing else.** Everything in the table above is discoverable.

## Choosing the map set

Start from `db`, `api`, `front`, `deploy` and then **correct it against the repo**,
because a fixed four is wrong more often than it is right:

- **No front end in this repo?** Drop `front`. `api`'s `Called from` becomes
  `external` everywhere, which is the honest answer.
- **A monorepo with several services?** One `api` map per deployable, not one
  merged map. `docs/map/api-billing.md`, `docs/map/api-identity.md`. Merged maps
  across deployables hide the boundary that matters most.
- **Mobile or desktop client?** `front` becomes `client`, with screens instead of
  routes. Same template, renamed columns.
- **Event-driven or queue-heavy?** Add `events` — topics, producers, consumers,
  payload shape. This is a seam map, and it is usually the one a team has never
  written down.
- **Scheduled work?** Add `jobs` — schedule, entry point, what it writes, whether
  it is idempotent.
- **No infrastructure in the repo?** Keep `deploy` but say so in its `Gaps`: the
  pipeline is elsewhere, and the map covers only configuration consumed here.

Use `templates/generic.md` for any map outside the four with their own template.

Budget: **four to six maps.** Past that nobody reads them, and the per-PR cost
stops being worth the signal. If a project seems to need eight, two of them are
probably one map with a grouping.

## File to write

```markdown
# Map conventions

Used by the `map` skill. Edit freely — these are read, not enforced.

## Output

- **Maps root**: `docs/map/`
- **Language**: <language the maps are written in>
- **Split threshold**: 250 rows — past that, a map becomes a directory
  grouped by <the grouping this project uses: domain, service, bounded context>

## Maps

| Map | Template | Sources (globs) | Mechanical source |
|---|---|---|---|
| db | `db.md` | `prisma/schema.prisma`, `prisma/migrations/**` | `docker compose exec -T db pg_dump -s -U app app` |
| api | `api.md` | `apps/server/src/routes/**`, `apps/server/src/middleware/**` | none — read the route files |
| front | `front.md` | `apps/web/src/**` | none |
| deploy | `deploy.md` | `.github/workflows/**`, `docker-compose*.yml`, `Dockerfile*`, `*.env.example` | none |

## Seam ownership

<Which map owns each cross-layer relationship, if it differs from the skill's
default. State explicitly when a layer's callers live outside this repository.>

## Diagrams

One per map, capped at ~15 nodes, falling back to a table when it would exceed
the cap:

| Map | Diagram |
|---|---|
| db | foreign-key graph, clustered by <domain> |
| api | middleware and auth chain |
| front | route → page → endpoint data flow for <the project's central route> |
| deploy | commit → CI → image → environment |

## Commands that need permission

<Anything that starts a service, needs credentials, or talks to a live
environment. The skill asks before running these.>

## Not covered

<Parts of the system deliberately outside the maps, and where they are
documented instead. Prevents a reader assuming the maps are complete.>
```

## After writing

Show the user: the map set with the sources of each, the commands you intend to
run, anything you had to guess, and what you decided **not** to map. Get
confirmation before generating. Then generate.
