# Lab: Git Merge Conflict Challenge

## 🎯 Scenario
Two developers modified the same line in the same file. You need to merge their changes without losing code.

## 🛠️ Setup
```bash
# Create repo
mkdir conflict-lab && cd conflict-lab
git init
echo "Line 1" > file.txt
git add . && git commit -m "Initial"

# Branch A
git checkout -b feature-a
echo "Line 1 modified by A" > file.txt
git commit -am "Feature A"

# Branch B
git checkout main
git checkout -b feature-b
echo "Line 1 modified by B" > file.txt
git commit -am "Feature B"
```

## ⚔️ The Conflict

```bash
git checkout main
git merge feature-a
# Fast-forward

git merge feature-b
# CONFLICT (content): Merge conflict in file.txt
```

## 🕵️ Resolution Steps

### 1. Check Status
```bash
git status
# Output: both modified: file.txt
```

### 2. Inspect File
```bash
cat file.txt
<<<<<<< HEAD
Line 1 modified by A
=======
Line 1 modified by B
>>>>>>> feature-b
```

### 3. Resolve
Decide: Keep A? Keep B? Or Keep Both?
**Goal**: Keep Both.
Edit `file.txt`:
```text
Line 1 modified by A
Line 1 modified by B
```

### 4. Commit
```bash
git add file.txt
git commit -m "Resolved conflict: kept both changes"
```

## ✅ Success Criteria
- [ ] `git status` is clean.
- [ ] `file.txt` contains desired content.
- [ ] History shows the merge commit.
