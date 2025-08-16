---
title: "Git"
type: "docs"
weight: 2
---

Git is a distributed version control system where each client will have a local repository (.git directory), allowing commits without network and pushing to server when network recovers.

![git](git.png)

## Repository

```shell
# Enter repository directory, initialize empty repository
cd project && git init

# Initialize bare repository.
# No working directory or staging area, only repository, saves space, suitable for server.
git init --bare

# Create project.git directory and initialize as bare repository.
# Recommended to name server repository folders with .git suffix.
git init --bare project.git

# Clone repository to local, http method
git clone https://git.test/username/project.git

# Clone repository to local, ssh method
git clone git@git.test:username/project.git

# View repository settings
git config --list

# Set global git username
git config --global user.name "name"

# Set global git email
git config --global user.email "test@test.com"`

# Force use ssh
git config --global url."git@github.com:".insteadOf "https://github.com/"

# Modify remote repository address
git remote set-url origin https://git.test/username/project2.git

# Modify remote repository address, SSH access method
git remote set-url origin git@git.test:username/project2.git

# Modify remote repository address, with username password, no need to enter password for subsequent operations
git remote set-url origin https://username:password@git.test/username/project2.git
```

### Multi-repository SSH Configuration

```shell
# Configure ssh, file content below
vi ~/.ssh/config

# Clone github repository to local
git clone github:username/project.git

# Clone test repository to local
git clone test:username/project.git

# Modify remote repository address, access Git service through private key
git remote set-url origin github:username/project2.git
```

```text
# Github
Host github    # Abbreviation
  HostName github.com    # Server address
  User git    #ssh user
  IdentityFile ~/.ssh/id_rsa_github    # Private key

# My Server
Host test
 HostName 127.0.0.1
 User git
 IdentityFile ~/.ssh/id_rsa_lynnclub
```

## Branches

```shell
# View local branches
# Recommend developing habit of frequently checking current branch to avoid operating wrong branch and causing trouble.
git branch

# View local and remote branches
git branch -a

# Check out local branch
# If doesn't exist locally but exists remotely, will check out remote branch to local
git checkout feature_test

# Create new branch based on current branch
git branch feature_test

# Create new branch and switch to new branch
git checkout -b feature_test

# Push new branch to remote server
git push --set-upstream origin feature_test

# Rename branch
git branch -m feature_test feature_rename

# Merge feature_test to current branch
git merge feature_test

# Cancel just-performed merge operation, invalid if already committed
git merge --abort

# Delete local branch
# Must checkout to other branch before deleting this branch
git branch -d feature_test

# Delete remote branch
git push origin --delete feature_test
# Or
git push origin :feature_test
```

## File Operations

```shell
# Display modified file list
git status

# View file modification differences
git diff a.php
# View all file modification differences
git diff .

# Discard working directory modifications
# Supports wildcards (e.g., \*.php)
# Dangerous operation, cannot recover after checkout. Recommend using stash for temporary storage, so after mistaken checkout, can still recover
git checkout a.php

# Move file or directory
git mv a.php b.php

# Delete file
git rm b.php

# Add file or directory to staging area, supports wildcards
git add a.php

# Commit staging area to local repository
# Must include -m description
# Recommend each commit has one and only one complete function. Frequent scattered commits not conducive to viewing records or rollback operations.
git commit -m 'test'

# Push to remote repository branch corresponding to local branch
git push

# View commit records
git log
```

### stash

```shell
# Temporarily store modifications, move modifications from working directory to stack
# Suitable for saving modifications not convenient to add to staging area
git stash

# Temporarily store modifications with notes
git stash save "test"

# View temporary storage list
git stash list

# Pop temporary storage content, restore modifications to working directory
git stash pop

# Pop specified temporary storage from stack, 1 means pop second one
git stash pop stash@{1}
```

## tag

```shell
# Create new tag
git tag 1.0

# Create annotated tag
git tag -a 1.0 -m "my tag"

# View tag detailed information
git show 1.0

# Push tag to remote
git push origin 1.0

# Delete local repository tag
git tag -d 1.0

# Delete remote repository tag
git push origin :refs/tags/1.0
```

## Rollback, Reset, Recovery

```shell
# Rollback specific commit
git revert 326fc9f70d022afdd31b0072dbbae003783d77ed

# Force reset HEAD to specified version
# Dangerous operation, use carefully
git reset 326fc9f70d022afdd31b0072dbbae003783d77ed

# Move back HEAD position, can be used for rollback
# HEAD points to last update of current branch
# 2 ^ symbols mean move back two commits
git reset HEAD^^

# 100 means move back one hundred commits
git reset HEAD~100

# Move back HEAD repository
git reset --soft HEAD^^

# Move back HEAD repository and staging area, mixed is default parameter
git reset --mixed HEAD^^

# Move back HEAD repository, staging area, and working directory
# Use with caution
git reset --hard HEAD^^

# Pull staging area files back to working directory
git reset HEAD -- <file>

# Change branch nodes, not commonly used
git rebase master

# Records every operation, even reset commits are recorded in reflog
git reflog

# Find lost data from database, can recover stash popped and mistakenly checked out lost data
git fsck --lost-found
```
