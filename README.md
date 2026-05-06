# auto-version-bump-action

A GitHub composite action that automatically bumps versions in `package.json` or `.csproj` files based on a `<!-- Version: TYPE -->` comment in a `CHANGELOG.md` `[Unreleased]` section.

It discovers all changelogs in the repository automatically — no configuration needed.

## How it works

1. Recursively finds every `CHANGELOG.md` in the repository.
2. Checks whether the `[Unreleased]` section contains a comment like:
   ```markdown
   <!-- Version: minor -->
   ```
3. Finds the `package.json` or `.csproj` in the same directory and bumps its version accordingly.
4. Moves the `[Unreleased]` content to a new versioned section with today's date and removes the comment.

Supported bump types: `major`, `minor`, `patch`.  
Supported version files: `package.json`, any `.csproj` using `<Version>` or `<ApplicationDisplayVersion>`.

## Usage

```yaml
name: Auto Version Bump

on:
  push:
    branches:
      - main
    paths:
      - '**/CHANGELOG.md'

permissions:
  contents: write

jobs:
  version-bump:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Configure Git
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

      - name: Bump versions
        id: version-bump
        uses: vensas/auto-version-bump-action@main

      - name: Commit and push changes
        if: steps.version-bump.outputs.has_changes == 'true'
        run: |
          git add -A
          git commit -m "chore: bump versions after merge [skip ci]

          ${{ steps.version-bump.outputs.updates }}"
          git push origin main
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `auto-commit` | No | `false` | Commit and push the version bump changes automatically. Requires `permissions: contents: write`. |
| `commit-message` | No | `chore: bump versions [skip ci]\n\n{updates}` | Commit message when `auto-commit` is enabled. `{updates}` is replaced with the list of version changes. |
| `post-bump-command` | No | _(none)_ | Shell command to run after bumping but before committing. Use this to regenerate lockfiles. Only runs when `auto-commit` is enabled and changes were made. |
| `git-user-name` | No | `github-actions[bot]` | Git `user.name` for the auto-commit. |
| `git-user-email` | No | `github-actions[bot]@users.noreply.github.com` | Git `user.email` for the auto-commit. |
| `version-files` | No | `{}` | JSON map of changelog path to version file path (both repo-relative). Overrides auto-discovery for specific entries. |

## Outputs

| Output | Description |
|---|---|
| `has_changes` | `true` if at least one version was bumped, otherwise `false` |
| `updates` | Comma-separated list of updates, e.g. `src/frontend: 1.2.0 -> 1.3.0 (minor)` |

## Auto-commit

To have the action commit and push changes itself, enable `auto-commit`. This removes the need for a separate commit step in your workflow:

```yaml
- name: Bump versions
  uses: vensas/auto-version-bump-action@main
  with:
    auto-commit: 'true'
```

If your project uses a lockfile that needs to be regenerated after a version bump (e.g. `package-lock.json`, `pnpm-lock.yaml`), use `post-bump-command` to handle that before the commit:

```yaml
- name: Bump versions
  uses: vensas/auto-version-bump-action@main
  with:
    auto-commit: 'true'
    post-bump-command: 'cd src/frontend && pnpm install --lockfile-only'
```

> **Note:** The default commit message includes `[skip ci]` to prevent the workflow from triggering itself again when the updated `CHANGELOG.md` is pushed back to the branch.

## Overriding version file discovery

If a changelog is not in the same directory as its version file, use `version-files` to map them explicitly:

```yaml
- name: Bump versions
  uses: vensas/auto-version-bump-action@main
  with:
    version-files: '{"src/backend/Feps/CHANGELOG.md": "src/backend/Feps/Feps.Api/Feps.Api.csproj"}'
```

Auto-discovery still runs for all other changelogs — only the mapped entries are overridden.

## Changelog format

```markdown
## [Unreleased]
<!-- Version: minor -->

### Added
- Some new feature
```

After the action runs, this becomes:

```markdown
## [Unreleased]

## [1.3.0] - 2026-05-05

### Added
- Some new feature
```