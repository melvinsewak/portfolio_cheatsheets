# Git & GitHub Commands Cheatsheet

Welcome to the Git and GitHub commands cheatsheet! This guide will help you master version control and collaboration.

## Overview

**Git** is a distributed version control system that tracks changes in source code during software development. **GitHub** is a web-based platform that hosts Git repositories and provides collaboration features like pull requests, issues, and project management.

### Git vs GitHub

- **Git**: Command-line tool for version control (works locally)
- **GitHub**: Cloud platform for hosting and collaborating on Git repositories

## Contents

- [Beginner.md](Beginner.md) - Basic Git commands, workflows, and GitHub basics
- [Intermediate.md](Intermediate.md) - Branching, merging, pull requests, and collaboration
- [Advanced.md](Advanced.md) - Rebasing, advanced workflows, hooks, and GitHub Actions

## Quick Reference

### Basic Workflow
```bash
# Initialize repository
git init

# Clone repository
git clone https://github.com/username/repo.git

# Check status
git status

# Add files
git add filename.txt
git add .

# Commit changes
git commit -m "Commit message"

# Push to remote
git push origin main
```

### Branching
```bash
# Create branch
git branch feature-name

# Switch to branch
git checkout feature-name
git switch feature-name  # Modern command

# Create and switch
git checkout -b feature-name
git switch -c feature-name

# Merge branch
git merge feature-name
```

### Viewing History
```bash
# View commit history
git log

# Compact view
git log --oneline

# With graph
git log --graph --oneline --all
```

## Key Features

- **Version Control**: Track every change in your codebase
- **Collaboration**: Work with teams seamlessly
- **Branching**: Experiment without affecting main code
- **History**: Complete audit trail of changes
- **Backup**: Distributed copies across multiple machines

## Getting Started

### Install Git

**Linux (Ubuntu/Debian)**:
```bash
sudo apt-get update
sudo apt-get install git
```

**macOS**:
```bash
brew install git
```

**Windows**:
Download from [git-scm.com](https://git-scm.com/)

### Configure Git
```bash
# Set username
git config --global user.name "Your Name"

# Set email
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "code --wait"

# View configuration
git config --list
```

### Create GitHub Account

1. Visit [github.com](https://github.com)
2. Sign up for free account
3. Set up SSH keys (recommended) or use HTTPS

### SSH Key Setup
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Start ssh-agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Copy public key
cat ~/.ssh/id_ed25519.pub

# Add to GitHub: Settings → SSH and GPG keys → New SSH key
```

## Essential Tips

✅ **Write meaningful commit messages** - Describe what and why, not how  
✅ **Commit often** - Small, focused commits are easier to manage  
✅ **Pull before push** - Always sync with remote before pushing  
✅ **Use .gitignore** - Exclude unnecessary files from version control  
✅ **Branch for features** - Keep main/master branch stable

## Common Git States

```
Working Directory → Staging Area → Local Repository → Remote Repository
     (modified)        (staged)       (committed)        (pushed)
        ↓                  ↓               ↓                ↓
    git add          git commit      git push
```

## Additional Resources

- [Official Git Documentation](https://git-scm.com/doc)
- [GitHub Docs](https://docs.github.com/)
- [Pro Git Book](https://git-scm.com/book/en/v2) (Free)
- [GitHub Learning Lab](https://lab.github.com/)
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)
