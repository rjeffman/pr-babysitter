pr-babysitter
=============

A CLI tool to monitor and restart failed checks in Github pull requests.

## Installation

YouYou can install `pr-babysitter` by downloading the script using `curl` and placing it in a directory within your `$HOME` path, such as `~/bin`.

### From a release tag

To install a specific version from a release tag (e.g., `v1.0.0`), use the following command:

```bash
mkdir -p "$HOME/bin"
curl -sL "https://github.com/rjeffman/pr-babysitter/releases/download/v1.0.0/pr-babysitter" -o "$HOME/bin/pr-babysitter"
chmod +x "$HOME/bin/pr-babysitter"
```

Replace `v1.0.0` with the desired release tag.

### From the main branch

To install the latest version directly from the `main` branch, use:

```bash
mkdir -p "$HOME/bin"
curl -sL "https://raw.githubusercontent.com/rjeffman/pr-babysitter/main/pr-babysitter" -o "$HOME/bin/pr-babysitter"
chmod +x "$HOME/bin/pr-babysitter"
```

Ensure that `~/bin` is in your system's `PATH` environment variable. If not, you might need to add it to your shell's configuration file (e.g., `.bashrc`, `.zshrc`) like so: `export PATH="$HOME/bin:$PATH"`.

## Dependencies

### Required

- **curl** - HTTP client for making API requests
- **jq** - Command-line JSON processor
- **git** - Version control system (for automatic repository detection)
- **fold** - Text folding utility (typically pre-installed on Unix systems)

The script requires a GitHub personal access token (see Authentication section below).

### Optional

- **notify-send** - Desktop notification utility (for failure notifications)

## Usage

```
pr-babysitter [OPTIONS] <PR_NUMBER> [REPO]
pr-babysitter [OPTIONS] -c CHECK_ID [REPO]
pr-babysitter [OPTIONS] -W WORKFLOW_ID [REPO]
pr-babysitter [OPTIONS] -m REPO <PR_NUMBER> [PR_NUMBER...]
```

Monitor the status of checks for a GitHub PR, or directly re-run checks/workflows. Can also monitor multiple PRs simultaneously.

Desktop notifications will be sent when checks fail (if available).

## Configuration

The script will source configuration from the first file found:
- `${HOME}/.pr-babysitter`
- `${HOME}/.config/pr-babysitter`

### Authentication

The script requires a GitHub personal access token with `repo` scope set in either `GITHUB_TOKEN` or `GH_TOKEN` environment variable.

You can set the token by:
- Exporting it in your shell:
  ```bash
  export GITHUB_TOKEN="your_token_here"
  ```
- Adding it to your configuration file (`~/.pr-babysitter` or `~/.config/pr-babysitter`):
  ```bash
  GITHUB_TOKEN="your_token_here"
  ```

## Options

- `-f` - Only show failed checks, do not monitor
- `-w SECONDS` - Wait time in seconds between checks (default: 300)
- `-m REPO` - Monitor multiple PRs for the specified repository
- `-r MODE` - Re-run mode for failed checks:
  - `ask` - Prompt for re-run option (default)
  - `check` - Re-run individual check runs
  - `workflow` - Re-run entire workflow runs
  - `failed-jobs` - Re-run only failed jobs in workflows
  - `label:NAME` - Add specified label NAME to the PR
- `-c CHECK_ID` - Directly re-run a specific check run by ID
- `-W WORKFLOW_ID` - Directly re-run a specific workflow run by ID
- `-v` - Enable verbose mode
- `-h` - Show this help message

## Arguments

- `PR_NUMBER` - The pull request number to monitor
- `REPO` - Optional repository in owner/repo format (defaults to current repo)

## Examples

### PR Monitoring

```bash
pr-babysitter 123                          # Monitor PR #123 in current repo
pr-babysitter 123 owner/repo               # Monitor PR #123 in specified repo
pr-babysitter -f 123                       # Show only failed checks
pr-babysitter -w 300 123                   # Monitor with 5 minute interval
pr-babysitter -r workflow 123              # Auto re-run entire workflows
pr-babysitter -r failed-jobs -f 123        # Re-run failed jobs, show failed only
pr-babysitter -r label:re-run 123          # Add 're-run' label to PR #12
```

### Multi-PR Monitoring

```bash
pr-babysitter -m owner/repo 123 456 789    # Monitor PRs 123, 456, 789
pr-babysitter -w 60 -m owner/repo 123 456  # Monitor with 1-minute interval
```

### Direct Re-run

```bash
pr-babysitter -c 61523939520 owner/repo    # Re-run specific check run
pr-babysitter -W 21373540416 owner/repo    # Re-run specific workflow run
```