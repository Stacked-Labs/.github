# Contributing & Branching Workflow

This is the **org-wide default** workflow for Stacked-Labs repos. GitHub shows
it on every repo that doesn't ship its own `CONTRIBUTING.md`. A repo with its
own copy (e.g. `poker-game`) overrides this one.

## The model: promotion, not sorting

```
feature branch  ──PR──▶  integration  ──PR──▶  main
  (your work)            (staging)              (production)
```

- **`main`** is production.
- The **integration branch** is staging. It's called **`dev`** in `poker-game`
  and **`develop`** in the backend repos (`poker-server`, `poker-indexer`,
  `poker-points`). Wherever this doc says "`dev`", use your repo's name.
- **The one rule:** `main` never receives a change that hasn't already lived in
  the integration branch. Code only moves *up* the pipeline. That keeps the
  integration branch a true superset of `main`, so every release is a non-event.

Don't sort changes into branches by risk ("this is just cosmetic, push it to
main"). **Every change takes the same route.** Risk is managed by review, CI,
and the staging environment — not by which branch you pick. Sorting by risk is
exactly what makes `main` and `dev` drift apart.

> **Single-branch repos** (`poker-docs`, `poker-recorder`, `stacked-roulette`)
> have only `main` — there's no staging branch, so PRs go straight into `main`.
> Everything below about promotion/hotfixes only applies to repos with an
> integration branch.

## Day-to-day: shipping a change

1. **Branch from the integration branch:**
   ```bash
   git checkout dev && git pull          # or: develop
   git checkout -b feat/short-description # or fix/… or chore/…
   ```
2. Do the work; commit in logical chunks.
3. **Open a PR into the integration branch.** Get **1 approving review** and a
   **green CI build** (where CI exists).
4. Merge (squash or merge-commit is fine for feature → integration).
5. Verify on staging.

You almost never touch `main` directly.

## Releasing: promoting the integration branch → `main`

1. Open a PR: **base `main`, head `dev`/`develop`**, e.g. `Release: 2026-06-16`.
2. Review the diff — it's everything already verified on staging. No surprises.
3. **Merge with a real merge commit — never squash** (see below).
4. Deploy + verify on production, then **tag the release**. For releases that
   span multiple repos (e.g. frontend + backend), use the **same tag in each**
   so a production state is reproducible across the stack.

## ⚠️ Never squash a promotion or reconciliation merge

Feature → integration: squashing is fine. But **integration → `main`** (and any
**`main` → integration** catch-up) **must** use *"Create a merge commit."*
Squashing collapses history into a new commit, so git stops seeing that the
branches share ancestry — they "forget" they're related and the next promotion
re-surfaces every old conflict. A merge commit preserves ancestry and keeps the
integration branch a true superset of `main`.

## Hotfixes (the only sanctioned reason to touch `main` directly)

1. **Branch from `main`:** `git checkout -b hotfix/the-problem main`
2. Fix, PR into `main`, review, merge, deploy.
3. **Immediately back-merge `main` into the integration branch** so they don't
   drift. This step is not optional. Better still, prefer **feature flags** so
   the integration branch stays releasable and hotfixes are rare.

## Branch protection

Public repos (`poker-game`, `poker-docs`) enforce this via GitHub Rulesets:
PR required, 1 approving review, stale-review dismissal, conversation
resolution, no force-push/deletion, and `main` restricted to merge-commits only
(plus a required build check where CI exists).

Private repos currently **rely on this document and team discipline** —
branch protection isn't available on the org's plan for private repos. Treat the
rules here as binding even though they aren't yet machine-enforced there.

## Branch naming

| Prefix     | Use for                              | Branch from        |
|------------|--------------------------------------|--------------------|
| `feat/`    | New feature or enhancement           | integration        |
| `fix/`     | Bug fix (non-urgent)                 | integration        |
| `chore/`   | Tooling, deps, docs, refactors       | integration        |
| `hotfix/`  | Urgent production fix                | `main`             |
| `release/` | Optional staging branch for a release| integration        |
