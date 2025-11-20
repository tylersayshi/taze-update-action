# Taze Update Action

Automatically update dependencies using [taze](https://github.com/antfu/taze) and create a pull request with the changes.

## Features

- Updates dependencies using taze
- Automatically runs `pnpm install` after updates
- Creates a pull request with the changes
- Customizable branch names, PR titles, and labels
- Skips PR creation if no updates are available

## Usage

```yaml
name: Update Dependencies

on:
  schedule:
    - cron: '0 0 * * 0' # Run weekly on Sunday at midnight
  workflow_dispatch: # Allow manual trigger

jobs:
  update-deps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Update dependencies
        uses: YOUR_USERNAME/taze-update-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `taze-input` | Flags or args to pass to taze. We recommend low concurrency with GitHub runners | No | `-w --concurrency 1` |
| `branch-prefix` | Prefix for the branch name | No | `chore/update-deps` |
| `pr-title` | Title for the pull request | No | `chore: update dependencies` |
| `pr-labels` | Comma-separated labels to add to the PR | No | `dependencies` |
| `github-token` | GitHub token for creating PRs | Yes | - |

## Examples

### Update dependencies with custom options

```yaml
- uses: YOUR_USERNAME/taze-update-action@v1
  with:
    taze-input: '-w --concurrency 2'
    branch-prefix: 'deps/update'
    pr-title: 'Update dependencies'
    pr-labels: 'dependencies,automated'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Update only minor versions

```yaml
- uses: YOUR_USERNAME/taze-update-action@v1
  with:
    taze-input: 'minor --concurrency 1'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Update only specific packages

```yaml
- uses: YOUR_USERNAME/taze-update-action@v1
  with:
    taze-input: '-w --include react,vue --concurrency 1'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Requirements

- Your repository must use pnpm as the package manager
- The workflow must have `actions/checkout@v4` run before this action
- The workflow must have `pnpm/action-setup` configured
- The workflow must have `actions/setup-node` configured with pnpm cache

## How it works

1. Installs taze globally using pnpm
2. Runs taze with the specified arguments to update dependencies
3. Runs `pnpm install --no-frozen-lockfile` to update the lockfile
4. Creates a new branch with the updates
5. Commits the changes
6. Pushes the branch to the repository
7. Creates a pull request with the specified title and labels

If no dependency updates are available, the action exits gracefully without creating a PR.

## License

MIT
