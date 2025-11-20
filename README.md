# Taze Update Action

Automatically update dependencies using [taze](https://github.com/antfu/taze) and create a pull request with the changes. This assumes that the project is using `pnpm`.

## Features

- Updates dependencies using taze
- Automatically runs `pnpm install` after updates
- Creates a pull request with the changes
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
          node-version: '24'
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

- Your project must use pnpm (I'd consider PRs to extend this somehow)
- A Personal Access Token (PAT) with the following repository permissions:
  - Read access to metadata
  - Read and Write access to code, pull requests, and workflows

  The PAT is required instead of `GITHUB_TOKEN` to [trigger workflow runs](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow#:~:text=When%20you%20use%20the%20repository's%20GITHUB_TOKEN%20to%20perform%20tasks%2C%20events%20triggered%20by%20the%20GITHUB_TOKEN%2C%20with%20the%20exception%20of%20workflow_dispatch%20and%20repository_dispatch%2C%20will%20not%20create%20a%20new%20workflow%20run) on the created PR. Store it as a repository secret and use it in place of `secrets.GITHUB_TOKEN`.

## How it works

1. Installs taze globally using pnpm
2. Runs taze with the specified arguments to update dependencies
3. Runs `pnpm install --no-frozen-lockfile` to update the lockfile
4. Creates a new branch with the updates
5. Commits the changes
6. Pushes the branch to the repository
7. Creates a pull request with the specified title and labels

If no dependency updates are available, the action exits gracefully without creating a PR.
