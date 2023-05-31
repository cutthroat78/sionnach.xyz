---
title: "Git"
---

# Commands/How Tos

## Fix ```error: Your local changes to the following files would be overwritten by merge``` and ```error: Untracked working tree file 'images/icon.png' would be overwritten by merge``` Errors

1. 
  - Save Local Changes to Use at a Later Day: ```git stash --include-untracked```
  - Discard Local Changes: ```git reset --hard```
    - Discard Untracked/New Files: ```git clean -fd```
2. Git Pull: ```git pull``` or use Alias: ```gl```

## Create a Bare Repo (A Repo to be used Remotely)

```git init --bare```

## Git Clone a Specific Branch in a Repo

```git clone -b <branch> <remote_repo>```
