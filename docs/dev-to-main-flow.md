# Dev-to-Main Workflow

## The flow

```text
feature/*  or  fix/*
        \      /
         dev
          |  (PR)
         main
          |
      production
```

All work starts from `dev`. Code reaches `main` only through a pull request. `main` always represents what's in production.

## Day-to-day workflow

### 1. Create a branch from `dev`

```bash
git checkout dev
git pull origin dev
git checkout -b feature/my-feature
```

Use `feature/` for new work and `fix/` for bug fixes.

### 2. Work and commit

Make your changes and commit. Use [signed commits](verified-commits.md) so they show as "Verified" on GitHub.

```bash
git add -A
git commit -m "add login page"
```

### 3. Open a PR to `dev`

Push your branch and open a pull request targeting `dev`. The [PR template](../.github/pull_request_template.md) will guide you through describing your change.

```bash
git push -u origin feature/my-feature
```

CI runs automatically. The PR can't merge until the **Lint & Test** check passes. See [docs/branch-protection.md](branch-protection.md) for how this is enforced.

### 4. Merge to `dev`

Once CI passes and the PR is approved, **squash merge** into `dev`. Squash merging collapses your branch into a single commit, keeping the `dev` history clean.

If you're working solo, you can merge your own PR after CI passes. The branch protection rules still enforce that CI must be green.

### 5. Promote `dev` to `main`

When `dev` is stable and ready for production, open a PR from `dev` to `main`. Use a **merge commit** (not squash) for this PR. A merge commit preserves the boundary between releases and makes it easy to see what went into each production deploy.

### 6. Deploy

After merging to `main`, deploy. How you deploy is up to you — this template doesn't prescribe a deployment method.

## Hotfix flow

Sometimes you need to fix production without going through `dev`.

1. Branch from `main`:

   ```bash
   git checkout main
   git pull origin main
   git checkout -b fix/critical-bug
   ```

2. Fix the issue and push.

3. Open a PR directly to `main`. Get it reviewed, wait for CI, and merge.

4. After merging the hotfix to `main`, merge `main` back into `dev` so `dev` picks up the fix:

   ```bash
   git checkout dev
   git pull origin dev
   git merge origin/main
   git push origin dev
   ```

   This prevents `dev` from diverging from production.

## Merge strategy summary

| PR direction | Merge strategy | Why |
|---|---|---|
| `feature/*` or `fix/*` to `dev` | Squash merge | Clean history, one commit per feature |
| `dev` to `main` | Merge commit | Preserves release boundary |
| Hotfix to `main` | Squash merge | Single clean fix |

## Solo developers

This workflow works fine for one person. You write the code, open the PR, wait for CI, and merge it yourself. The value isn't peer review — it's the forcing function. A PR gives you a moment to read your own diff, confirm CI passes, and make the merge an intentional action instead of a raw push.
