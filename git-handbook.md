---
layout: default
title: Git handbook
description: Git handbook for a begginner, by a git beginner.
tags: git beginner tips
---

* Table of contents
{:toc}

Thoroughly assemble your .gitignore - files excluded from the remote repository.
SSH keys can be used to access [Github](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)

**Try to use small changes/commits and concise messages.**

HEAD and branches, including master, can be pictured as labels on an object. Git commands make more sense when looking at git from the inside.
A good talk on git internals by [Michael Schwern at Linux.conf.au 2013](https://www.youtube.com/watch?v=eQFZ_MPTVpc).

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
git init myrepo.git
git clone https://github.com/c0hen/c0hen.github.io
```

### Very basic flow with remote:

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

[Try and learn it on Github](https://try.github.io/levels/1/challenges/1)

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

#### Squash 2 pushed commits

Using --force may overwrite refs other than the current branch, including local ones. man git push

```sh
git rebase -i origin/master~1 master
git push origin +master
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

Take care to understand each rebase.
Rebasing allows to merge related commits and keep mistakes from littering the project's history.

```sh
git rebase -i
git push
```
To automatically rebase before a pull add this to git config:

```
[pull]
        rebase = true
```

#### Remove a manually deleted file from tree

```sh
git ls-files --deleted -z | xargs -0 git rm
```

### Branches

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

#### Commit branch to master

```sh
git checkout master
git merge social
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
git remote add remotename gituser@secondary.server:/path/to/myrepo.git 
git remote -v
```

#### Add git remote with SSH specified key login

Can be set per repository. See man git-config

```sh
cd myrepo.git
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
git --bare --shared=group init myrepo.git
chgrp gitgroup myrepo.git
```

On the machine you have your current latest repo and plan to push from:
```sh
cd myrepo.git
```
Allow external protocol, ssh transport with separate key.
Only from user initiated commands, not from git initiated automated commands.
See man git-config, one SSH key is simpler and allows avoiding configuring
the external ssh transport.
```sh
git config protocol.allow.ext user
```
Copy your ssh key to the secondary server, test login.
```sh
ssh-copy-id -i ~/.ssh/id_secondary gituser@secondary.server 
ssh gituser@secondary.server
exit
```
Add the remote with the short name "secondary" and ssh URL
```sh
git remote add secondary 'ext::ssh -i ~/.ssh/id_secondary gituser@secondary.server %S /home/gituser/myrepo.git'
```
List remotes
```sh
git remote -v
```
Push your up to date local repo to the remote secondary repo
```sh
git push secondary
```
For further git configuration to allow remote checking via external transport etc, see man git-config.

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

```sh
gh pr list
gh pr ready 2 # mark a draft request ready for review
gh pr merge 2 --squash --body 'add gathered content' --delete-branch
```

#### Get a single file from a github hosted repo

```sh
wget --content-disposition https://github.com/c0hen/c0hen.github.io/blob/master/README.md?raw=true
```
