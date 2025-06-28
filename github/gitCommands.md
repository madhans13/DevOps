# 🚀 Complete Git Commands Guide

A comprehensive reference for Git commands with real-world scenarios and best practices. Know exactly when and how to use each command!

[![Git](https://img.shields.io/badge/git-%23F05033.svg?style=for-the-badge&logo=git&logoColor=white)](https://git-scm.com/)
[![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColor=white)](https://github.com/)

## 📚 Table of Contents

- [Initial Setup](#-initial-setup)
- [Repository Basics](#-repository-basics)
- [Daily Workflow](#-daily-workflow)
- [Branch Management](#-branch-management)
- [Remote Operations](#-remote-operations)
- [Viewing History](#-viewing-history)
- [Undoing Changes](#-undoing-changes)
- [Stashing](#-stashing)
- [Tags & Releases](#-tags--releases)
- [Advanced Operations](#-advanced-operations)
- [Emergency Commands](#-emergency-commands)
- [Workflow Examples](#-workflow-examples)
- [Best Practices](#-best-practices)

---

## 🔧 Initial Setup

### **When to use:** First time Git setup or new machine configuration

```bash
# Set your identity (required before first commit)
git config --global user.name "John Doe"
git config --global user.email "john@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Set preferred editor
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "vim"          # Vim

# View all configurations
git config --list
git config --global --list
```

**💡 Pro Tip:** Use `--global` for machine-wide settings, omit for repository-specific settings.

---

## 📁 Repository Basics

### **When to use:** Starting new projects or joining existing ones

```bash
# Start a new project
git init
git init my-project                    # Create and initialize directory

# Join an existing project
git clone https://github.com/user/repo.git
git clone <url> my-folder-name         # Clone to specific folder
git clone --depth 1 <url>              # Shallow clone (faster)

# Check repository status
git status                             # See changes and staging area
git status -s                          # Short format
git status --porcelain                 # Machine-readable format
```

**🎯 Use Cases:**
- `git init`: Starting a new project from scratch
- `git clone`: Downloading an existing project
- `git status`: Before commits, after making changes, debugging issues

---

## 🔄 Daily Workflow

### **When to use:** Your regular development cycle

```bash
# Stage changes
git add filename.txt                   # Stage specific file
git add src/                          # Stage entire directory  
git add .                             # Stage all changes
git add -A                            # Stage all (including deletions)
git add -u                            # Stage only tracked files
git add -p                            # Interactive staging (partial commits)

# Commit changes
git commit -m "Add user authentication"
git commit -am "Fix login bug"         # Add and commit tracked files
git commit --amend -m "Updated message" # Fix last commit message
git commit --amend --no-edit           # Add files to last commit

# Push changes
git push                              # Push current branch
git push origin main                  # Push to specific remote/branch
git push -u origin feature-branch     # Push and set upstream
git push --force-with-lease           # Safer force push
```

**🎯 Use Cases:**
- `git add .`: When you want to commit all changes
- `git add -p`: When you want to commit only parts of files
- `git commit -am`: Quick commit for bug fixes
- `git push -u`: First time pushing a new branch

**⚠️ When NOT to use:**
- `git add .` when you have temporary/test files
- `git push --force` on shared branches (use `--force-with-lease`)

---

## 🌿 Branch Management

### **When to use:** Feature development, experimentation, collaboration

```bash
# View branches
git branch                            # List local branches
git branch -r                         # List remote branches
git branch -a                         # List all branches
git branch -v                         # List with last commit info

# Create branches
git branch feature/user-profile       # Create branch
git checkout -b feature/api-integration # Create and switch
git switch -c hotfix/security-patch   # Modern syntax (Git 2.23+)

# Switch branches
git checkout main                     # Switch to main
git switch develop                    # Modern syntax
git checkout -                        # Switch to previous branch

# Merge branches
git merge feature/user-profile        # Merge into current branch
git merge --no-ff feature-branch      # Force merge commit
git merge --squash feature-branch     # Squash all commits into one

# Delete branches
git branch -d feature-completed       # Safe delete (merged only)
git branch -D feature-abandoned       # Force delete
git push origin --delete feature-old  # Delete remote branch
```

**🎯 Use Cases:**
- Create branch: Starting new features, experiments, hotfixes
- `git merge --no-ff`: When you want to preserve branch history
- `git merge --squash`: Clean up messy commit history
- Delete branch: After feature completion or abandoning work

**⚠️ When NOT to use:**
- Don't delete branches that aren't fully merged (unless intentional)
- Avoid force delete (`-D`) unless you're sure

---

## 🌐 Remote Operations

### **When to use:** Collaborating with teams, syncing changes

```bash
# Remote management
git remote -v                         # List remotes
git remote add upstream https://...    # Add upstream remote
git remote remove origin              # Remove remote
git remote rename origin old-origin   # Rename remote

# Fetch and pull
git fetch                             # Download remote changes
git fetch origin                      # Fetch from specific remote
git fetch --all                       # Fetch from all remotes
git pull                              # Fetch and merge
git pull --rebase                     # Fetch and rebase instead of merge
git pull origin develop               # Pull specific branch

# Push variations
git push                              # Push current branch
git push --all                        # Push all branches
git push --tags                       # Push all tags
git push origin :feature-branch       # Delete remote branch (old syntax)
```

**🎯 Use Cases:**
- `git fetch`: Check what others have done without affecting your work
- `git pull --rebase`: Keep linear history, avoid merge commits
- `git remote add upstream`: Contributing to open source projects
- `git push --tags`: Releasing versions

**⚠️ When NOT to use:**
- `git pull` without understanding what you're merging
- `git push --force` on shared branches

---

## 📖 Viewing History

### **When to use:** Understanding project evolution, debugging, code reviews

```bash
# Basic history
git log                               # Full commit history
git log --oneline                     # Compact view
git log --graph --all                 # Visual branch structure
git log -n 5                          # Last 5 commits
git log --since="2 weeks ago"         # Time-based filtering
git log --author="John"               # Filter by author

# Advanced history
git log --stat                        # Show changed files
git log -p                            # Show actual changes
git log --grep="bug"                  # Search commit messages
git log main..feature                 # Commits in feature not in main
git log --follow filename.txt         # Track file renames

# Show changes
git diff                              # Working directory vs staging
git diff --staged                     # Staging vs last commit
git diff HEAD~1                       # Current vs previous commit
git diff main..feature                # Compare branches
git diff --name-only                  # Just file names

# Specific commits
git show HEAD                         # Show last commit
git show abc123                       # Show specific commit
git show HEAD~2:filename.txt          # File from 2 commits ago
```

**🎯 Use Cases:**
- `git log --oneline`: Quick overview of recent work
- `git log --graph`: Understanding branch merges
- `git diff --staged`: Review before committing
- `git show`: Examining specific changes
- `git log --follow`: Tracking file history through renames

---

## ↩️ Undoing Changes

### **When to use:** Fixing mistakes, reverting changes

```bash
# Unstage changes
git reset filename.txt                # Unstage specific file
git reset                            # Unstage all files
git restore --staged filename.txt     # Modern syntax

# Undo commits (local only)
git reset --soft HEAD~1              # Undo commit, keep changes staged
git reset --mixed HEAD~1             # Undo commit, unstage changes  
git reset --hard HEAD~1              # Undo commit, lose all changes
git reset abc123                     # Reset to specific commit

# Revert commits (safe for shared repos)
git revert HEAD                      # Revert last commit
git revert abc123                    # Revert specific commit
git revert HEAD~3..HEAD              # Revert multiple commits

# Discard working changes
git checkout -- filename.txt         # Discard file changes
git restore filename.txt             # Modern syntax
git checkout .                       # Discard all changes
git clean -fd                        # Remove untracked files/directories
```

**🎯 Use Cases:**
- `git reset --soft`: Fix commit message or add forgotten files
- `git reset --hard`: Completely abandon current changes
- `git revert`: Undo changes on shared/public branches
- `git clean -fd`: Remove build artifacts, temporary files

**⚠️ When NOT to use:**
- `git reset --hard` without backup on important changes
- `git reset` on commits that have been pushed to shared branches

---

## 📦 Stashing

### **When to use:** Temporarily saving work, switching contexts

```bash
# Basic stashing
git stash                            # Stash current changes
git stash push -m "WIP: user feature" # Stash with message
git stash -u                         # Include untracked files
git stash -a                         # Include all files (even ignored)

# Managing stashes
git stash list                       # List all stashes
git stash show                       # Show latest stash changes
git stash show stash@{1}             # Show specific stash

# Applying stashes
git stash pop                        # Apply and remove latest stash
git stash apply                      # Apply without removing
git stash apply stash@{2}            # Apply specific stash
git stash branch new-feature stash@{1} # Create branch from stash

# Cleaning stashes
git stash drop                       # Delete latest stash
git stash drop stash@{1}             # Delete specific stash
git stash clear                      # Delete all stashes
```

**🎯 Use Cases:**
- Quick branch switch with uncommitted changes
- Pull latest changes while working on features
- Experiment with different approaches
- Save work before risky operations

---

## 🏷️ Tags & Releases

### **When to use:** Marking release points, versioning

```bash
# Create tags
git tag v1.0.0                       # Lightweight tag
git tag -a v1.0.0 -m "Release 1.0"   # Annotated tag (recommended)
git tag -a v1.0.1 abc123 -m "Hotfix" # Tag specific commit

# List and show tags
git tag                              # List all tags
git tag -l "v1.*"                    # List tags matching pattern
git show v1.0.0                     # Show tag information

# Push tags
git push origin v1.0.0               # Push specific tag
git push origin --tags               # Push all tags
git push --follow-tags               # Push commits and reachable tags

# Delete tags
git tag -d v1.0.0                    # Delete local tag
git push origin --delete tag v1.0.0  # Delete remote tag
```

**🎯 Use Cases:**
- Software releases and versions
- Marking stable points in development
- Creating release notes
- Rollback references

---

## 🔥 Advanced Operations

### **When to use:** Complex scenarios, cleanup, advanced workflows

```bash
# Interactive rebase (rewrite history)
git rebase -i HEAD~3                 # Edit last 3 commits
git rebase -i main                   # Rebase onto main
git rebase --continue                # Continue after resolving conflicts
git rebase --abort                   # Cancel rebase

# Cherry-pick commits
git cherry-pick abc123               # Apply specific commit
git cherry-pick abc123..def456       # Apply range of commits
git cherry-pick -n abc123            # Apply without committing

# Bisect (find bugs)
git bisect start                     # Start bisect session
git bisect bad HEAD                  # Mark current as bad
git bisect good v1.0.0               # Mark tag as good
git bisect reset                     # End bisect session

# Reflog (recovery)
git reflog                           # Show reference log
git reflog expire --expire=90.days.ago --all # Clean old entries
git reset --hard HEAD@{5}            # Recover using reflog

# Submodules
git submodule add <url> path/to/sub  # Add submodule
git submodule update --init --recursive # Initialize submodules
git submodule foreach git pull origin main # Update all submodules
```

**🎯 Use Cases:**
- `git rebase -i`: Clean up commit history before merging
- `git cherry-pick`: Apply hotfixes to multiple branches
- `git bisect`: Find which commit introduced a bug
- `git reflog`: Recover "lost" commits

---

## 🆘 Emergency Commands

### **When to use:** You messed up and need to fix things quickly

```bash
# "Oh no!" scenarios
git reflog                           # See everything you've done
git reset --hard HEAD@{3}           # Go back to previous state
git fsck --lost-found               # Find dangling commits

# Forgot to add files to commit
git add forgotten-file.txt
git commit --amend --no-edit

# Wrong commit message
git commit --amend -m "Correct message"

# Committed to wrong branch
git log --oneline -n 1               # Note the commit hash
git reset --hard HEAD~1             # Remove from current branch
git checkout correct-branch
git cherry-pick <commit-hash>        # Apply to correct branch

# Pushed sensitive data
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch secrets.txt' \
  --prune-empty --tag-name-filter cat -- --all
git push origin --force --all        # ⚠️ Dangerous!

# Merge conflicts
git status                           # See conflicted files
# Edit files to resolve conflicts
git add .                            # Stage resolved files
git commit                           # Complete merge
# OR
git merge --abort                    # Cancel merge
```

**🎯 Emergency Toolkit:**
1. `git reflog` - Your safety net
2. `git stash` - Quick save before trying fixes
3. `git reset --hard HEAD@{1}` - Undo last operation
4. `git branch backup-$(date +%s)` - Create backup before risky operations

---

## 🔄 Workflow Examples

### Feature Branch Workflow

```bash
# Start new feature
git checkout main
git pull origin main
git checkout -b feature/user-authentication

# Work on feature
# ... make changes ...
git add .
git commit -m "Add login form"
git commit -m "Add password validation"
git push -u origin feature/user-authentication

# Prepare for merge
git checkout main
git pull origin main
git checkout feature/user-authentication
git rebase main                      # Optional: keep linear history

# Merge feature
git checkout main
git merge --no-ff feature/user-authentication
git push origin main
git branch -d feature/user-authentication
git push origin --delete feature/user-authentication
```

### Hotfix Workflow

```bash
# Emergency fix
git checkout main
git pull origin main
git checkout -b hotfix/security-patch

# Make fix
git add .
git commit -m "Fix security vulnerability"
git push -u origin hotfix/security-patch

# Deploy fix
git checkout main
git merge hotfix/security-patch
git tag -a v1.0.1 -m "Security hotfix"
git push origin main --follow-tags
```

### Open Source Contribution

```bash
# Fork and clone
git clone https://github.com/yourusername/project.git
cd project
git remote add upstream https://github.com/originalowner/project.git

# Create feature branch
git checkout -b feature/improvement
# ... make changes ...
git add .
git commit -m "Improve documentation"
git push origin feature/improvement

# Keep fork updated
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

---

## ✅ Best Practices

### Commit Messages
```bash
# Good commit messages
git commit -m "Add user authentication endpoint"
git commit -m "Fix memory leak in image processor"
git commit -m "Update dependencies to latest versions"

# Bad commit messages
git commit -m "fix"
git commit -m "changes"
git commit -m "update stuff"
```

### Branching Strategy
```bash
# Use descriptive branch names
git checkout -b feature/user-profile
git checkout -b bugfix/login-error
git checkout -b hotfix/security-patch
git checkout -b chore/update-dependencies

# Avoid generic names
git checkout -b temp
git checkout -b test
git checkout -b fix
```

### Safety First
```bash
# Always check status before operations
git status

# Create backups before risky operations
git branch backup-before-rebase

# Use safer alternatives
git push --force-with-lease          # Instead of --force
git pull --rebase                    # Instead of creating merge commits
```

### Performance Tips
```bash
# Shallow clone for large repos
git clone --depth 1 <url>

# Fetch specific branches
git fetch origin main:main

# Clean up regularly
git gc                               # Garbage collection
git prune                           # Remove unreachable objects
```

---

## 🎯 Quick Reference

### Most Used Commands (80/20 rule)
```bash
git status                          # Check current state
git add .                           # Stage all changes
git commit -m "message"             # Commit changes
git push                            # Push to remote
git pull                            # Get latest changes
git checkout -b branch-name         # Create and switch branch
git merge branch-name               # Merge branch
git log --oneline                   # View history
```

### Command Aliases (Add to ~/.gitconfig)
```bash
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    ca = commit -am
    lg = log --oneline --graph --all
    unstage = reset HEAD --
    last = log -1 HEAD
    visual = !gitk
```

---

## 📚 Additional Resources

- [Pro Git Book](https://git-scm.com/book) - Comprehensive Git guide
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf) - Quick reference
- [Interactive Git Tutorial](https://learngitbranching.js.org/) - Visual learning
- [Git Workflows](https://www.atlassian.com/git/tutorials/comparing-workflows) - Team strategies

---

## 🤝 Contributing

Found an error or want to add more scenarios? Contributions are welcome!

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/new-scenario`)
3. Commit your changes (`git commit -am 'Add new emergency scenario'`)
4. Push to the branch (`git push origin feature/new-scenario`)
5. Open a Pull Request

---

## 📄 License

This guide is released under the [MIT License](LICENSE).

---

⭐ **Star this repository** if it helped you master Git!

📢 **Share** with developers who are learning Git!

🐛 **Report issues** if you find any problems or have suggestions!