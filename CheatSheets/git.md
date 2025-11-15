# Git Cheat Sheet - Complete Reference

## 🔧 Configuration

### Initial Setup

```bash
# Configure user
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Configure editor
git config --global core.editor "nano"
git config --global core.editor "code --wait"  # VS Code

# Configure default branch
git config --global init.defaultBranch main

# View configuration
git config --list
git config --global --list
git config user.name
git config user.email

# Set aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
```

## 📦 Repository Management

### Creating Repositories

```bash
# Initialize repository
git init
git init --bare                    # Bare repository
git init --template=template-dir   # With template

# Clone repository
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my-folder
git clone --branch branch-name https://github.com/user/repo.git
git clone --depth 1 https://github.com/user/repo.git  # Shallow clone
git clone --recursive https://github.com/user/repo.git  # With submodules
```

### Repository Status

```bash
# Check status
git status
git status -s                      # Short format
git status --short
git status --porcelain             # Machine-readable
git status --ignored               # Show ignored files
```

## 📝 Making Changes

### Staging Files

```bash
# Add files
git add file.txt                   # Add specific file
git add .                          # Add all changes
git add *.txt                      # Add all .txt files
git add -A                         # Add all (including deletions)
git add -u                         # Update tracked files
git add -p                          # Interactive patch

# Unstage files
git reset HEAD file.txt
git restore --staged file.txt      # Newer command
git reset HEAD .                   # Unstage all
```

### Committing

```bash
# Commit changes
git commit -m "Commit message"
git commit -m "Title" -m "Description"
git commit -am "Quick commit"       # Add and commit
git commit --amend                  # Amend last commit
git commit --amend -m "New message"
git commit --amend --no-edit       # Amend without changing message

# Commit with sign-off
git commit -s -m "Signed commit"

# Empty commit
git commit --allow-empty -m "Empty commit"
```

### Viewing Changes

```bash
# View changes
git diff                            # Working directory vs staged
git diff --staged                   # Staged vs last commit
git diff HEAD                       # Working directory vs HEAD
git diff branch1..branch2          # Between branches
git diff commit1..commit2          # Between commits

# View file at specific commit
git show commit:path/to/file
git show HEAD:path/to/file

# Diff options
git diff --stat                     # Summary
git diff --name-only               # File names only
git diff --color-words             # Word-level diff
git diff -w                         # Ignore whitespace
```

## 🌿 Branching

### Creating and Switching

```bash
# List branches
git branch                          # Local branches
git branch -a                       # All branches
git branch -r                       # Remote branches
git branch -v                       # With last commit
git branch --merged                 # Merged branches
git branch --no-merged              # Not merged

# Create branch
git branch branch-name
git branch branch-name commit-hash  # From specific commit
git checkout -b branch-name        # Create and switch
git switch -c branch-name           # Newer command

# Switch branch
git checkout branch-name
git switch branch-name              # Newer command

# Delete branch
git branch -d branch-name           # Safe delete
git branch -D branch-name          # Force delete
git branch -d -r origin/branch-name # Delete remote tracking
```

### Branch Operations

```bash
# Rename branch
git branch -m old-name new-name
git branch -m new-name             # Rename current branch

# Track remote branch
git branch --set-upstream-to=origin/branch-name branch-name
git branch -u origin/branch-name branch-name

# Show branch info
git branch -vv                     # With tracking info
```

## 🔄 Merging and Rebasing

### Merging

```bash
# Merge branch
git merge branch-name
git merge branch-name --no-ff      # No fast-forward
git merge branch-name --squash     # Squash merge
git merge --abort                  # Abort merge

# Merge strategies
git merge branch-name -s ours      # Ours strategy
git merge branch-name -s theirs    # Theirs strategy
```

### Rebasing

```bash
# Rebase branch
git rebase branch-name
git rebase origin/main

# Interactive rebase
git rebase -i HEAD~3                # Last 3 commits
git rebase -i commit-hash          # From commit
git rebase -i --root                # From beginning

# Rebase operations
git rebase --continue               # Continue after resolving
git rebase --abort                  # Abort rebase
git rebase --skip                   # Skip commit

# Rebase onto
git rebase --onto newbase oldbase branch
```

## 📜 History

### Viewing History

```bash
# Commit history
git log
git log --oneline                  # One line per commit
git log --graph                    # With graph
git log --all --graph --oneline    # All branches
git log --decorate                 # Show refs

# Filtered history
git log --author="Name"
git log --since="2024-01-01"
git log --until="2024-12-31"
git log --grep="pattern"
git log -S "function_name"         # Search in code
git log -- path/to/file            # File history
git log --follow path/to/file     # Follow renames

# Format history
git log --pretty=format:"%h - %an, %ar : %s"
git log --pretty=fuller
git log --stat                     # With stats
git log -p                          # With patches
git log --name-only                # File names only
git log --name-status              # File status

# Limit history
git log -n 10                      # Last 10 commits
git log --max-count=10
git log --skip=5 -n 10             # Skip 5, show 10
```

### Viewing Commits

```bash
# Show commit
git show commit-hash
git show HEAD                       # Latest commit
git show branch-name               # Latest on branch
git show HEAD~1                    # Previous commit
git show HEAD^                     # Parent commit
git show HEAD~2                    # 2 commits back

# Show file changes
git show commit-hash:path/to/file
git show commit-hash --stat        # Summary
git show commit-hash -p            # With patches
```

## 🔀 Remote Operations

### Managing Remotes

```bash
# List remotes
git remote
git remote -v                      # With URLs
git remote show origin             # Detailed info

# Add remote
git remote add origin https://github.com/user/repo.git
git remote add upstream https://github.com/original/repo.git

# Remove remote
git remote remove origin
git remote rm origin

# Rename remote
git remote rename old-name new-name

# Update remote URL
git remote set-url origin https://github.com/user/repo.git
```

### Fetching and Pulling

```bash
# Fetch changes
git fetch
git fetch origin
git fetch origin branch-name
git fetch --all                    # All remotes
git fetch --prune                  # Remove deleted remote branches

# Pull changes
git pull
git pull origin branch-name
git pull --rebase                  # Rebase instead of merge
git pull --ff-only                 # Fast-forward only
```

### Pushing

```bash
# Push changes
git push
git push origin branch-name
git push -u origin branch-name     # Set upstream
git push --set-upstream origin branch-name

# Push options
git push --force                   # Force push (dangerous!)
git push --force-with-lease        # Safer force push
git push --all                     # All branches
git push --tags                    # All tags
git push origin --delete branch-name  # Delete remote branch
```

## 🏷️ Tags

### Managing Tags

```bash
# List tags
git tag
git tag -l "v1.*"                  # Filter tags
git tag -l --sort=-version:refname # Sorted

# Create tag
git tag v1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"  # Annotated
git tag v1.0.0 commit-hash         # Tag specific commit

# Show tag
git show v1.0.0
git tag -v v1.0.0                  # Verify GPG signature

# Delete tag
git tag -d v1.0.0                  # Local
git push origin --delete v1.0.0    # Remote
git push origin :refs/tags/v1.0.0   # Alternative

# Push tags
git push origin v1.0.0
git push --tags                    # All tags
git push origin --tags
```

## 🔄 Undoing Changes

### Unstaging

```bash
# Unstage file
git reset HEAD file.txt
git restore --staged file.txt      # Newer command
```

### Discarding Changes

```bash
# Discard working directory changes
git checkout -- file.txt
git restore file.txt               # Newer command
git restore .                       # All files

# Discard all changes
git reset --hard HEAD
git clean -fd                       # Remove untracked files
```

### Resetting

```bash
# Reset to commit
git reset --soft HEAD~1            # Keep changes staged
git reset --mixed HEAD~1           # Keep changes unstaged (default)
git reset --hard HEAD~1            # Discard all changes (dangerous!)

# Reset to specific commit
git reset --hard commit-hash
git reset --hard origin/main       # Reset to remote
```

### Reverting

```bash
# Revert commit (creates new commit)
git revert commit-hash
git revert HEAD                     # Revert last commit
git revert --no-commit commit-hash  # Don't auto-commit
```

## 💾 Stashing

### Stash Operations

```bash
# Save changes
git stash
git stash save "Work in progress"
git stash push -m "Message"
git stash -u                        # Include untracked files
git stash -a                        # Include ignored files

# List stashes
git stash list

# Show stash
git stash show
git stash show stash@{0}
git stash show -p stash@{0}        # With patches

# Apply stash
git stash apply
git stash apply stash@{0}
git stash pop                      # Apply and remove
git stash pop stash@{0}

# Drop stash
git stash drop stash@{0}
git stash clear                    # Remove all stashes

# Create branch from stash
git stash branch branch-name stash@{0}
```

## 🔍 Searching

### Finding Commits

```bash
# Search in commits
git log --grep="pattern"
git log --grep="pattern" -i        # Case insensitive
git log -S "function_name"         # When function added/removed
git log -G "pattern"               # Regex search

# Find when bug introduced
git bisect start
git bisect bad                      # Current commit is bad
git bisect good commit-hash        # This commit was good
git bisect reset                    # End bisect

# Blame
git blame file.txt                  # Who changed what
git blame -L 10,20 file.txt        # Specific lines
git blame -w file.txt              # Ignore whitespace
```

### Finding Files

```bash
# Find file in history
git log --all --full-history -- path/to/file
git log --all -- path/to/file

# Find when file was deleted
git log --all --full-history --diff-filter=D -- path/to/file
```

## 🔐 Security

### GPG Signing

```bash
# Configure GPG
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true

# Sign commits
git commit -S -m "Signed commit"
git commit -S --amend

# Sign tags
git tag -s v1.0.0 -m "Signed tag"
git tag -u YOUR_KEY_ID v1.0.0

# Verify signatures
git log --show-signature
git tag -v v1.0.0
```

### SSH Keys

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"

# Add to SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Test connection
ssh -T git@github.com
```

## 🧹 Cleanup

### Removing Files

```bash
# Remove file from Git
git rm file.txt
git rm --cached file.txt           # Keep local file
git rm -r directory/               # Remove directory
git rm '*.log'                     # Remove by pattern
```

### Pruning

```bash
# Prune remote branches
git remote prune origin
git fetch --prune

# Clean untracked files
git clean -n                        # Dry run
git clean -f                        # Force
git clean -fd                       # Include directories
git clean -x                        # Include ignored files
```

## 📊 Statistics

### Repository Stats

```bash
# Commits per author
git shortlog
git shortlog -sn                   # Summary with counts

# Lines changed
git log --stat
git log --numstat                  # Numbers only
git diff --stat                    # Diff stats

# File statistics
git ls-files | xargs wc -l         # Total lines
git diff --shortstat               # Short stats
```

## 🎯 Advanced

### Submodules

```bash
# Add submodule
git submodule add https://github.com/user/repo.git path

# Initialize submodules
git submodule init
git submodule update
git submodule update --init --recursive

# Update submodules
git submodule update --remote
```

### Worktrees

```bash
# Create worktree
git worktree add ../branch-name branch-name
git worktree list
git worktree remove branch-name
```

### Hooks

```bash
# List hooks
ls .git/hooks/

# Install hooks
# Edit .git/hooks/pre-commit
# Make executable: chmod +x .git/hooks/pre-commit
```

## 📝 Common Workflows

### Feature Branch

```bash
git checkout -b feature/new-feature
# Make changes
git add .
git commit -m "Add feature"
git push -u origin feature/new-feature
# Create PR, merge
git checkout main
git pull
git branch -d feature/new-feature
```

### Hotfix

```bash
git checkout -b hotfix/critical-bug main
# Fix bug
git add .
git commit -m "Fix critical bug"
git checkout main
git merge hotfix/critical-bug
git tag -a v1.0.1 -m "Hotfix release"
git push origin main --tags
```

### Squash Commits

```bash
git rebase -i HEAD~5
# Change 'pick' to 'squash' for commits to squash
# Edit commit message
git push --force-with-lease
```

## 🚨 Troubleshooting

### Common Issues

```bash
# Merge conflicts
git status                          # See conflicts
# Edit conflicted files
git add resolved-file.txt
git commit

# Detached HEAD
git checkout branch-name           # Return to branch

# Lost commits
git reflog                         # Find lost commits
git checkout commit-hash          # Recover

# Wrong commit message
git commit --amend -m "New message"
```

## 📚 Quick Reference

### Common Aliases

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
git config --global alias.lg "log --oneline --decorate --all --graph"
```

### File States

```
Untracked → Staged → Committed
     ↓         ↓
  Modified  Modified
```

---

**Remember**: Git is powerful. Always review before force pushing. Use `--dry-run` when available. Keep commits atomic and messages clear.
