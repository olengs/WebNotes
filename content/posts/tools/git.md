---
title: "Git CLI"
summary: "Learnings for git cli"
date: 2025-08-23
tags: ["Git", "Github"]
author: ["JC"]
draft: false
weight: 0
ShowToc: true
---

### Summary

My learning for git and github CLI.

### Git Core Commands
```
# Adds file listed changes into staged status
git add <filename> 

# Adds all files with extension
git add *.txt

# Adds all files from current folder into staged status
git add . 

# Checks the current status of the working repo
git status 

# Remove all files from staging
git reset
#removes specific file from staging
git reset <filename>

# Commits the staged changes to the local repo
git commit

# Add a message to the commit command
git commit -m <message>

# Moodifies the most recent commit message

# Push to github servers
git push

# Pull from github servers
git pull

```

### Logging

These functions are to log and see it in a graph

```
# filters and find out when a file is removed
# D => deleted
git log --diff-filter=D -- <filename>
```

### Stashing
The functions are used for stashing purposes. To be used when saving but doesn't want to commit. Note: If there are conflicts, it will still pop the stack but with the conflict headers.
```
# Add to stash stack
git stash push

# Remove from stash stack and merges with existing
git stash pop

# View stash stack
git stash list

# Remove certain block from stash
git stash apply <stash block name>
```

### Branching
Creating branch and managing branches. All branches including history have an id reference, where you can point the "head pointer" point to, where the head pointer is where your local repo is looking at
```
# View branch
git checkout <id>
git checkout <branch-name>

# View all branches (the highlight in red are remote only)
git branch -a

# Create a new branch
git branch <branch_name>

# 

```