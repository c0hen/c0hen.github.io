---
layout: default
title: Git handbook
description: Git handbook for a begginner, by a git beginner.
tags: git beginner tips
---

* Table of contents
{:toc}

# Git

Thoroughly assemble your .gitignore - files excluded from the remote repository.
SSH keys can be used to access remote git repositories.

**Try to use small changes/commits and concise messages.**

HEAD and branches, including master, can be pictured as labels on an object. Git commands make more sense when looking at git from the inside.
A good talk on git internals by [Michael Schwern at Linux.conf.au 2013](https://www.youtube.com/watch?v=eQFZ_MPTVpc).

## Git brisk start

It's easiest to experiment on the local system.
```sh
mkdir -p ~/git/myrepo
cd ~/git/myrepo
git init
cd /tmp/
git clone ~/git/myrepo
cat /tmp/myrepo/.git/config
```
```
*** snip ***
[remote "origin"]
	url = /home/user1/git/myrepo/
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```
Add a remote.
```sh
mkdir -p /media/backup/git/myrepo/
git init /media/backup/git/myrepo/
git remote add backup_repo /media/backup/git/myrepo/
cd /tmp/myrepo
touch hello
git add hello
git commit -m'added greetings'
```
Since `backup_repo` already contains the master branch, it needs another before we can push master in /tmp/myrepo to it.
```sh
git branch try_push
git checkout try_push
git branch
git push backup_repo try_push
cd /media/backup/git/myrepo/
git checkout try_push
cd /tmp/myrepo
git push crucial master
```

### [Terms](https://git-scm.com/docs/gitglossary)

- master - the first branch created by default with git init, not special
- HEAD - the pointer to the branch you're currently on (have checked out)
- commit - saved state with an ID consisting of a SHA-1 checksum of all information of the state. Immutable constant - repo breaks if this is changed. Git does not change history unless forced.
- reference - a commit ID, label or tag
- worktree - directory that contains a branch to work on, tree of actual checked out files
- index - files tracked by git, .git/index (holds a snapshot of the content of the working tree, this is taken as the contents of the next commit)
- staging - preparing files for the next commit in index (not yet part of the commit)

### Start or get a repository

```sh
git init myrepo
git clone https://github.com/c0hen/c0hen.github.io
```

### Very basic flow with remote

git pull is git fetch && git merge

```sh
git pull

ls .
work.txt work.md work.tex

git log
git status
touch legionkitteh
git add .
git status
git commit -a -m'Short commit message committing all changes'
echo "It\'s dangerous to go alone! Take this." >> legionkitteh
touch newidea
git status
git add legionkitteh
git status
git commit --amend -m'Create kitteh'
git status
git add newidea
git commit -m'Create new idea'
git status
git push
```

```sh
git checkout thing maybe_more_things
```
means "switch to", "make active in current context", for example get a file from the status of a commit Bravo and put it into the git directory as it was at the time of commit Bravo.

#### Display commit messages

```sh
git log
```

#### Visual representation of the repository with messages, show a graph

```sh
git log --graph --decorate --all
```

#### Show diff of a file already added to be commited

```sh
git diff --cached path/to/file
```
Show diff of all staged files and last commit
```sh
git diff --cached
```

#### Reset and checkout file in the state before changes (forget staged change)

```sh
git reset HEAD hello.c
git checkout hello.c
```

#### Remove a file that recently became untracked in .gitignore from repo

```sh
git rm --cached path/to/file
```

#### Move mistakenly commited file back to staging area, don't remove local file

```sh
git reset --soft HEAD~
git reset HEAD path/to/unwanted_file
git commit -c ORIG_HEAD
```

#### Get a copy of a file discarding changes since last $git add

```sh
git checkout path/to/file
```

#### Remove pushed commit from repo 

Resets local files in repo. --hard means check out the change in addition to reset.
Don't do this in public ones! Others may be using it already.

```sh
git reset --hard 40digit_commit_id
git push --force
```

#### Checkout a single file from remote

```sh
git fetch
git checkout origin/master -- path/to/file
```

The fetch will download all the recent changes, but it will not put it in your current checked out code (working area).

The checkout will update the working tree with the particular file from the downloaded changes (origin/master).

#### Checkout file deleted by previous commit

The deleting commit for checking out is retrieved by rev-list. The commit before it is referenced by ^ or ~1 (search "ancestry references" for info) .

```sh
git checkout $(git rev-list -n 1 HEAD -- "path/file")^ -- "path/file"
```

#### Get commits with deleted files and the files deleted

```sh
git log --diff-filter=D --summary
```

#### Rebase when pulling and before pushing

```
git pull --rebase
```

Take care to understand each rebase. Tends to be confusing, I prefer merging `main` profusely and then [squash merging into a (new) finalized branch](#clean-commit-history).
Rebasing allows to merge related commits and keep mistakes from littering the project's history.

```sh
git rebase -i
git push
```

#### Remove a manually deleted file from tree

```sh
git ls-files --deleted -z | xargs -0 git rm
```

## Branches

### New feature testing workflow

#### Create a branch (still on master) and switch to it (on new branch).

```sh
git branch social
git checkout social
```

#### Merge changes from the master branch and run tests

```sh
git merge master
```
Run your tests. Fix as necessary and retest until code ok. Add and commit as needed.

#### Merge and commit branch to master

```sh
git checkout master
git merge social
git commit
```

#### Create a branch and switch to it

```sh
git checkout -b social
```

#### List remote branches

```sh
git branch -r
```

#### Fetch all branches from remote called origin

```sh
git fetch origin
```

#### Pull all tracked branches

```sh
git pull --all
```

### More about branches

#### Show repository log with graph

```sh
git log --all --oneline --graph
```

#### Stash changes not wanted in a commit

```sh
git add *
git stash push -m'CSS' css/
git stash push -m'tests' tests/
git stash push -m'includes' _includes/
git stash list
git checkout -b fix_includes
git stash show
git stash pop stash@{0}
git commit -m'Fix includes'
```
Keep the stashes and apply (copy) the target stash:
```sh
git checkout main
git stash apply 1 # same as stash@{1}
git commit -m'Modify CSS styles'
```
Drop a single stash:
```sh
git stash drop 1
```
Pop stashes to where needed, clear (delete) the rest:
```sh
git stash clear
```

#### Addressing parents

~ always addresses linear parents, on the same branch. 2 commits before HEAD:
```sh
git show HEAD~2
```
In case of branches ^ (think of the caret symbol as branching) allows addressing a different branch, selecting the parent in case of a merge. If feature was merged into master and HEAD is at the merge commit:
```sh
git show HEAD^1~2 # master branch, effectively same
git show HEAD^2~2 # feature branch, 2 commits before HEAD
```
Branch inspection, see what's unique to each parent of a merge.
```sh
git log HEAD^1 --oneline --not HEAD^2
git log HEAD^2 --oneline --not HEAD^1
```
Usage with reset in case of a merge.
```sh
# Undo merge but keep both branches intact
git reset HEAD^1

# Go back before the merge entirely
git reset HEAD~1
```

#### Find upstream of all branches:

```sh
while read branch; do
  upstream=$(git rev-parse --abbrev-ref $branch@{upstream} 2>/dev/null)
  if [[ $? == 0 ]]; then
    echo $branch tracks $upstream
  else
    echo $branch has no upstream configured
  fi
done < <(git for-each-ref --format='%(refname:short)' refs/heads/*)
```

#### Delete a remote branch

```sh
git push --delete origin my_branch
```

### Working with remote repositories

#### Check remote repository before fetching it

```sh
git ls-remote remotename
```

#### Add git remote with SSH URL

Add an existing git remote, check that it was added by listing remotes.
```sh
git remote add remotename gituser@secondary.gitdomain.tld:/path/to/myrepo
git remote -v
```

#### Add git remote with SSH specified key login

Can be set per repository. See man git-config

```sh
cd myrepo
git config core.sshCommand "ssh -i ~/.ssh/id_git -F /dev/null"
```

#### Multiple remotes with multiple ssh keys in one repository - external transport example

Create a bare remote git repository and set up access via ssh, with a per-repo key.
Add write permissions by group for accountable sharing.

On the remote:
```sh
aptitude install git
adduser gituser
addgroup gitgroup
su gituser
cd
git --bare --shared=group init myrepo
chgrp gitgroup myrepo
```

On the machine you have your current latest repo and plan to push from:
```sh
cd myrepo
```
Allow external protocol, ssh transport with separate key.
Only from user initiated commands, not from git initiated automated commands.
See man git-config, one SSH key is simpler and allows avoiding configuring the external ssh transport.
```sh
git config protocol.allow.ext user
```
Copy your ssh key to the secondary.gitdomain.tld, test login.
```sh
ssh-copy-id -i ~/.ssh/id_secondary gituser@secondary.gitdomain.tld
ssh gituser@secondary.gitdomain.tld
exit
```
Add the remote with the short name "secondary" and ssh URL
```sh
git remote add secondary 'ext::ssh -i ~/.ssh/id_secondary gituser@secondary.gitdomain.tld %S /home/gituser/myrepo'
```
List remotes
```sh
git remote -v
```
Push your up to date local repo to the remote secondary repo
```sh
git push secondary
```
For further git configuration to allow remote checking via external transport etc, see `man git-config`.

#### Clean up commit history / reinit the repository {#reinit-repository-method}

Deleting the .git folder may cause problems in your git repository. If you want to delete all your commit history but keep the code in its current state, it is very safe to do it as in the following:

Checkout/create orphan branch (this branch won't show in git branch command).
```sh
git checkout --orphan latest_branch
```
Add all the files to the newly created branch:
```sh
git add -A
```
Commit the changes:
```sh
git commit -am "Initial commit, notable in description"
```
Delete main (default) branch (this step is permanent).
```sh
git branch -D main
```
Rename the current branch to main:
```sh
git branch -m main
```
Finally, all changes are completed on your local repository, force update your remote repository.
```sh
git push -f origin main
```

#### Clean up branch commit history avoiding rebase {#clean-commit-history}

Similar to the [reinit method](#reinit-repository-method). This method has been automated as push as the last step by [arielf](https://github.com/arielf/clean-push/tree/main).

Rebase is confusing. Usual `git merge --squash` does not show a branch as merged.
```sh
git log --graph --decorate --pretty=oneline --abbrev-commit
```
Get a clean commit history from a long used feature branch, say `xml_support`. Pull before this to be sure no changes sneak in.

1. Create a new branch from the commit the main branch diverged from the feature branch, `main` if nothing has been added since.
```sh
git branch xml_support_finalized main
```
1. Squash merge feature branch into the new branch.
```sh
git merge --squash xml_support xml_support_finalized
git commit -m'Add XML support' # or skip -m and see what is merged, edit the message.
```
1. Merge the new branch into main.
```sh
git checkout main
git merge xml_support_finalized
git diff main xml_support
```
1. If merging into main failed you can hard reset and merge again.
```sh
git reset --hard id_of_commit_before_merge
```
1. Success! Clean up, removing the branches. Since `xml_support` has the same content as the merged `xml_support_finalized`, it's safe to ignore warnings about it not being merged.

#### Get a single file from a github hosted repo

```sh
wget --content-disposition https://github.com/c0hen/c0hen.github.io/blob/master/README.md?raw=true
```
## Worktrees

```sh
git worktree list
git worktree add --checkout xml_support
git branch
git worktree add -b xml_support_hotfix
```
In general, all pseudo refs are per-worktree and all refs starting with refs/ are shared. Pseudo refs are ones like HEAD which are directly under `$GIT_DIR` instead of inside `$GIT_DIR/refs`. There are exceptions, however: refs inside refs/bisect, refs/worktree and refs/rewritten are not shared.
```sh
git rev-parse --verify HEAD
```
To access refs, it’s best not to look inside `$GIT_DIR` directly. Instead use commands such as `git-rev-parse` or `git-update-ref` which will handle refs correctly.

By default, the repository config file is shared across all worktrees.

## Debugging and helper tools

### git environment variables and debugging

There are environment variables (traces) that help debug aspects of git.
The possible values of these variables are as follows:

- "true", "1", or "2" – the trace category is written to stderr.

- An absolute path starting with / – the trace output will be written to that file.

```sh
GIT_TRACE=/tmp/git_trace.txt
GIT_CURL_VERBOSE=1
```
```sh
GIT_MERGE_VERBOSITY=5
```

### CLI tools for working with git hosts

#### [gitlab](https://docs.gitlab.com/cli/) - [glab](https://packages.debian.org/stable/utils/glab)

Work with merge requests (pull requests).
```sh
glab mr --help
```

#### [github](https://cli.github.com/manual/gh) - [gh](https://packages.debian.org/stable/utils/gh)

Work with pull requests.
```sh
gh pr --help
```

```sh
gh auth status
gh auth switch
```

Github fine grained access required for pull request merging seems to be:
- Contents
- Merge queues
- Metadata
- Pull requests

```sh
gh pr list
gh pr ready 2 # mark a draft request ready for review
gh pr merge 2 --squash --body 'add gathered content' --delete-branch
```
