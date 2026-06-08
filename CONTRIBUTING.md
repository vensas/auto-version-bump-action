# Contributing to Auto Version Bump Action

Thank you for your interest in contributing! This document provides guidelines and instructions for contributing.

## Code of Conduct

Please be respectful and constructive in all interactions. We welcome contributors of all experience levels.

## How to Contribute

### Reporting Bugs

1. Check existing issues to avoid duplicates
2. Use a clear, descriptive title
3. Include steps to reproduce the issue
4. Provide your environment details (OS, Node.js version, etc.)
5. Include relevant logs or error messages

### Suggesting Features

1. Check existing issues and discussions first
2. Describe the use case and expected behavior
3. Explain why this would benefit other users

### Pull Requests

1. Fork the repository
2. Create a feature branch from `main`:
   ```bash
   git checkout -b feat/your-feature-name
   ```
3. Make your changes
4. Test your changes locally
5. Commit using [conventional commits](https://www.conventionalcommits.org/):
   - `feat:` for new features
   - `fix:` for bug fixes
   - `docs:` for documentation changes
   - `refactor:` for code refactoring
   - `test:` for adding tests
   - `chore:` for maintenance tasks
6. Push to your fork and open a pull request
7. Fill out the pull request template with relevant details

## Development Setup

1. Clone your fork:
   ```bash
   git clone https://github.com/YOUR_USERNAME/auto-version-bump-action.git
   cd auto-version-bump-action
   ```

2. The action is a composite action with an inline Node.js script in `action.yml`. No build step is required.

3. To test locally, create a test repository with a `CHANGELOG.md` and version file, then run the inline script.

## Adding Support for New Languages

The most impactful contribution is adding support for additional version file formats. Currently supported:

| Language / Platform | Version file | Version tag |
|---|---|---|
| JavaScript / Node.js | `package.json` | `version` field |
| C# (.NET) | `*.csproj` | `<Version>` or `<ApplicationDisplayVersion>` |

To add a new format, modify these functions in `action.yml`:

1. **`findVersionFile`** - Recognize the new file type (e.g., `pyproject.toml`, `Cargo.toml`)
2. **`getVersion`** - Extract the version string from the file format
3. **`setVersion`** - Write the bumped version back to the file

### Example: Adding Python Support

```javascript
// In findVersionFile:
const pyproject = path.join(dir, 'pyproject.toml');
if (fs.existsSync(pyproject)) return { path: pyproject, type: 'python' };

// In getVersion:
if (type === 'python') {
  const match = content.match(/version\s*=\s*["']([^"']+)["']/);
  return match ? match[1] : null;
}

// In setVersion:
if (type === 'python') {
  return content.replace(
    /version\s*=\s*["'][^"']+["']/,
    `version = "${version}"`
  );
}
```

## Testing Your Changes

1. Create a test directory with sample files:
   ```
   test-project/
   ├── CHANGELOG.md
   └── package.json (or your target version file)
   ```

2. Add a version marker to the changelog:
   ```markdown
   ## [Unreleased]
   <!-- Version: patch -->

   ### Fixed
   - Test fix
   ```

3. Run the action locally or in a test workflow

## Style Guidelines

- Keep the inline script readable and well-commented
- Follow existing code patterns
- Use clear variable names
- Handle errors gracefully with informative messages

## Questions?

Open an issue with your question and we'll be happy to help.

## License

By contributing, you agree that your contributions will be licensed under the same license as the project (see [LICENSE](LICENSE)).
