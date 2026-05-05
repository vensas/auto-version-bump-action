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

## Outputs

| Output | Description |
|---|---|
| `has_changes` | `true` if at least one version was bumped, otherwise `false` |
| `updates` | Comma-separated list of updates, e.g. `src/frontend: 1.2.0 -> 1.3.0 (minor)` |

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