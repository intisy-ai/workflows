# workflows

Shared **reusable** GitHub Actions workflows for the intisy-ai ecosystem.

## Branching & release standard

Every consuming repo follows a two-stage, tag-gated promotion flow:

| You push | What happens |
| --- | --- |
| any branch (`feature/*`, `experimental`) | tests run (repo CI); nothing is merged |
| a **non-`v` tag** (e.g. on `feature/*`) | tagged commit is merged into **`experimental`** (`merge-to-experimental.yml`) |
| a **`v*` tag** (cut from `experimental`) | tagged commit is merged into **`main`** (`merge-to-main.yml`) **and** the repo's own build/publish runs |

Lifecycle: work on `feature/x` -> push a non-`v` tag to land it on `experimental` -> push a `vX.Y.Z` tag to release to `main`. Plain branch pushes never merge.

## Consuming these workflows

Add two thin caller workflows to a repo under `.github/workflows/`:

```yaml
# promote-experimental.yml
name: Promote to Experimental
on:
  push:
    tags: ['*', '!v*']
permissions:
  contents: write
jobs:
  promote:
    uses: intisy-ai/workflows/.github/workflows/merge-to-experimental.yml@main
```

```yaml
# promote-main.yml
name: Promote to Main
on:
  push:
    tags: ['v*']
permissions:
  contents: write
jobs:
  promote:
    uses: intisy-ai/workflows/.github/workflows/merge-to-main.yml@main
```

The merge workflows use the built-in `GITHUB_TOKEN` (org branches are unprotected), so no `PAT` is required. Each repo's publish/README workflow should trigger on `tags: ['v*']` (not `['*']`) so non-`v` feature tags don't cut a release.
