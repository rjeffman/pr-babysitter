pr-babysitter
=============

A CLI tool to monitor and restart failed checks in GitHub pull requests and GitLab merge requests. Also supports monitoring checks on Gitea/Forgejo instances (Codeberg, self-hosted).

## Installation

You can install `pr-babysitter` by downloading the script using `curl` and placing it in a directory within your `$HOME` path, such as `~/bin`.

### From a release tag

To install a specific version from a release tag (e.g., `v1.1.0`), use the following command:

```bash
mkdir -p "$HOME/bin"
curl -sL "https://github.com/rjeffman/pr-babysitter/releases/download/v1.1.0/pr-babysitter" -o "$HOME/bin/pr-babysitter"
chmod +x "$HOME/bin/pr-babysitter"
```

Replace `v1.1.0` with the desired release tag.

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

**Note:** Authentication tokens are required for GitHub and GitLab. Gitea/Forgejo instances use public API endpoints.

### Optional

- **notify-send** - Desktop notification utility (for failure notifications)

## Usage

```
pr-babysitter [OPTIONS] <PR_NUMBER> [REPO]
pr-babysitter [OPTIONS] <REPO> <PR_NUMBER> [PR_NUMBER...]
pr-babysitter [OPTIONS] -c CHECK_ID [REPO]
pr-babysitter [OPTIONS] -W WORKFLOW_ID [REPO]
```

Monitor the status of checks for pull requests on GitHub, GitLab, Codeberg, or custom Gitea/Forgejo instances. For GitHub, can directly re-run failed checks/workflows. For GitLab, can restart failed pipelines. Supports monitoring multiple PRs/MRs simultaneously.

Desktop notifications will be sent when checks fail (if available).

## Supported Services

- **GitHub** (github.com) - Full support: monitoring and re-running checks/workflows
- **GitLab** (gitlab.com or self-hosted) - Full support: monitoring and restarting pipelines
  - Pipeline restart only works for pipelines in FAILURE state (not CANCELED or UNKNOWN)
  - Requires `api` scope token for restart, or `read_api` for monitoring only
- **Codeberg** (codeberg.org) - Monitoring only (no re-run support)
- **Custom Gitea/Forgejo** - Monitoring only (no re-run support)

## PR Status Indicators

### Color-Coded Display

The tool uses color-coded output for improved readability:

- **PR/MR titles**: Bold white
- **URLs**: Cyan
- **Labels**: Magenta
- **Author names**: Bright yellow
- **Commit authors**: Bright yellow (in multi-PR mode)

### Overall Status Glyphs

A status glyph appears between the PR number and title to show the overall check status:

- 🔴 **Red Circle** - Failed or canceled checks (no checks running)
- 🟡 **Yellow Circle** - Failed checks with checks still running
- ⏳ **Hourglass** - Checks running (no failures)
- ✅ **Green Checkmark** - All checks passed

**Display format:**
```
#123 🔴👀 Feature: Add new authentication system
Labels: enhancement, needs-review
https://github.com/owner/repo/pull/123
Last push: 2026-02-13 14:30:25 by @username
Author: John Doe (@johndoe)
```

### Eyes Emoji 👀

When monitoring a PR, an eyes emoji (👀) appears after the status glyph to indicate that the PR has received a recent push that may need review attention.

**The 👀 emoji appears when:**
- The last push to the PR is more recent than the last updated comment from someone other than the PR author
- This indicates that code has been updated since the last time a reviewer or collaborator commented on the PR

**The 👀 emoji does NOT appear when:**
- No non-author comments exist on the PR
- The most recent comment update from others is newer than the last push
- The PR author's own comments don't count (only comments from reviewers/collaborators matter)

This helps identify PRs where reviewers should take another look because new changes have been pushed since they last engaged with the PR.

### Check Status Emojis

The tool uses emoji indicators to show the status of individual CI/CD checks:

- ✅ **Success** - Check passed successfully
- 🔵 **In Progress** - Check is currently running
- 🧵 **Queued** - Check is waiting to start
- 🚨 **Failure** - Check failed
- 🚫 **Canceled** - Check was canceled
- ❓ **Unknown** - Check status is unknown or could not be determined

## Configuration

The script will source configuration from the first file found:
- `${HOME}/.pr-babysitter`
- `${HOME}/.config/pr-babysitter`

### Authentication

**For GitHub (required):**

A GitHub personal access token with `repo` scope is required. Set it in either `GITHUB_TOKEN` or `GH_TOKEN` environment variable.

You can set the token by:
- Exporting it in your shell:
  ```bash
  export GITHUB_TOKEN="your_token_here"
  ```
- Adding it to your configuration file (`~/.pr-babysitter` or `~/.config/pr-babysitter`):
  ```bash
  GITHUB_TOKEN="your_token_here"
  ```

**For GitLab (required):**

A GitLab personal access token is required. Set it in either `GITLAB_TOKEN` or `GITLAB_ACCESS_TOKEN` environment variable.

- For pipeline restart functionality: Token requires `api` scope (full API access)
- For monitoring only: Token can use `read_api` scope (read-only access)

You can add the token to your configuration file (`~/.pr-babysitter` or `~/.config/pr-babysitter`):
```bash
GITLAB_TOKEN="your_token_here"
```

**For Gitea/Forgejo (Codeberg, etc.):**

No authentication required. The script uses public API endpoints.

## Options

- `-f` - Only show failed checks, do not monitor
- `-w SECONDS` - Wait time in seconds between checks (default: 300)
- `-s SERVICE` - Remote service to use (default: github)
  - `github` - GitHub (github.com)
  - `gitlab` - GitLab (gitlab.com)
  - `codeberg` - Codeberg (codeberg.org)
  - `https://HOST` - Custom GitLab/Gitea/Forgejo instance URL
- `-r MODE` - Re-run mode for failed checks:
  - `ask` - Prompt for re-run option (default)
  - `check` - Re-run individual check runs (GitHub only)
  - `workflow` - Re-run entire workflow runs (GitHub only)
  - `failed-jobs` - Re-run only failed jobs in workflows (GitHub only)
  - `label:NAME` - Add specified label NAME to the PR (GitHub only)
  - `comment:TEXT` - Add comment TEXT to the PR/MR (GitHub and GitLab)
- `-c CHECK_ID` - Directly re-run a specific check run by ID (GitHub only)
- `-W WORKFLOW_ID` - Directly re-run a specific workflow run by ID (GitHub only)
- `-v` - Enable verbose mode
- `-V` - Show version and exit
- `-h` - Show this help message

## Arguments

- `PR_NUMBER` - The pull request number to monitor
- `REPO` - Repository in owner/repo format (defaults to current repo)
  - If the first argument is not a number, it's treated as the repository
  - Multiple PR numbers automatically enable multi-PR monitoring mode

## Examples

### PR Monitoring

```bash
# GitHub (default)
pr-babysitter 123                          # Monitor PR #123 in current repo
pr-babysitter 123 owner/repo               # Monitor PR #123 in specified repo
pr-babysitter -f 123                       # Show only failed checks
pr-babysitter -w 300 123                   # Monitor with 5 minute interval
pr-babysitter -r workflow 123              # Auto re-run entire workflows (GitHub only)
pr-babysitter -r failed-jobs -f 123        # Re-run failed jobs (GitHub only)
pr-babysitter -r label:re-run 123          # Add 're-run' label to PR (GitHub only)
pr-babysitter -r comment:"Please review" 123  # Add comment to PR (GitHub)

# GitLab
pr-babysitter -s gitlab 123 owner/repo     # Monitor MR on GitLab.com
pr-babysitter -s gitlab -r comment:"Restarting tests" 123 owner/repo  # Add comment to MR
pr-babysitter -s https://gitlab.example.com 123 owner/repo  # Self-hosted GitLab
# When monitoring completes with failures, GitLab offers interactive pipeline restart
# Note: Only pipelines in FAILURE state can be restarted

# Codeberg
pr-babysitter -s codeberg 123 owner/repo   # Monitor PR on Codeberg

# Custom Gitea/Forgejo instance
pr-babysitter -s https://git.example.com 123 owner/repo
```

### Multi-PR Monitoring

Multi-PR mode is automatically activated when you provide a repository followed by multiple PR numbers:

```bash
# GitHub
pr-babysitter owner/repo 123 456 789       # Monitor PRs 123, 456, 789
pr-babysitter -w 60 owner/repo 123 456     # Monitor with 1-minute interval

# GitLab
pr-babysitter -s gitlab owner/repo 123 456 789    # Monitor MRs on GitLab.com
pr-babysitter -s https://gitlab.example.com owner/repo 123 456  # Self-hosted GitLab
# Press 'R' to restart failed pipelines, 'D' to view MR details

# Codeberg
pr-babysitter -s codeberg owner/repo 123 456 789

# You can also use traditional syntax with one PR
pr-babysitter owner/repo 123               # Same as: pr-babysitter 123 owner/repo
```

### Direct Re-run (GitHub only)

```bash
pr-babysitter -c 61523939520 owner/repo    # Re-run specific check run
pr-babysitter -W 21373540416 owner/repo    # Re-run specific workflow run
```

## Interactive Restart Features

### GitHub

When monitoring completes and failures are detected, GitHub offers an interactive menu with options:
1. Re-run individual check runs (default)
2. Re-run entire workflow runs (all jobs)
3. Re-run only failed jobs in workflows
4. Add a label to re-run checks
5. Add a comment to the PR

In multi-PR mode, press `R` to enter restart mode for all PRs with failures.

### GitLab

When monitoring completes and failures are detected, GitLab offers interactive restart options:
- **Single MR mode**: Menu with options:
  1. Restart pipeline (cancel active and create new)
  2. Add a comment to the MR
- **Multi-MR mode**: Press `R` to access restart options for all MRs with failures

**Important notes for GitLab:**
- Only pipelines in **FAILURE** state can be restarted
- Pipelines in CANCELED or UNKNOWN state cannot be restarted
- The restart process:
  1. Cancels any active/running pipelines
  2. Creates a new pipeline for the MR's source branch

### Gitea/Forgejo

No automatic restart functionality available. These services do not support check re-runs via API.