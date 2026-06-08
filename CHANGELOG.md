# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-06-08

### Added

- CHANGELOG.md to track changes
- Release workflow that dogfoods the action itself
- package.json to track version

## [0.1.0] - 2026-05-06

### Added

- Auto-discover all `CHANGELOG.md` files in repository recursively
- Detect bump type from `<!-- Version: TYPE -->` comment in `[Unreleased]` section
- Support for `package.json` version bumping (Node.js/JavaScript)
- Support for `.csproj` version bumping (.NET/C#)
- Auto-commit and push changes with `auto-commit` input
- Configurable commit message with `{updates}` placeholder
- Configurable git user name and email for commits
- Post-bump command support via `post-bump-command` input
- Version file mapping via `version-files` JSON input for custom paths
- Outputs: `has_changes` and `updates` for downstream workflow steps

### Fixed

- Git reference handling for PR branches
- Push command targeting correct branch
- Commit credentials configuration
