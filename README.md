# GitHub Power Commands — Complete Guide

> A beginner-to-advanced reference for controlling **every major feature of GitHub** from the command line.  
> Every command is commented to explain *what* it does, *why* you would use it, and *how popular IDEs react* to it.  
> Use this as a learning resource and adapt any command to your own workflow.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Git Configuration](#2-git-configuration)
3. [Repository Management](#3-repository-management)
4. [Branching & Merging](#4-branching--merging)
5. [Staging, Committing & History](#5-staging-committing--history)
6. [Remote Operations — Push / Pull / Fetch](#6-remote-operations--push--pull--fetch)
7. [Pull Requests (GitHub CLI)](#7-pull-requests-github-cli)
8. [Issues (GitHub CLI)](#8-issues-github-cli)
9. [Releases & Tags](#9-releases--tags)
10. [GitHub Actions (GitHub CLI)](#10-github-actions-github-cli)
11. [Gists](#11-gists)
12. [Collaborators & Teams](#12-collaborators--teams)
13. [GitHub Packages](#13-github-packages)
14. [Security & Secrets](#14-security--secrets)
15. [GitHub Pages](#15-github-pages)
16. [Notifications & Dashboard](#16-notifications--dashboard)
17. [Advanced Git Techniques](#17-advanced-git-techniques)
18. [IDE Integration Notes](#18-ide-integration-notes)

---

## 1. Prerequisites

### Install Git

```bash
# macOS (using Homebrew)
brew install git

# Ubuntu / Debian
sudo apt update && sudo apt install git -y

# Windows (using winget)
winget install --id Git.Git -e --source winget
# Or download the installer from https://git-scm.com/download/win

# Verify installation — prints the installed version
git --version
```

### Install GitHub CLI (`gh`)

The **GitHub CLI** (`gh`) lets you control Pull Requests, Issues, Actions, Releases,
Gists, and more without leaving your terminal.

```bash
# macOS
brew install gh

# Ubuntu / Debian
sudo apt install gh -y
# (or follow https://cli.github.com/manual/installation for your distro)

# Windows
winget install --id GitHub.cli -e --source winget

# Verify installation
gh --version
```

### Authenticate with GitHub

```bash
# Opens a browser-based OAuth flow — logs you into github.com
# Required before any 'gh' command that touches the API
gh auth login

# Authenticate using a Personal Access Token (PAT) stored in an env var
# Useful for CI/CD pipelines where browser login is not possible
echo "$GITHUB_TOKEN" | gh auth login --with-token

# Show current authentication status and the scopes granted
gh auth status
```

---

## 2. Git Configuration

```bash
# Set your global display name — shown in every commit you make
git config --global user.name "Your Name"

# Set your global email — must match the email on your GitHub account
# for commits to be linked to your profile
git config --global user.email "you@example.com"

# Set VS Code as the default editor for commit messages, rebase instructions, etc.
# Requires the 'code' CLI to be in PATH.
# In VS Code: open Command Palette (Ctrl+Shift+P) and run
# "Shell Command: Install 'code' command in PATH" to enable this.
git config --global core.editor "code --wait"

# Other popular editor values:
#   "vim"            — Vim / Neovim
#   "nano"           — Nano (beginner-friendly)
#   "idea --wait"    — IntelliJ IDEA / JetBrains IDEs
#                      Requires the 'idea' command-line launcher to be configured.
#                      See: https://www.jetbrains.com/help/idea/working-with-the-ide-features-from-command-line.html
#   "xed"            — Eclipse (via command-line launcher)

# Set the default branch name to 'main' for all new repos
# GitHub uses 'main' by default since 2020
git config --global init.defaultBranch main

# Enable colorized output in the terminal (highly recommended)
git config --global color.ui auto

# Cache your credentials for 15 minutes after the first use
# Avoids entering your password on every push
git config --global credential.helper cache

# On macOS, store credentials in the system Keychain permanently
git config --global credential.helper osxkeychain

# On Windows, use the Git Credential Manager (use 'manager' for GCM v2+;
# older versions used 'manager-core')
git config --global credential.helper manager

# Set 'rebase' strategy when pulling — keeps history linear (no merge commits)
# Great for teams that prefer a clean, linear commit history
git config --global pull.rebase true

# Show all current global configuration values
git config --global --list
```

---

## 3. Repository Management

### Create and Clone Repositories

```bash
# Initialize a new local Git repository in the current directory
# Creates a hidden .git folder that tracks all version history
git init

# Initialize a bare repository (used for servers / remotes, no working tree)
git init --bare my-repo.git

# Clone a public or private repository from GitHub to your machine
# Creates a new folder named after the repo
git clone https://github.com/OWNER/REPO.git

# Clone into a custom local directory name
git clone https://github.com/OWNER/REPO.git my-local-name

# Clone only the latest commit (shallow clone) — faster for large repos
# Useful when you only need the latest code, not the full history
git clone --depth 1 https://github.com/OWNER/REPO.git

# Clone a specific branch only
git clone --branch develop https://github.com/OWNER/REPO.git

# Create a new GitHub repository (public) via GitHub CLI
gh repo create REPO_NAME --public

# Create a new private repository
gh repo create REPO_NAME --private

# Create a repo AND push the current local directory to it in one step
gh repo create REPO_NAME --public --source=. --push

# Clone a repo via the GitHub CLI (also sets up gh context)
gh repo clone OWNER/REPO

# Fork a repository to your own GitHub account
gh repo fork OWNER/REPO

# Fork and clone it to your local machine in one command
gh repo fork OWNER/REPO --clone

# View repository details (description, stars, license, topics, etc.)
gh repo view OWNER/REPO

# Open the repository in your default web browser
gh repo view OWNER/REPO --web

# List all repos for a user or organization
gh repo list OWNER

# Rename the repository
gh repo rename NEW_NAME

# Delete a repository (IRREVERSIBLE — use with caution)
gh repo delete OWNER/REPO --confirm

# Archive a repository (makes it read-only, keeps all data)
gh repo archive OWNER/REPO

# Edit repository settings (description, homepage, topics, visibility)
gh repo edit --description "My awesome project" --homepage "https://example.com"
gh repo edit --add-topic "cli,github,tools"
gh repo edit --visibility private
```

---

## 4. Branching & Merging

```bash
# List all local branches (* marks the currently checked-out branch)
git branch

# List all branches including remote-tracking branches
git branch -a

# List only remote branches
git branch -r

# Create a new branch (does NOT switch to it)
git branch feature/my-feature

# Switch to an existing branch
git switch feature/my-feature
# (older syntax, also works:)
git checkout feature/my-feature

# Create a new branch AND switch to it immediately
git switch -c feature/my-feature
# (older syntax:)
git checkout -b feature/my-feature

# Create a branch from a specific commit or tag (not from HEAD)
git switch -c hotfix/issue-42 v1.2.3

# Rename the current branch
git branch -m new-branch-name

# Delete a branch that has been fully merged (safe delete)
git branch -d feature/my-feature

# Force-delete a branch even if it has unmerged commits (use carefully)
git branch -D feature/my-feature

# Delete a remote branch on GitHub
git push origin --delete feature/my-feature

# Merge another branch into the current branch
# Creates a merge commit if histories have diverged
git merge feature/my-feature

# Merge with a commit message, even for fast-forward merges (keeps history)
git merge --no-ff feature/my-feature -m "Merge feature/my-feature into main"

# Abort an in-progress merge (if you run into conflicts you can't resolve now)
git merge --abort

# Rebase the current branch on top of main
# Replays your commits on the tip of main — creates a linear history
git rebase main

# Interactively rebase the last 3 commits (squash, reorder, edit messages)
git rebase -i HEAD~3

# Abort a rebase in progress
git rebase --abort

# Continue a rebase after resolving conflicts
git rebase --continue

# Cherry-pick a single commit from another branch into the current branch
# Useful for applying a specific bug fix without merging the entire branch
git cherry-pick <commit-sha>

# Cherry-pick a range of commits
git cherry-pick abc123^..def456
```

---

## 5. Staging, Committing & History

```bash
# Show the current state of the working directory and staging area
# Tells you which files are modified, staged, or untracked
git status

# Stage a specific file for the next commit
git add README.md

# Stage all changes in the current directory (new, modified, deleted)
git add .

# Stage changes interactively — choose individual hunks (chunks) to stage
# Great for keeping commits small and focused
git add -p

# Unstage a file (remove from staging area, keep changes in working directory)
git restore --staged README.md
# (older syntax:)
git reset HEAD README.md

# Discard all unstaged changes in a file (DESTRUCTIVE — changes are lost)
git restore README.md

# Commit staged changes with a message
git commit -m "feat: add user login feature"

# Stage all tracked files AND commit in one step (skips 'git add')
# Does NOT include untracked (new) files
git commit -am "fix: correct typo in homepage"

# Amend the most recent commit — useful for fixing commit messages or
# adding forgotten files (only do this BEFORE pushing to a shared branch)
git commit --amend -m "feat: add user login feature (corrected)"

# Amend without changing the commit message
git commit --amend --no-edit

# Show the full commit log
git log

# Compact one-line log with branch/tag decoration
git log --oneline --graph --decorate --all

# Show changes introduced by each commit
git log -p

# Show commits by a specific author
git log --author="Your Name"

# Show commits from the last 7 days
git log --since="7 days ago"

# Show what changed in a specific commit
git show <commit-sha>

# Show the diff between the working directory and the staging area
git diff

# Show the diff between staged changes and the last commit
git diff --staged

# Show diff between two branches
git diff main..feature/my-feature

# Save uncommitted changes temporarily (without committing)
# Useful when you need to switch branches quickly
git stash

# Stash with a descriptive label
git stash push -m "WIP: half-done login form"

# List all stashes
git stash list

# Apply the most recent stash and remove it from the stash list
git stash pop

# Apply a specific stash without removing it
git stash apply stash@{2}

# Drop (delete) a specific stash
git stash drop stash@{0}

# Clear all stashes
git stash clear

# Create a .gitignore file for a specific language/framework
# (requires the 'curl' command)
curl -o .gitignore https://www.toptal.com/developers/gitignore/api/node,macos,visualstudiocode
```

---

## 6. Remote Operations — Push / Pull / Fetch

```bash
# List all configured remotes (name + URL)
git remote -v

# Add a remote called 'origin' pointing to a GitHub repository
git remote add origin https://github.com/OWNER/REPO.git

# Change the URL of the 'origin' remote (e.g. after renaming the repo)
git remote set-url origin https://github.com/OWNER/NEW_REPO_NAME.git

# Remove a remote
git remote remove upstream

# Add a second remote called 'upstream' to track the original repo
# (common when working on a fork)
git remote add upstream https://github.com/ORIGINAL_OWNER/REPO.git

# Download changes from a remote without merging them
# Safe — does not touch your working directory
git fetch origin

# Fetch ALL remotes at once
git fetch --all

# Pull = fetch + merge (or fetch + rebase if pull.rebase is set)
# Brings your local branch up to date with the remote
git pull origin main

# Push the current branch to origin and set it as the upstream tracking branch
# After this, you can just run 'git push' / 'git pull' without arguments
git push -u origin feature/my-feature

# Push the current branch to its tracking remote
git push

# Force-push after a rebase (DANGEROUS on shared branches — rewrites history)
# Use --force-with-lease instead of --force — it fails if someone else pushed
git push --force-with-lease origin feature/my-feature

# Push all local branches to origin
git push --all origin

# Push all tags to origin
git push --tags
```

---

## 7. Pull Requests (GitHub CLI)

```bash
# Create a pull request from the current branch targeting 'main'
gh pr create --base main --title "feat: add login feature" \
  --body "Implements the user login form described in issue #12."

# Create a draft pull request (work in progress, not ready for review)
gh pr create --draft --base main --title "WIP: new dashboard layout"

# Create a PR and open it in the browser immediately
gh pr create --web

# List open pull requests in the current repository
gh pr list

# List merged pull requests
gh pr list --state merged

# List PRs assigned to you
gh pr list --assignee "@me"

# Filter PRs by label
gh pr list --label "bug"

# View details of PR #42 (title, status, reviewers, checks)
gh pr view 42

# Open PR #42 in the browser
gh pr view 42 --web

# Show the diff of PR #42 in the terminal
gh pr diff 42

# Check out PR #42 locally so you can test or review the code
gh pr checkout 42

# Request a review from a specific user or team
gh pr edit 42 --add-reviewer username,org/team-name

# Add a label to a PR
gh pr edit 42 --add-label "enhancement"

# Add an assignee
gh pr edit 42 --add-assignee username

# Change the base branch of a PR
gh pr edit 42 --base develop

# Approve PR #42 with a review comment
gh pr review 42 --approve --body "Looks great! Ship it."

# Request changes on PR #42
gh pr review 42 --request-changes --body "Please add unit tests."

# Leave a comment on a PR
gh pr comment 42 --body "Could you also update the docs?"

# Check the CI status of PR #42
gh pr checks 42

# Merge PR #42 using a merge commit
gh pr merge 42 --merge

# Merge using squash (all commits squashed into one)
gh pr merge 42 --squash

# Merge using rebase (linear history, no merge commit)
gh pr merge 42 --rebase

# Auto-merge as soon as all required checks pass
gh pr merge 42 --auto --squash

# Close a PR without merging
gh pr close 42

# Reopen a closed PR
gh pr reopen 42
```

---

## 8. Issues (GitHub CLI)

```bash
# Create a new issue interactively (opens an editor)
gh issue create

# Create an issue with title and body directly
gh issue create --title "Bug: login button not working on Safari" \
  --body "Steps to reproduce: 1. Open Safari 2. Click login 3. Nothing happens."

# Create an issue and assign it to yourself immediately
gh issue create --title "Refactor auth module" --assignee "@me"

# Create an issue with a label and milestone
gh issue create --title "Add dark mode" --label "enhancement" \
  --milestone "v2.0"

# List all open issues
gh issue list

# List closed issues
gh issue list --state closed

# List issues assigned to you
gh issue list --assignee "@me"

# Filter by label
gh issue list --label "bug"

# Search issues by keyword
gh issue list --search "login"

# View details of issue #7
gh issue view 7

# Open issue #7 in the browser
gh issue view 7 --web

# Add a comment to issue #7
gh issue comment 7 --body "I can reproduce this on macOS Sonoma with Safari 17."

# Edit the title and labels of issue #7
gh issue edit 7 --title "Bug: login button unresponsive on Safari 17" \
  --add-label "confirmed"

# Pin an issue (makes it appear at the top of the issues list)
gh issue pin 7

# Close issue #7 with a reason
gh issue close 7 --reason "completed"

# Reopen issue #7
gh issue reopen 7

# Transfer issue #7 to another repository
gh issue transfer 7 OWNER/OTHER_REPO

# Delete issue #7 (requires admin privileges; IRREVERSIBLE)
gh issue delete 7 --confirm
```

---

## 9. Releases & Tags

```bash
# Create a lightweight tag on the current commit
git tag v1.0.0

# Create an annotated tag with a message (recommended for releases)
# Annotated tags store tagger info, date, and a message
git tag -a v1.0.0 -m "Release version 1.0.0 — initial stable release"

# Tag a specific past commit
git tag -a v0.9.0 <commit-sha> -m "Beta release"

# List all tags
git tag

# Show details of a specific tag
git show v1.0.0

# Push a single tag to GitHub
git push origin v1.0.0

# Push ALL tags to GitHub at once
git push --tags

# Delete a local tag
git tag -d v1.0.0-rc1

# Delete a remote tag on GitHub
git push origin --delete v1.0.0-rc1

# Create a GitHub Release from a tag via the CLI
gh release create v1.0.0 \
  --title "v1.0.0 — First Stable Release" \
  --notes "## What's New\n- Feature A\n- Bug fix B"

# Create a release and upload binary assets
gh release create v1.0.0 ./dist/myapp-linux ./dist/myapp-windows.exe \
  --title "v1.0.0" --notes "See CHANGELOG.md"

# Create a pre-release (alpha / beta)
gh release create v2.0.0-beta.1 --prerelease \
  --title "v2.0.0 Beta 1" --notes "Test this before the final release."

# Auto-generate release notes from merged PRs since the last tag
gh release create v1.1.0 --generate-notes

# List all releases
gh release list

# View details of a specific release
gh release view v1.0.0

# Download all assets from a release
gh release download v1.0.0

# Edit an existing release
gh release edit v1.0.0 --title "v1.0.0 — Stable" --notes "Updated changelog."

# Delete a release (does NOT delete the underlying tag)
gh release delete v1.0.0 --confirm
```

---

## 10. GitHub Actions (GitHub CLI)

```bash
# List all workflows defined in the repository
gh workflow list

# View details of a specific workflow
gh workflow view "CI Pipeline"

# Manually trigger a workflow that has a 'workflow_dispatch' trigger
gh workflow run "CI Pipeline"

# Trigger with input parameters (if the workflow defines inputs)
gh workflow run "Deploy" --field environment=staging

# Enable a disabled workflow
gh workflow enable "CI Pipeline"

# Disable a workflow (stops future scheduled / event-triggered runs)
gh workflow disable "Nightly Build"

# List all recent workflow runs across all workflows
gh run list

# List runs for a specific workflow
gh run list --workflow "CI Pipeline"

# Filter runs by branch
gh run list --branch main

# Filter by status: queued, in_progress, completed, success, failure, cancelled
gh run list --status failure

# View details and logs of a specific run (use the run ID from 'gh run list')
gh run view <run-id>

# Stream live logs of a run to your terminal
gh run watch <run-id>

# Download the logs of a completed run
gh run view <run-id> --log

# Download artifacts produced by a run
gh run download <run-id>

# Re-run a failed workflow run
gh run rerun <run-id>

# Re-run only the failed jobs (not the whole workflow)
gh run rerun <run-id> --failed-only

# Cancel a workflow run that is in progress
gh run cancel <run-id>

# Delete a workflow run record (useful for cleaning up old runs)
gh run delete <run-id>

# List secrets available to Actions in the repository
gh secret list

# Set / update a secret (value is read securely from stdin)
echo "my-secret-value" | gh secret set MY_SECRET

# Set a secret for a specific environment (e.g. 'production')
gh secret set DB_PASSWORD --env production

# Delete a secret
gh secret delete MY_SECRET

# List repository variables (non-secret configuration values)
gh variable list

# Set a repository variable
gh variable set APP_ENV --body "production"

# Delete a variable
gh variable delete APP_ENV
```

---

## 11. Gists

Gists are single-file (or small multi-file) snippets hosted on GitHub.
They have their own Git repository and can be public or secret (unlisted).

```bash
# Create a public gist from a local file
gh gist create myfile.txt

# Create a public gist with a description
gh gist create myfile.txt --desc "Useful shell aliases"

# Create a secret (unlisted) gist
gh gist create myfile.txt --secret

# Create a gist from multiple files
gh gist create file1.py file2.sh

# Create a gist from stdin (pipe content)
echo "print('Hello')" | gh gist create --filename hello.py

# List your gists
gh gist list

# List your secret gists
gh gist list --secret

# View a gist (use the gist ID from 'gh gist list')
gh gist view <gist-id>

# Open a gist in the browser
gh gist view <gist-id> --web

# Edit a gist (opens your $EDITOR)
gh gist edit <gist-id>

# Clone a gist as a local Git repository (you can push updates back)
gh gist clone <gist-id>

# Delete a gist
gh gist delete <gist-id>
```

---

## 12. Collaborators & Teams

```bash
# Invite a user as a collaborator with 'write' access
# Access levels: pull (read), triage, push (write), maintain, admin
gh api repos/OWNER/REPO/collaborators/USERNAME \
  --method PUT --field permission=push

# List all collaborators
gh api repos/OWNER/REPO/collaborators

# Remove a collaborator
gh api repos/OWNER/REPO/collaborators/USERNAME --method DELETE

# List all teams in an organization
gh api orgs/ORG_NAME/teams

# Create a new team in an organization
gh api orgs/ORG_NAME/teams --method POST \
  --field name="backend-team" \
  --field description="Back-end developers" \
  --field privacy=closed

# Add a member to a team
gh api orgs/ORG_NAME/teams/TEAM_SLUG/memberships/USERNAME \
  --method PUT --field role=member

# Give a team access to a repository
gh api orgs/ORG_NAME/teams/TEAM_SLUG/repos/OWNER/REPO \
  --method PUT --field permission=push

# List members of an organization
gh api orgs/ORG_NAME/members

# Invite a user to an organization
gh api orgs/ORG_NAME/invitations --method POST \
  --field login=USERNAME --field role=member
```

---

## 13. GitHub Packages

GitHub Packages hosts Docker images, npm packages, Maven artifacts, and more.

```bash
# Authenticate Docker with GitHub Container Registry (ghcr.io)
echo "$GITHUB_TOKEN" | docker login ghcr.io -u USERNAME --password-stdin

# Build a Docker image
docker build -t ghcr.io/OWNER/IMAGE_NAME:latest .

# Push the image to GitHub Container Registry
docker push ghcr.io/OWNER/IMAGE_NAME:latest

# Pull an image from GitHub Container Registry
docker pull ghcr.io/OWNER/IMAGE_NAME:latest

# List packages for a user or organization via the API
gh api user/packages?package_type=container

# Delete a specific package version
gh api \
  /user/packages/container/IMAGE_NAME/versions/VERSION_ID \
  --method DELETE

# Publish an npm package to GitHub Packages
# 1. Add to package.json: "publishConfig": { "registry": "https://npm.pkg.github.com" }
# 2. Add to .npmrc: //npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
npm publish

# Install an npm package from GitHub Packages
npm install @OWNER/PACKAGE_NAME
```

---

## 14. Security & Secrets

```bash
# List Dependabot alerts (dependency vulnerabilities)
gh api repos/OWNER/REPO/vulnerability-alerts

# Enable Dependabot alerts on the repository
gh api repos/OWNER/REPO/vulnerability-alerts --method PUT

# Enable Dependabot security updates (auto-creates PRs to fix vulnerabilities)
gh api repos/OWNER/REPO/automated-security-fixes --method PUT

# List code scanning alerts (requires code scanning to be set up)
gh api repos/OWNER/REPO/code-scanning/alerts

# Dismiss a code scanning alert
gh api repos/OWNER/REPO/code-scanning/alerts/ALERT_NUMBER \
  --method PATCH --field state=dismissed \
  --field dismissed_reason="false positive"

# List secret scanning alerts (leaked tokens/credentials in the repo)
gh api repos/OWNER/REPO/secret-scanning/alerts

# Resolve a secret scanning alert
gh api repos/OWNER/REPO/secret-scanning/alerts/ALERT_NUMBER \
  --method PATCH --field state=resolved \
  --field resolution="revoked"

# List all repository-level secrets (names only — values are never returned)
gh secret list

# Set an Actions secret
echo "super-secret" | gh secret set MY_SECRET

# Delete an Actions secret
gh secret delete MY_SECRET

# List Dependabot secrets
gh secret list --app dependabot

# Set a Dependabot secret
echo "my-token" | gh secret set NPM_TOKEN --app dependabot

# Add branch protection rules via API
# Requires push to main to go through at least 1 approved review
gh api repos/OWNER/REPO/branches/main/protection \
  --method PUT \
  --input - <<EOF
{
  "required_status_checks": null,
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "required_approving_review_count": 1
  },
  "restrictions": null
}
EOF
```

---

## 15. GitHub Pages

```bash
# Enable GitHub Pages from the 'gh-pages' branch via the API
gh api repos/OWNER/REPO/pages --method POST \
  --field source='{"branch":"gh-pages","path":"/"}'

# Enable Pages from the 'docs' folder on the 'main' branch
gh api repos/OWNER/REPO/pages --method POST \
  --field source='{"branch":"main","path":"/docs"}'

# View the current Pages configuration
gh api repos/OWNER/REPO/pages

# Trigger a Pages rebuild
gh api repos/OWNER/REPO/pages/builds --method POST

# Disable GitHub Pages
gh api repos/OWNER/REPO/pages --method DELETE

# Common workflow: push built site to the gh-pages branch
# (using the 'git subtree push' trick)
git subtree push --prefix dist origin gh-pages
```

---

## 16. Notifications & Dashboard

```bash
# List all unread GitHub notifications
gh api notifications

# Mark all notifications as read
gh api notifications --method PUT --field read=true

# List notifications for a specific repo
gh api repos/OWNER/REPO/notifications

# Subscribe to notifications on a thread
gh api notifications/threads/THREAD_ID/subscription \
  --method PUT --field subscribed=true

# Unsubscribe from a notification thread
gh api notifications/threads/THREAD_ID/subscription \
  --method PUT --field ignored=true

# List starred repositories
gh api user/starred

# Star a repository
gh api user/starred/OWNER/REPO --method PUT

# Unstar a repository
gh api user/starred/OWNER/REPO --method DELETE

# Watch a repository (get all notifications)
gh api repos/OWNER/REPO/subscription \
  --method PUT --field subscribed=true

# Ignore a repository (suppress notifications)
gh api repos/OWNER/REPO/subscription \
  --method PUT --field ignored=true

# List repositories you are watching
gh api user/subscriptions
```

---

## 17. Advanced Git Techniques

```bash
# Find the commit that introduced a bug using binary search
# Git checks out commits halfway between good and bad until it pinpoints the culprit
git bisect start
git bisect bad                  # current commit is broken
git bisect good v1.0.0          # this tag was known good
# Git checks out a midpoint commit — test it, then tell Git the result:
git bisect good                 # or: git bisect bad
# Repeat until Git prints "X is the first bad commit"
git bisect reset                # return to HEAD when done

# Show who last changed each line of a file (author, commit, date)
git blame README.md

# Search all commits for a string (find when something was added/removed)
git log -S "functionName" --source --all

# Apply only the changes from a specific commit to the current branch
git cherry-pick <commit-sha>

# Undo the most recent commit but keep the changes staged
git reset --soft HEAD~1

# Undo the most recent commit and unstage the changes (files remain modified)
git reset HEAD~1

# Undo the most recent commit and DISCARD all changes (DESTRUCTIVE)
git reset --hard HEAD~1

# Create a new commit that reverses the changes of a specific commit
# Safe for shared branches — does not rewrite history
git revert <commit-sha>

# Clean untracked files and directories from the working tree
git clean -fd

# Dry run — show what 'git clean' would delete without actually deleting
git clean -nfd

# Work with multiple branches checked out simultaneously (Git 2.5+)
# Great when you need to reference or build another branch without switching
git worktree add ../hotfix-branch hotfix/urgent-fix

# List all worktrees
git worktree list

# Remove a worktree
git worktree remove ../hotfix-branch

# Create an alias for a long command
# After this, 'git lg' shows a pretty log graph
git config --global alias.lg \
  "log --oneline --graph --decorate --all"

# Squash all commits on a feature branch into one before merging
git switch main
git merge --squash feature/my-feature
git commit -m "feat: add my feature (squashed)"

# Subtree merge — include another repo as a subdirectory
git remote add plugin-remote https://github.com/OWNER/PLUGIN.git
git fetch plugin-remote
git merge -s subtree --no-commit --allow-unrelated-histories plugin-remote/main
```

---

## 18. IDE Integration Notes

Every command above works in any terminal. Below is how **popular IDEs expose Git and GitHub** visually — and how they react when you run commands in the terminal alongside them.

---

### VS Code (Visual Studio Code)

| Feature | Where to find it |
|---|---|
| Source Control panel | Click the branch icon in the Activity Bar (or `Ctrl+Shift+G`) |
| Stage / unstage / commit | Graphical buttons in the Source Control panel |
| Branch switching | Click the branch name in the bottom-left status bar |
| Pull / Push | `...` menu in Source Control, or Command Palette (`Ctrl+Shift+P`) |
| Merge conflict resolver | Inline editor with "Accept Current / Incoming / Both" buttons |
| GitHub Pull Requests | Install **"GitHub Pull Requests and Issues"** extension |
| GitHub Actions | Install **"GitHub Actions"** extension — see workflow runs inline |

**How VS Code reacts to terminal commands:**
- VS Code **watches the file system** in real time. Any `git` command you run in the integrated terminal (`` Ctrl+` ``) is immediately reflected in the Source Control panel — staged files appear, commits show up in the Graph, and branch names update automatically.
- Running `git switch` in the terminal updates the branch indicator in the status bar within seconds.
- The `gh pr checkout` command checks out a PR branch; VS Code detects the branch change and updates accordingly.

---

### IntelliJ IDEA / JetBrains IDEs (WebStorm, PyCharm, GoLand, etc.)

| Feature | Where to find it |
|---|---|
| Git tool window | `View > Tool Windows > Git` (or `Alt+9`) |
| Commit dialog | `Ctrl+K` — stage, commit, and review diffs |
| Push dialog | `Ctrl+Shift+K` |
| Branch management | Click branch name in the bottom-right status bar |
| Log / history | **Git** tab → **Log** |
| GitHub integration | **Settings > Version Control > GitHub** — add account |
| PRs & issues | **Git > GitHub > Create Pull Request** |

**How JetBrains IDEs react to terminal commands:**
- JetBrains IDEs auto-refresh when they detect external file system changes. After running `git pull` or `git reset` in a terminal, the editor reloads modified files automatically (you may see a "File changed externally" prompt for unsaved changes).
- Running `git rebase -i` in the integrated terminal opens the rebase todo list in your configured `$GIT_EDITOR`; if set to `idea --wait`, JetBrains will open the interactive editor.
- Avoid switching branches externally while the IDE has unsaved files open — it may prompt you to save or discard.

---

### Vim / Neovim

| Feature | Plugin |
|---|---|
| Git status / stage / commit | **vim-fugitive** (`Gstatus`, `Gcommit`, `Gpush`) |
| Inline blame | **gitsigns.nvim** or `vim-fugitive` |
| GitHub PRs & Issues | **octo.nvim** (Neovim only) |
| Diff view | **diffview.nvim** |

**How Vim/Neovim reacts to terminal commands:**
- Vim does not auto-reload files by default. After external `git` operations, use `:checktime` or set `autoread` in your config to reload changed buffers automatically.
- With `vim-fugitive`, many Git operations can be done entirely inside Vim without switching to a terminal.
- Neovim's built-in terminal (`:terminal`) lets you run `gh` commands and switch back to editing seamlessly.

---

### Eclipse

| Feature | Where to find it |
|---|---|
| Git perspective | `Window > Perspective > Open Perspective > Git` |
| Stage / commit | **Git Staging** view |
| Branch management | **Git Repositories** view |
| GitHub integration | Install **EGit** plugin (usually bundled) |

**How Eclipse reacts to terminal commands:**
- Eclipse does **not** auto-refresh by default after external changes. Press `F5` on a project or enable **"Refresh using native hooks or polling"** in `Preferences > General > Workspace`.
- After running `git pull` or `git checkout` externally, refresh the project so Eclipse picks up the new/modified files.

---

### Xcode (macOS)

| Feature | Where to find it |
|---|---|
| Source Control navigator | `View > Navigators > Source Control` |
| Commit | `Source Control > Commit…` |
| Branch management | `Source Control > Checkout…` / `New Branch…` |
| Remote management | `Source Control > Repositories` |

**How Xcode reacts to terminal commands:**
- Xcode monitors the file system and updates the Source Control navigator automatically when you run `git` commands in Terminal.app.
- After running `git pull` or `git merge` in the terminal, Xcode will show the updated branch state and any merge conflicts in the editor gutter.
- For GitHub-specific operations (PRs, Actions), you still need to use `gh` CLI or the GitHub website — Xcode does not have native GitHub PR integration.

---

## Quick Reference Cheat Sheet

```
git init / clone       — start a repository
git add / commit       — record changes
git push / pull        — sync with GitHub
git branch / switch    — manage branches
git merge / rebase     — integrate branches
git stash              — shelve WIP changes
git tag                — mark a release point
git log / diff         — inspect history

gh auth login          — authenticate GitHub CLI
gh repo create/clone   — manage repositories
gh pr create/merge     — manage pull requests
gh issue create/close  — manage issues
gh release create      — publish a release
gh run list/watch      — monitor Actions workflows
gh secret set          — manage secrets
gh gist create         — share code snippets
```

---

> **Tip:** Run `git help <command>` or `gh help <command>` at any time for the
> full official documentation for that specific command, right in your terminal.

---

*Maintained in [github-power-cmd](https://github.com/sowmitra8/github-power-cmd). Contributions welcome — open an issue or PR!*
