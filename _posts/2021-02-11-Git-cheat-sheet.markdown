---
layout: post
title:  "Git: Cheat Sheet for git commands"
date:   2021-02-11 12:00:00 +0100
categories: git
---

In this post I compile useful commands for using git.

# Initialize a new repo

In a new repository: 
```sh
git init
git remote add origin https://github.com/<userName>/<repoName>.git
# for instance: 
#git remote add origin https://github.com/ChloeVincent/French-new-quotatives.git
```
After some local changes: 
```sh
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

`git commit -m "Some changes I made"` commits the changes that were previously added with the message given after -m (if no message is given a window will open to ask for it). 
Use `-a` to automatically add the changes on files already in git.

`git push` will push the modification to the remote repository

# Branches
To create a new branch: 
```sh
git checkout -b <newBranchName>
```

# Visualizing git tree
`git log` shows the last commits. 

To get a more user-friendly visualization use `gitk`.

# Changing history (not recommended)
To delete the last 4 commits use:
```sh
git reset --hard HEAD~4
git push -f
```
The first line resets the working directory 4 commits behind, and the second line forces the change to the server.
As a general rule of thumb, it is not a goot idea to force push anything since that's where errors come from.

To squash multiple commits use:
```sh
git rebase -i <hash of the commit you want to rebase on>
```
You can also use `HEAD~5`, for instance, which will squash the 4 last commits in a new one coming right after `HEAD~5`.
The option `-i` is for interactive: a editor window will open asking for directive on how to rebase. 
To squash, you need to `pick` the first commit (aka leave the line as it is) and squash the next commits (aka replace "pick" at the start of the line with "squash" or simply "s").
Instructions will be available in the comments of the editor window.

## Initialize a new repo on a server ssh

Connect to your ssh server, by running the following command in a terminal: 
```
ssh <username>@<server address>
```
The server address looks something like `ssh....hosting.ovh.net`.
It will prompt for the password.

On the server, create a new bare repo git:
```
mkdir myRepoName
cd myRepoName
git --bare init
```

In your local environment (another terminal), clone your newly created repo with: 
```
git clone <username>@<server address>:myRepoName (localName)
```
The local name is optional, but can be useful.

Once your local repo is created, you can add modifications, commit and push.

In order to update the files on the server that are accessible through the website, there are multiple solutions. 

First solution is to simply clone the bare repo on the server, on the same level as the bare:
```
git clone myRepoName myWebsiteFolder
```

The problem with this solution is that myWebsiteFolder will be a working git repository, and thus the `.git` will be accessible through the website. 
Another problem is that we need to manually update the website everytime we push from our local repo (by pulling from the working repo on the server).

### Using hooks
Hooks allow us to run script automatically (see `man git hooks`).
We will create a `post-receive` hook, which will run everytime a push is received by the server (aka someone pushes their local work to the server).

On the server, in the bare repo, there is a folder "hooks" which contains multiple samples of possible hooks. 
I copied the `post-update.sample` to `post-receive` and replaced the example line with the following:
```
git --work-tree <website folder path> checkout --force
```

`<website folder path>` must already exist !
The option `--work-tree` allows to have a working directory separated from the `.git`, which means it wont be accessible on the website.
The option `--force` forces the modifications to be apply despite the warning that it will modify the files.

In order to modify the file `post-receive`, I used the text editor nano which was available on the server.
It is a command line style editor, so no mouse available, use the arrows and ctrl+O to save, crtl+x to exit.

In order to get the website folder path, I used the command `pwd`.
If nano is already opened, use ctrl+z to put the process in the background, `pwd`, copy the folder path and `fg` to return to the process in the background: nano.

Both problems mentioned earlier are solved: everytime someone pushes their changes on the server, the website will be automatically modified and the git repo will not be accessible from the  website. 

For a better understanding, on the server, in the "bare repo" (myRepoName), one can run the following command:
```
git --work-tree ../websiteFolder status
``` 
Without the `--work-tree` option, git does not accept the command since bare is not a working repository.

In the website folder (websiteFolder), one can run the following command:
```
git --git-dir ../myRepoName --work-tree . status
```
Without any option, git says this is not a git repo and no `.git` can be found in the parent folders.
`--git-dir` option tells git where to look for the `.git`. 
`--work-tree` option is still needed, otherwise git does not accept the command because it does not reconize website folder as a working repository.