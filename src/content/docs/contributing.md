---
title: Contributing
description: How to set up the Flux API repo, branch, and open a PR that lands cleanly.
---

Contributions are welcome. This page mirrors the canonical
[`CONTRIBUTING.md`](https://github.com/PedroL3m0z/Flux-Api/blob/main/.github/CONTRIBUTING.md)
and [`BRANCHING.md`](https://github.com/PedroL3m0z/Flux-Api/blob/main/.github/BRANCHING.md)
in the API repo — read those for the full detail; this is the quick orientation.

## Set up the dev environment

Run the infra in Docker and the app on the host with hot-reload:

```bash
git clone https://github.com/PedroL3m0z/Flux-Api.git
cd Flux-Api
cp .env.example .env          # optional — the app boots zero-config
yarn install
yarn prisma:generate
docker compose -f docker-compose.dev.yml up -d   # Postgres + Redis only
yarn start:dev
```

The API comes up on `http://localhost:3000`. For the running gateway itself,
see [Getting started](/flux-docs/getting-started/).

## Workflow

Flux uses **GitHub Flow** (trunk-based): one protected `main`, short-lived
feature branches, squash-merged PRs. There is **no** `develop` branch.

1. Branch from the latest `main` — never from another feature branch:
   ```bash
   git checkout main && git pull origin main
   git checkout -b feat/my-feature
   ```
2. Keep each PR focused — **one concern per branch**, small and reviewable.
3. Make sure it builds, lints, and tests pass (below).
4. Open a PR **into `main`** with the template; link related issues.
5. After merge, **delete the branch**. Close any superseded/duplicate PRs.

:::caution[Two things that bite]
**Don't** stack PRs (base = another `feat/*` branch), and **don't** merge two
PRs for the same work — close the stale one first. Both confuse the
release-please automation.
:::

## Branch naming

Use Conventional Commit prefixes as the branch name:

| Prefix | For |
| --- | --- |
| `feat/<desc>` | new functionality |
| `fix/<desc>` | bug fix |
| `chore/<desc>` | tooling, deps, CI |
| `docs/<desc>` | documentation only |
| `refactor/<desc>` | internal change, no behaviour difference |

Examples: `feat/zero-config`, `fix/auth-register-api-key`. Don't reuse a branch
name after its PR merged — cut a fresh one.

## Quality gates

These run in CI on every PR; run them locally first:

```bash
yarn lint        # ESLint (--fix)
yarn build       # tsc / nest build
yarn test        # unit tests
yarn test:e2e    # e2e tests
```

Rebase on `main` before opening or updating a PR so CI runs on a clean diff:

```bash
git fetch origin
git rebase origin/main
git push --force-with-lease
```

## Commit messages

Follow [Conventional Commits](https://www.conventionalcommits.org/) — they drive
automated versioning and the changelog:

```
feat(auth): add API key authentication strategy
fix(prisma): close pool on shutdown
docs(readme): document /health endpoint
```

Enable the shared template locally:

```bash
git config commit.template .github/.gitmessage.txt
```

## Database changes

The schema lives in `src/core/prisma/schema.prisma`. After editing it:

```bash
yarn prisma:migrate     # creates a migration + regenerates the client
```

Commit the generated migration files. Do **not** commit
`src/core/prisma/generated`.

## Releases

Releases are automated with **release-please** — don't hand-edit version numbers
or `CHANGELOG.md`. Merged `feat:` / `fix:` commits accumulate into a release PR
(`chore(main): release X.Y.Z`); merging that PR tags the version and cuts the
GitHub Release.

## Reporting bugs & security

Use the GitHub issue templates for bugs and feature requests. For **security**
issues, follow
[`SECURITY.md`](https://github.com/PedroL3m0z/Flux-Api/blob/main/.github/SECURITY.md)
— do not open a public issue. The project follows the
[Contributor Covenant](https://github.com/PedroL3m0z/Flux-Api/blob/main/.github/CODE_OF_CONDUCT.md).
