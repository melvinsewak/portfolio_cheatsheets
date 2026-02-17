# Git & GitHub Beginner Commands Cheatsheet

## Git Basics

### git init - Initialize Repository
```bash
# Create new Git repository in current directory
git init

# Create repository in specific directory
git init my-project

# Creates .git directory (hidden) containing Git metadata
```

### git clone - Copy Repository
```bash
# Clone remote repository
git clone https://github.com/username/repo.git

# Clone to specific directory
git clone https://github.com/username/repo.git my-folder

# Clone specific branch
git clone -b branch-name https://github.com/username/repo.git

# Shallow clone (faster, recent history only)
git clone --depth 1 https://github.com/username/repo.git
```

### git status - Check Repository State
```bash
# Show working tree status
git status

# Short format
git status -s
git status --short

# Output:
# M - Modified
# A - Added (staged)
# D - Deleted
# ?? - Untracked
# MM - Modified, staged, then modified again
```

## Basic Workflow

### git add - Stage Changes
```bash
# Add specific file
git add filename.txt

# Add multiple files
git add file1.txt file2.txt

# Add all files in directory
git add .

# Add all modified files
git add -A
git add --all

# Add by pattern
git add *.txt
git add src/*.js

# Interactive staging
git add -i

# Add hunks (parts of files)
git add -p
```

### git commit - Save Changes
```bash
# Commit with message
git commit -m "Add new feature"

# Multi-line commit message
git commit -m "Add user authentication" -m "Implemented login and registration"

# Commit all tracked modified files (skip staging)
git commit -am "Update documentation"

# Amend last commit (change message)
git commit --amend -m "New commit message"

# Amend last commit (add forgotten files)
git add forgotten-file.txt
git commit --amend --no-edit

# Commit with detailed message in editor
git commit
```

### git push - Upload to Remote
```bash
# Push to remote branch
git push origin main
git push origin master

# Push and set upstream (first time)
git push -u origin main
git push --set-upstream origin main

# Push all branches
git push --all

# Force push (dangerous!)
git push -f origin main
git push --force origin main

# Push tags
git push --tags
```

### git pull - Download from Remote
```bash
# Pull from remote branch
git pull origin main

# Pull with rebase (cleaner history)
git pull --rebase origin main

# Pull all branches
git pull --all

# Pull specific branch
git pull origin feature-branch
```

## Viewing History

### git log - Show Commit History
```bash
# Show all commits
git log

# Compact one-line format
git log --oneline

# Show last N commits
git log -5

# Show with file changes
git log --stat

# Show with actual changes
git log -p
git log --patch

# Show graph
git log --graph --oneline --all

# Show commits by author
git log --author="John Doe"

# Show commits in date range
git log --since="2 weeks ago"
git log --after="2024-01-01" --before="2024-12-31"

# Search commit messages
git log --grep="bugfix"

# Show commits affecting specific file
git log filename.txt
```

### git show - Show Commit Details
```bash
# Show latest commit
git show

# Show specific commit
git show abc123

# Show specific file from commit
git show abc123:path/to/file.txt

# Show commit with stats
git show --stat abc123
```

### git diff - Show Differences
```bash
# Show unstaged changes
git diff

# Show staged changes
git diff --staged
git diff --cached

# Show changes in specific file
git diff filename.txt

# Compare branches
git diff main feature-branch

# Compare commits
git diff abc123 def456

# Show only changed file names
git diff --name-only

# Show summary
git diff --stat
```

## Working with Remotes

### git remote - Manage Remotes
```bash
# List remotes
git remote
git remote -v  # Verbose (shows URLs)

# Add remote
git remote add origin https://github.com/username/repo.git

# Remove remote
git remote remove origin

# Rename remote
git remote rename origin upstream

# Show remote details
git remote show origin

# Change remote URL
git remote set-url origin https://github.com/username/new-repo.git
```

### git fetch - Download from Remote
```bash
# Fetch from origin
git fetch origin

# Fetch all remotes
git fetch --all

# Fetch and prune deleted branches
git fetch -p
git fetch --prune

# Fetch specific branch
git fetch origin main
```

## Basic Branching

### git branch - Manage Branches
```bash
# List local branches
git branch

# List all branches (including remote)
git branch -a

# List remote branches
git branch -r

# Create new branch
git branch feature-name

# Delete branch
git branch -d feature-name

# Force delete branch
git branch -D feature-name

# Rename current branch
git branch -m new-name

# Show last commit on each branch
git branch -v
```

### git checkout - Switch Branches
```bash
# Switch to existing branch
git checkout main
git checkout feature-name

# Create and switch to new branch
git checkout -b new-feature

# Switch to previous branch
git checkout -

# Discard changes in file
git checkout -- filename.txt

# Checkout specific commit (detached HEAD)
git checkout abc123
```

### git switch - Modern Branch Switching
```bash
# Switch to branch (Git 2.23+)
git switch main

# Create and switch to new branch
git switch -c new-feature

# Switch to previous branch
git switch -

# Restore working tree files
git restore filename.txt
```

## .gitignore File

### Creating .gitignore
```bash
# Create .gitignore file
touch .gitignore

# Common patterns
echo "node_modules/" >> .gitignore
echo "*.log" >> .gitignore
echo ".env" >> .gitignore
echo "dist/" >> .gitignore
```

### Common .gitignore Patterns
```
# Dependencies
node_modules/
vendor/

# Build outputs
dist/
build/
*.exe
*.dll

# Environment files
.env
.env.local

# IDE files
.vscode/
.idea/
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Temporary files
*.tmp
temp/
```

## GitHub Basics

### Creating Repository on GitHub
```bash
# Via GitHub website:
# 1. Click "+" → "New repository"
# 2. Enter repository name
# 3. Choose public/private
# 4. Initialize with README (optional)
# 5. Click "Create repository"

# Connect local repo to GitHub:
git remote add origin https://github.com/username/repo.git
git branch -M main
git push -u origin main
```

### README.md
```markdown
# Project Title

Brief description of the project.

## Installation

```bash
npm install
```

## Usage

```bash
npm start
```

## Contributing

Pull requests are welcome!

## License

MIT
```

### Cloning from GitHub
```bash
# HTTPS (requires password)
git clone https://github.com/username/repo.git

# SSH (requires SSH key setup)
git clone git@github.com:username/repo.git

# GitHub CLI
gh repo clone username/repo
```

## Best Practices (Do's)

✅ **Write clear commit messages**
```bash
# Good
git commit -m "Add user authentication feature"
git commit -m "Fix: Resolve null pointer exception in user service"

# Bad
git commit -m "fixed stuff"
git commit -m "changes"
```

✅ **Commit related changes together**
```bash
# Group related changes
git add src/auth.js src/user.js
git commit -m "Add authentication module"
```

✅ **Pull before you push**
```bash
git pull origin main
git push origin main
```

✅ **Use meaningful branch names**
```bash
# Good
git branch feature/user-authentication
git branch bugfix/login-error
git branch hotfix/security-patch

# Avoid
git branch test
git branch branch1
```

✅ **Check status before committing**
```bash
git status
git diff
git add .
git commit -m "Message"
```

✅ **Use .gitignore properly**
```bash
# Add before first commit
echo "node_modules/" > .gitignore
git add .gitignore
git commit -m "Add gitignore"
```

## Common Mistakes (Don'ts)

❌ **Don't commit large files**
```bash
# Bad - commits large binary files
git add video.mp4  # 500MB file

# Good - use .gitignore or Git LFS
echo "*.mp4" >> .gitignore
```

❌ **Don't commit sensitive data**
```bash
# Bad - commits password
git add .env  # Contains API keys

# Good - ignore sensitive files
echo ".env" >> .gitignore
```

❌ **Don't work directly on main branch**
```bash
# Bad
git checkout main
# make changes
git commit -m "Experimental feature"

# Good
git checkout -b feature/new-feature
# make changes
git commit -m "Add new feature"
```

❌ **Don't use git add . without checking**
```bash
# Bad - might add unwanted files
git add .
git commit -m "Update"

# Good - review first
git status
git add specific-file.txt
git commit -m "Update specific file"
```

❌ **Don't force push to shared branches**
```bash
# Dangerous on shared branches
git push -f origin main

# Better - use force-with-lease
git push --force-with-lease origin feature-branch
```

❌ **Don't ignore error messages**
```bash
# Read and understand errors
git push
# error: failed to push some refs to 'origin'

# Fix the issue (usually need to pull first)
git pull origin main
git push origin main
```

❌ **Don't create messy commit history**
```bash
# Bad - too many tiny commits
git commit -m "typo"
git commit -m "another typo"
git commit -m "forgot semicolon"

# Good - combine related changes
# Make changes
git add .
git commit -m "Fix typos and formatting issues"
```

## Undoing Changes

### Before Staging
```bash
# Discard changes in working directory
git checkout -- filename.txt
git restore filename.txt  # Modern way
```

### After Staging
```bash
# Unstage file
git reset HEAD filename.txt
git restore --staged filename.txt  # Modern way
```

### After Committing
```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Create new commit that undoes changes
git revert HEAD
```

## Quick Command Reference

```bash
# Setup
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Start
git init
git clone <url>

# Work
git status
git add <file>
git commit -m "message"
git push origin <branch>
git pull origin <branch>

# Branch
git branch <name>
git checkout <name>
git merge <name>

# Info
git log
git diff
git show
```
