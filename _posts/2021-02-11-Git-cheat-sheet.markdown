---
layout: post
title:  "Git: Cheat Sheet for git commands"
date:   2021-02-11 19:00:00 +0100
categories: git
---

In this post I compile useful commands for using git.

# Initialize a new repo

In a new repository: 
```
git init
git remote add origin https://github.com/<userName>/<repoName>.git
# for instance: 
#git remote add origin https://github.com/ChloeVincent/French-new-quotatives.git
```
After some local changes: 
```
git commit -m "Initial commit"
git push -u origin main
```
-u origin main is to make main the default branch

# Normal work

When working on a project, usual commands are `git pull` to pull any changes from the remote repository (if there was changes made by another user/dev, or changes made on github like adding a README for instance).

`git status` gives you the current state of your repository (what files are changed, added, deleted)

`git diff` or `git diff <file>` will show you the differences between your current working file(s) and the last version committed (very useful to check what you are about to commit)

`git add .` takes all changes to prepare them for a commit (you can specify which files you want to add instead of ".")

In case some files are added incorrectly, you can "unstage" the file so that it won't be in the next commit with the command `git restore --stagged <file>`. Be careful `git restore <file>` on an unstaged file (=before add) will restore it to the previous added version.

`git commit -m "Some changes I made"` commits the changes that were previously added with the message given after -m (if no message is given a window will open to ask for it) 

`git push` will push the modification to the remote repository