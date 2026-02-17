# Git & GitHub Advanced Commands Cheatsheet

## Rebasing

### git rebase - Rewrite History
```bash
# Rebase current branch onto main
git checkout feature-branch
git rebase main

# Interactive rebase (last 5 commits)
git rebase -i HEAD~5

# Rebase onto specific commit
git rebase abc123

# Continue rebase after resolving conflicts
git rebase --continue

# Skip current commit
git rebase --skip

# Abort rebase
git rebase --abort

# Preserve merges
git rebase -p main
git rebase --preserve-merges main
```

### Interactive Rebase Commands
```bash
# In interactive rebase editor:
# pick = use commit
# reword = use commit, edit message
# edit = use commit, stop for amending
# squash = use commit, meld into previous
# fixup = like squash, discard message
# drop = remove commit

# Example:
pick abc123 Add feature
squash def456 Fix typo
reword ghi789 Update docs
drop jkl012 Remove debug code
```

### Rebase vs Merge
```bash
# Merge (preserves history)
git checkout main
git merge feature-branch
# Creates merge commit, keeps all history

# Rebase (linear history)
git checkout feature-branch
git rebase main
# Moves commits on top of main, cleaner history

# When to use:
# - Rebase: feature branches before merging
# - Merge: integrating completed features
# - Never rebase public/shared branches!
```

## Cherry-Picking

### git cherry-pick - Apply Specific Commits
```bash
# Cherry-pick single commit
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456

# Cherry-pick range
git cherry-pick abc123..def456

# Cherry-pick without committing
git cherry-pick -n abc123
git cherry-pick --no-commit abc123

# Continue after resolving conflicts
git cherry-pick --continue

# Abort cherry-pick
git cherry-pick --abort

# Add sign-off
git cherry-pick -s abc123
```

## Advanced Git Log

### Complex Filtering
```bash
# Show commits that changed function
git log -L :function_name:file.c

# Show commits between tags
git log v1.0..v2.0

# Show commits on branch but not main
git log main..feature-branch

# Show commits on either branch but not both
git log --left-right main...feature-branch

# Show commits affecting specific path
git log -- path/to/directory

# Show merge commits only
git log --merges

# Show non-merge commits
git log --no-merges

# Show first parent only
git log --first-parent

# Diff filter (added, deleted, modified)
git log --diff-filter=D  # Deleted files
git log --diff-filter=A  # Added files
git log --diff-filter=M  # Modified files
```

### Custom Formatting
```bash
# Custom format
git log --pretty=format:"%C(yellow)%h%Creset %C(blue)%ad%Creset %C(green)%an%Creset %s" --date=short

# JSON format (requires jq)
git log --pretty=format:'{"commit":"%H","author":"%an","date":"%ad","message":"%s"}' | jq -s '.'

# Graph with details
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative

# Show only commits from last release
git log $(git describe --tags --abbrev=0)..HEAD
```

## Submodules

### Managing Submodules
```bash
# Add submodule
git submodule add https://github.com/user/repo.git path/to/submodule

# Initialize submodules
git submodule init

# Update submodules
git submodule update

# Clone with submodules
git clone --recursive https://github.com/user/repo.git

# Update all submodules to latest
git submodule update --remote

# Execute command in all submodules
git submodule foreach 'git pull origin main'

# Remove submodule
git submodule deinit path/to/submodule
git rm path/to/submodule
rm -rf .git/modules/path/to/submodule
```

## Git Hooks

### Client-Side Hooks
```bash
# Located in .git/hooks/

# pre-commit - Run before commit
#!/bin/bash
npm run lint
npm test

# commit-msg - Validate commit message
#!/bin/bash
commit_msg_file=$1
commit_msg=$(cat "$commit_msg_file")

if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|test|chore):"; then
    echo "Error: Commit message must start with type (feat|fix|docs|...)"
    exit 1
fi

# pre-push - Run before push
#!/bin/bash
npm test
npm run build

# post-checkout - Run after checkout
#!/bin/bash
npm install

# Make hook executable
chmod +x .git/hooks/pre-commit
```

### Server-Side Hooks
```bash
# pre-receive - Run before receiving push
#!/bin/bash
# Enforce branch protection

# update - Run for each branch being updated
#!/bin/bash
# Validate commit messages

# post-receive - Run after receiving push
#!/bin/bash
# Trigger CI/CD, send notifications
```

### Husky (Popular Hook Manager)
```bash
# Install
npm install husky --save-dev

# Initialize
npx husky install

# Add pre-commit hook
npx husky add .husky/pre-commit "npm test"

# Add commit-msg hook
npx husky add .husky/commit-msg "npx --no -- commitlint --edit $1"
```

## GitHub Actions

### Basic Workflow
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run tests
      run: npm test
    
    - name: Build
      run: npm run build
```

### Advanced Workflow Features
```yaml
# Matrix builds
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [14, 16, 18]
runs-on: ${{ matrix.os }}

# Conditional steps
- name: Deploy
  if: github.ref == 'refs/heads/main'
  run: npm run deploy

# Secrets
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: ./deploy.sh

# Artifacts
- name: Upload artifact
  uses: actions/upload-artifact@v3
  with:
    name: dist
    path: dist/

# Caching
- name: Cache node modules
  uses: actions/cache@v3
  with:
    path: node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

### Workflow Commands
```bash
# List workflows
gh workflow list

# Run workflow
gh workflow run workflow-name

# View workflow run
gh run list
gh run view 123456

# Download artifacts
gh run download 123456

# Watch workflow run
gh run watch

# Re-run workflow
gh run rerun 123456
```

## Advanced Collaboration

### Code Review Best Practices
```bash
# Review commits
git log --oneline main..feature-branch

# View changes
git diff main...feature-branch

# Check individual file changes
git show feature-branch:path/to/file.js

# Leave comments on GitHub PR
gh pr review 123 --comment -b "Please add tests"

# Request changes
gh pr review 123 --request-changes -b "Needs error handling"

# Approve
gh pr review 123 --approve
```

### Protected Branches
```
# GitHub Settings → Branches → Add Rule

Options:
- Require pull request reviews before merging
- Require status checks to pass
- Require conversation resolution before merging
- Require signed commits
- Require linear history
- Include administrators
- Restrict who can push
- Allow force pushes
- Allow deletions
```

### Branch Naming Conventions
```bash
# Feature branches
feature/user-authentication
feature/add-payment-gateway

# Bug fixes
bugfix/fix-login-error
bugfix/resolve-memory-leak

# Hotfixes
hotfix/security-patch
hotfix/critical-bug

# Release branches
release/v1.2.0
release/2024-01-sprint

# Experimental
experiment/new-architecture
experiment/performance-test
```

## Git Reflog

### Recovery with Reflog
```bash
# View reflog
git reflog

# Show reflog for specific branch
git reflog show feature-branch

# Recover lost commit
git reflog
# Find commit hash
git checkout abc123

# Recover deleted branch
git reflog
# Find branch HEAD
git branch recovered-branch abc123

# Undo reset
git reset --hard abc123
# Oops! Recover:
git reflog
git reset --hard HEAD@{1}
```

## Git Bisect

### Binary Search for Bugs
```bash
# Start bisect
git bisect start

# Mark current as bad
git bisect bad

# Mark known good commit
git bisect good abc123

# Git checks out middle commit, test it:
npm test

# Mark as good or bad
git bisect good  # or git bisect bad

# Repeat until Git finds the bad commit

# End bisect
git bisect reset

# Automate bisect
git bisect start HEAD abc123
git bisect run npm test
```

## Performance and Large Repositories

### Shallow Clones
```bash
# Clone with limited history
git clone --depth 1 https://github.com/user/repo.git

# Clone single branch
git clone --single-branch --branch main https://github.com/user/repo.git

# Deepen shallow clone
git fetch --depth=100

# Unshallow
git fetch --unshallow
```

### Partial Clones
```bash
# Clone without blobs
git clone --filter=blob:none https://github.com/user/repo.git

# Clone without trees
git clone --filter=tree:0 https://github.com/user/repo.git

# Fetch missing objects
git fetch origin
```

### Git LFS (Large File Storage)
```bash
# Install Git LFS
git lfs install

# Track large files
git lfs track "*.psd"
git lfs track "*.mp4"

# Add .gitattributes
git add .gitattributes

# Commit and push
git add file.psd
git commit -m "Add design file"
git push

# Clone LFS repository
git lfs clone https://github.com/user/repo.git

# Fetch LFS files
git lfs pull
```

### Maintenance
```bash
# Garbage collection
git gc

# Aggressive garbage collection
git gc --aggressive --prune=now

# Check repository size
git count-objects -vH

# Clean up unnecessary files
git clean -fd

# Prune old objects
git prune

# Optimize repository
git repack -a -d --depth=250 --window=250
```

## Advanced Git Configuration

### Global Settings
```bash
# Editor
git config --global core.editor "code --wait"

# Diff tool
git config --global diff.tool vscode
git config --global difftool.vscode.cmd "code --wait --diff $LOCAL $REMOTE"

# Merge tool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd "code --wait $MERGED"

# Auto-correct
git config --global help.autocorrect 1

# Credential helper
git config --global credential.helper store

# Default branch name
git config --global init.defaultBranch main

# Rebase by default
git config --global pull.rebase true

# Show original state in conflicts
git config --global merge.conflictstyle diff3
```

### Repository-Specific Settings
```bash
# Use local config (no --global)
git config user.name "Work Name"
git config user.email "work@company.com"

# Include external config
git config --global include.path ~/.gitconfig-work
```

## Best Practices (Do's)

✅ **Use rebase for feature branches**
```bash
# Keep feature branch up to date
git checkout feature-branch
git fetch origin
git rebase origin/main
```

✅ **Write atomic commits**
```bash
# One logical change per commit
# Makes bisect, revert, cherry-pick easier
```

✅ **Use meaningful tag messages**
```bash
git tag -a v1.0.0 -m "Release 1.0.0 - Added user authentication, fixed bugs #123, #456"
```

✅ **Set up CI/CD early**
```yaml
# Add GitHub Actions from start
# Automated testing catches issues early
```

✅ **Use signed commits for security**
```bash
# Generate GPG key
gpg --gen-key

# Configure Git
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true

# Commit with signature
git commit -S -m "Signed commit"
```

✅ **Protect sensitive data**
```bash
# Use environment variables
# Never commit secrets
# Use git-secrets or similar tools
git secrets --scan
```

## Common Mistakes (Don'ts)

❌ **Don't rebase public branches**
```bash
# NEVER do this:
git checkout main
git rebase feature-branch

# Rewrites history others depend on
```

❌ **Don't use git add . blindly**
```bash
# Bad
git add .

# Good - review first
git status
git diff
git add -p  # Interactive staging
```

❌ **Don't store large binaries in Git**
```bash
# Use Git LFS or external storage
# Git is optimized for text, not binaries
```

❌ **Don't ignore .gitignore**
```bash
# Start with proper .gitignore
# Use templates from gitignore.io
```

❌ **Don't commit broken code**
```bash
# Always test before committing
npm test
npm run build
git commit
```

❌ **Don't use git push --force on shared branches**
```bash
# Very dangerous
# Use --force-with-lease if you must
git push --force-with-lease origin feature-branch
```

## Security Best Practices

```bash
# Enable commit signing
git config --global commit.gpgsign true

# Verify signatures
git log --show-signature

# Scan for secrets before commit
# Use tools like git-secrets, trufflehog

# Use SSH keys instead of passwords
ssh-keygen -t ed25519

# Enable two-factor authentication on GitHub

# Review permissions regularly
gh auth status
```

## Quick Advanced Commands Reference

```bash
# Rebase
git rebase -i HEAD~5
git rebase main

# Cherry-pick
git cherry-pick abc123

# Reflog
git reflog
git reset --hard HEAD@{1}

# Bisect
git bisect start
git bisect bad
git bisect good abc123

# Submodules
git submodule add <url>
git submodule update --init --recursive

# Stash
git stash save "WIP"
git stash pop

# Clean
git clean -fd

# GC
git gc --aggressive
```
