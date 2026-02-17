# Git & GitHub Intermediate Commands Cheatsheet

## Branch Management

### Advanced Branching
```bash
# List branches with last commit
git branch -v

# List merged branches
git branch --merged

# List unmerged branches
git branch --no-merged

# Show branches containing commit
git branch --contains abc123

# Track remote branch
git branch --track feature origin/feature

# Set upstream for existing branch
git branch --set-upstream-to=origin/main main
```

### Merging Branches
```bash
# Merge branch into current branch
git checkout main
git merge feature-branch

# Merge with no fast-forward (creates merge commit)
git merge --no-ff feature-branch

# Merge with squash (combine all commits)
git merge --squash feature-branch

# Abort merge
git merge --abort

# Continue merge after resolving conflicts
git merge --continue
```

## Resolving Merge Conflicts

### Conflict Resolution
```bash
# Check conflict status
git status

# View conflicts
git diff

# Open conflicted files - look for markers:
# <<<<<<< HEAD
# (your changes)
# =======
# (incoming changes)
# >>>>>>> feature-branch

# After resolving manually:
git add resolved-file.txt
git commit  # Or git merge --continue

# Use specific version
git checkout --ours file.txt    # Keep your version
git checkout --theirs file.txt  # Keep their version

# Use merge tool
git mergetool
```

### Merge Strategies
```bash
# Recursive (default)
git merge -s recursive feature-branch

# Ours (keep our version)
git merge -s ours feature-branch

# Theirs (prefer their changes)
git merge -X theirs feature-branch
```

## Stashing Changes

### git stash - Save Work Temporarily
```bash
# Stash changes
git stash
git stash save "Work in progress"

# Stash including untracked files
git stash -u
git stash --include-untracked

# List stashes
git stash list

# Show stash contents
git stash show
git stash show -p stash@{0}

# Apply latest stash (keep stash)
git stash apply

# Apply specific stash
git stash apply stash@{2}

# Pop latest stash (remove after applying)
git stash pop

# Drop stash
git stash drop stash@{0}

# Clear all stashes
git stash clear

# Create branch from stash
git stash branch new-branch-name
```

## Undoing Changes

### git reset - Undo Commits
```bash
# Soft reset (keep changes staged)
git reset --soft HEAD~1

# Mixed reset (keep changes unstaged) - default
git reset HEAD~1
git reset --mixed HEAD~1

# Hard reset (discard all changes) - DANGEROUS
git reset --hard HEAD~1

# Reset to specific commit
git reset --hard abc123

# Reset specific file
git reset HEAD filename.txt
```

### git revert - Safe Undo
```bash
# Revert last commit (creates new commit)
git revert HEAD

# Revert specific commit
git revert abc123

# Revert without committing
git revert -n abc123

# Revert range of commits
git revert abc123..def456

# Abort revert
git revert --abort
```

### git restore - Restore Files
```bash
# Restore file from HEAD
git restore filename.txt

# Restore file from specific commit
git restore --source=abc123 filename.txt

# Unstage file
git restore --staged filename.txt

# Restore and unstage
git restore --staged --worktree filename.txt
```

## Tagging

### Creating Tags
```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (recommended)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag specific commit
git tag -a v1.0.0 abc123 -m "Release 1.0.0"

# List tags
git tag
git tag -l "v1.*"

# Show tag details
git show v1.0.0

# Push tags to remote
git push origin v1.0.0
git push origin --tags

# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0
```

## Pull Requests (GitHub)

### Creating Pull Requests
```bash
# Via command line (GitHub CLI)
gh pr create
gh pr create --title "Add feature" --body "Description"

# Via web interface:
# 1. Push branch to GitHub
git push origin feature-branch

# 2. Go to repository on GitHub
# 3. Click "Compare & pull request"
# 4. Fill in title and description
# 5. Select reviewers
# 6. Click "Create pull request"
```

### Managing Pull Requests
```bash
# List PRs
gh pr list

# View PR
gh pr view 123

# Checkout PR locally
gh pr checkout 123
git fetch origin pull/123/head:pr-123
git checkout pr-123

# Merge PR
gh pr merge 123

# Close PR
gh pr close 123

# Reopen PR
gh pr reopen 123

# Review PR
gh pr review 123 --approve
gh pr review 123 --request-changes
gh pr review 123 --comment
```

## Forking Workflow

### Contributing to Open Source
```bash
# 1. Fork repository on GitHub (click Fork button)

# 2. Clone your fork
git clone https://github.com/yourusername/repo.git
cd repo

# 3. Add upstream remote
git remote add upstream https://github.com/original/repo.git

# 4. Create feature branch
git checkout -b feature-branch

# 5. Make changes and commit
git add .
git commit -m "Add feature"

# 6. Keep fork updated
git fetch upstream
git checkout main
git merge upstream/main

# 7. Push to your fork
git push origin feature-branch

# 8. Create pull request on GitHub
```

### Sync Fork with Upstream
```bash
# Fetch upstream changes
git fetch upstream

# Merge into local main
git checkout main
git merge upstream/main

# Push to your fork
git push origin main

# Alternative: rebase
git checkout main
git rebase upstream/main
git push origin main --force-with-lease
```

## GitHub Issues

### Working with Issues
```bash
# Create issue (GitHub CLI)
gh issue create
gh issue create --title "Bug report" --body "Description"

# List issues
gh issue list

# View issue
gh issue view 123

# Close issue
gh issue close 123

# Reopen issue
gh issue reopen 123

# Link commit to issue
git commit -m "Fix bug (closes #123)"
git commit -m "Work on feature (refs #456)"
```

## Git Aliases

### Creating Shortcuts
```bash
# Set aliases
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status

# Complex aliases
git config --global alias.lg "log --graph --oneline --all"
git config --global alias.unstage "reset HEAD --"
git config --global alias.last "log -1 HEAD"
git config --global alias.visual "log --graph --oneline --all --decorate"

# Usage
git co main
git br feature
git st
git lg
```

### Common Aliases
```bash
# Add to ~/.gitconfig or use git config

[alias]
    co = checkout
    br = branch
    ci = commit
    st = status
    unstage = reset HEAD --
    last = log -1 HEAD
    lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    undo = reset --soft HEAD~1
    amend = commit --amend --no-edit
```

## Advanced Git Log

### Filtering and Formatting
```bash
# Filter by date
git log --since="2 weeks ago"
git log --until="2024-01-01"

# Filter by author
git log --author="John Doe"

# Filter by committer
git log --committer="John Doe"

# Search commit messages
git log --grep="bugfix"

# Search code changes
git log -S "function_name"  # Pickaxe
git log -G "regex_pattern"

# Filter by file
git log --follow filename.txt

# Show commits that changed specific line range
git log -L 10,20:filename.txt

# Custom format
git log --pretty=format:"%h - %an, %ar : %s"

# Show commits between branches
git log main..feature-branch

# Show commits in one but not other
git log main...feature-branch

# First parent only (merge commits)
git log --first-parent

# Reverse order (oldest first)
git log --reverse
```

## GitHub Projects

### Project Management
```bash
# Create project (GitHub CLI)
gh project create --owner @me --title "My Project"

# List projects
gh project list

# View project
gh project view 1

# Add issue to project
gh project item-add 1 --owner @me --content-id <issue-id>
```

## Collaborative Workflows

### Feature Branch Workflow
```bash
# 1. Create feature branch
git checkout -b feature/user-authentication

# 2. Work on feature
git add .
git commit -m "Implement login"

# 3. Keep updated with main
git checkout main
git pull origin main
git checkout feature/user-authentication
git merge main

# 4. Push feature branch
git push origin feature/user-authentication

# 5. Create pull request
gh pr create

# 6. After approval, merge
gh pr merge
```

### Code Review Process
```bash
# Reviewer fetches branch
git fetch origin
git checkout feature-branch

# Review changes
git diff main...feature-branch
git log main..feature-branch

# Test locally
npm install
npm test

# Provide feedback on GitHub PR page

# Author addresses feedback
git add .
git commit -m "Address review comments"
git push origin feature-branch
```

## Best Practices (Do's)

✅ **Use meaningful branch names**
```bash
# Good
git checkout -b feature/add-user-auth
git checkout -b bugfix/fix-login-error
git checkout -b hotfix/security-patch-v2

# Pattern: type/description
# Types: feature, bugfix, hotfix, refactor, docs
```

✅ **Keep branches up to date**
```bash
# Regularly sync with main
git checkout main
git pull origin main
git checkout feature-branch
git merge main
```

✅ **Write descriptive PR descriptions**
```markdown
## What
Add user authentication feature

## Why
Users need to log in to access personalized content

## How
- Implement JWT authentication
- Add login/register endpoints
- Create auth middleware

## Testing
- Added unit tests for auth service
- Manual testing completed
```

✅ **Use draft PRs for work in progress**
```bash
# Create draft PR
gh pr create --draft

# Mark as ready
gh pr ready
```

✅ **Squash commits before merging**
```bash
# Interactive rebase
git rebase -i HEAD~5

# Mark commits as 'squash' or 'fixup'
```

✅ **Use protected branches**
```
# On GitHub: Settings → Branches → Add rule
- Require pull request reviews
- Require status checks
- Require branches to be up to date
```

## Common Mistakes (Don'ts)

❌ **Don't force push to shared branches**
```bash
# Bad
git push -f origin main

# Good - only on feature branches if needed
git push --force-with-lease origin feature-branch
```

❌ **Don't create huge pull requests**
```bash
# Bad - 50 files changed
# Too hard to review

# Good - smaller, focused PRs
# One feature or fix per PR
```

❌ **Don't merge without testing**
```bash
# Bad
gh pr merge 123  # Without testing

# Good
gh pr checkout 123
npm test
npm build
gh pr merge 123
```

❌ **Don't commit merge conflicts**
```bash
# Bad - files with conflict markers
<<<<<<< HEAD
code
=======
other code
>>>>>>> branch

# Good - resolve all conflicts first
```

❌ **Don't work on main directly**
```bash
# Bad
git checkout main
# make changes
git commit -m "Quick fix"

# Good
git checkout -b hotfix/quick-fix
# make changes
git commit -m "Quick fix"
# Create PR
```

❌ **Don't ignore CI failures**
```bash
# Always fix failing tests before merging
# Check GitHub Actions/CI output
# Address all issues
```

## GitHub CLI Quick Reference

```bash
# Authentication
gh auth login

# Repository
gh repo create
gh repo clone username/repo
gh repo view

# Pull Requests
gh pr create
gh pr list
gh pr view 123
gh pr checkout 123
gh pr merge 123

# Issues
gh issue create
gh issue list
gh issue view 123
gh issue close 123

# Workflows
gh workflow list
gh workflow run workflow-name
gh run list
gh run view 123
```
