# Git & GitHub Cheetsheet

- [Git & GitHub Cheetsheet](#git--github-cheetsheet)
  - [Git](#git)
    - [Basics](#basics)
    - [Branching](#branching)
    - [Merging](#merging)
    - [Stash](#stash)
    - [Restore](#restore)
    - [Reset](#reset)
    - [Revert](#revert)
    - [Diff](#diff)
  - [GitHub](#github)
  - [Yet to be added](#yet-to-be-added)

References:

- [Atlassian Tutorials](https://www.atlassian.com/git/tutorials)
- [Git Command Reference](https://git-scm.com/docs)

## Git

### Basics

 `git init` Initialize repository \
 `git add .` Add all the files in current dir to staging \
 `git restore --staged <file_name>` unstage the file \
 `git commit -m "message"` commit the files

### Branching

 `git branch` list all the branches \
 `git branch <branch_name>` create a new branch \
 `git switch <branch_name>` switch to a different branch

 ![2021-11-13_15-27.png](assets/2021-11-13_15-27_1636837698047_0.png)

 `git branch -d <branch_name>` delete the branch \
 `git branch -M <branch_name>` rename a branch

### Merging

 `git merge <branch_to_merge>` merge a branch to current branch \

 > **Note**
 >
 > - if the same file was edited on the different branches that you are merging, it is possible to get a [[merge conflict]]
 > - If the same file was not modified, then you get a fast forward  merge

### Stash

 `git stash` save the current changes to stash \
 `git stash pop` apply the recently stashed files \
 `git stash apply`apply the recently stashed files \

 > **Note**
 >
 > - `stash pop` can be used only to apply stashed files only in ^^current branch^^ and it ^^removes files^^ from the stash list
 > - `stash apply` can be used only to apply stashed files ^^across branches^^ and it ^^does not remove files^^ from the stash list so the files can be reused

 `git stash drop stash@{x}` delete a particular stash `x` from list \
 `git stash clear` clear all stashes

### Restore

 `git checkout <old_commit_hash>` move the HEAD from the latest commit to an old commit

 > **Note**
 >
 > - This concept is known as ==detached head==
 > - This can also be achieved by using reference to HEAD as below
 > - `git checkout HEAD~n` move the HEAD to **n** commit before the latest commit
 > - To point the HEAD back to the latest commit, switch to the branch with `git switch <branch_name>`

 `git restore <file_name>` discard all changes made to a file since last commit \
 `git restore --source HEAD~n <file_name>` restore the file to what it was **n** commits ago

### Reset

 `git reset <commit_hash>` go back to a specific commit and delete all the commits done after
 > **Note**
 >
 > - This command only removes the commits from the history of commits when using `git log` command but the ==changes made to the files are not reverted back== the the specified commit
 > - Instead, the files modified between these commits will be put on the staging dir with the changes made on them

 `git reset <commit_hash>` go back to a specific commit and delete all the commits done after along with the changes made to the files

### Revert

 `git revert <commit_hash>` create a new commit undoing all the changes done till the specified commit

 > **Note**
 >
 > - `revert` is very similar to `reset` but instead of deleting the commits made, it creates a new commit with all changes discarded.
 > - `reset` will show the commit history like the wrong commits never happened but `revert` will keep record of the commits so we can have them as reference even after discarding the changes made by those commits
 > - `revert` is very useful when collaborating with other to work on code so the ==commit history does not mess up during merging==

### Diff

 `git diff` show all the changes in working directory that are not staged \
 `git diff --staged` show all the changes between staging area and last commit \
 `git diff  commit_x..commit_y` show all the changes between commits \
 `git diff  branch_x..branch_y` show all the changes between branches



## GitHub

All commands are applicable for remote repositories like GitLab & Bit Bucket

`git clone <url>` clone a repository and initialize it

- Remote

`git remote -v` list all the remotes available (remotes are the repos created in GitHub) \
`git remote add <name> <url>` add a new remote

> **Note**
>
> - `origin` is usually used for the name of the remote

`git remote rename <old_name> <new_name>` rename a remote \
`git remote remove <name>` delete a remote

- Before working with push / pull with GitHub, its is important to be aware of below info

> **Note**
>
> - GitHub renamed the default branch from master to main
> - All repositories ==**initialized**== by adding a README file or .gitignore file on GitHub will have main as the default branch
> ![2021-11-13_19-54.png](assets\2021-11-13_19-54_1636837660856_0.png)
> - But git command still uses master as the default branch. All repositories initialized locally with git will have master as the default branch.
> - In case of conflict, rename the default branch to master / main before pushing to `remote`

- Push

`git push <remote> <remote_branch>` push branch to a remote repo to a remote branch

> **Note**
>
> - if the remote branch specified does not exist, it will be created

`git push <remote> <local_branch>:<remote_branch>` push local branch to a remote repo \
`git push -u origin <branch>` sets upstream of the current branch on local to specified branch on origin

> **Note**
>
> - setting upstream will connect / map a local branch to a remote branch allowing us to just use just `git push` instead of specifying the remote and branch every time we want to push

- Remote Tracking branch

`git branch -r` shows the remote branch HEAD

> **Note**
>
> - it is what keeps track of the `remote/main` branch with the local commits
> ![2021-11-13_21-11.png](assets/2021-11-13_21-11_1636837613928_0.png)

- Fetch

> **Note**
>
> - `fetch` updates the `remote tracking branch` with latest updates from remote repo but does not add the change to the current HEAD / working dir or merge the changes

`git fetch <remote>` get the latest changes from the remote repo \
`git fetch <remote> <branch>` get the latest changes from the remote repo

![2021-11-14_00-03.png](assets/2021-11-14_00-03_1636844790813_0.png)

- Pull

> **Note**
>
> - `pull` updates the `remote tracking branch` with latest updates from remote repo and also merges the changes to the current HEAD
> - `git pull` = `git fetch` + `git merge`

`git pull <remote>` git pull the remote repo
`git pull <remote> <branch>` git pull a specific branch the remote repo

![2021-11-14_00-04.png](assets/2021-11-14_00-04_1636844958837_0.png)

> **Warning**
>
> - sometimes it is possible to get a [[merge conflict]] during a `git pull`
>

- Rebase

`git rebase main` rebase current branch to the HEAD of the main branch

> **Warning**
>
> - DO NOT rebase the branch that others are working on or has been shared with others!
This will mess up the whole history and become a pain to fix 😫

`git rebase -i HEAD~n` perform [[interactive rebase]] to re-write the commit history

- Interactive Rebase
used to re-write the commit history and below are the options available

`pick` : use the commit \
`reword` : use the commit, but edit the commit message \
`edit` : use the commit but stop for amending \
`squash` : use commit contents but meld it into previous commit with new commit message \
`fixup` : use commit contents but meld it into previous commit and discard the commit message \
`drop` : remove the commit

- Interactive rebase example
  - The plan
 ![2021-11-15_12-16.png](assets/2021-11-15_12-16_1636975216849_0.png)
  - change the commit message

  `git rebase -i HEAD~9` then update the action in 1st window and change the commit message in 2nd window
  ![2021-11-15_11-29.png](assets/2021-11-15_11-29_1636972952959_0.png)
  ![2021-11-15_11-29_1.png](assets/2021-11-15_11-29_1_1636972964354_0.png)
  ![2021-11-15_11-30.png](assets/2021-11-15_11-30_1636973234182_0.png)

  - perform `fixup` to save the commit content but discard the commit messages
  
  `git rebase -i HEAD~9`
  
  ![2021-11-15_11-33.png](assets/2021-11-15_11-33_1636973215819_0.png)
  ![2021-11-15_11-34.png](assets/2021-11-15_11-34_1636973342424_0.png)

  - completely drop a commit and changes made in that commit

  `git rebase -i HEAD~6`

  > **Warning**
  >
  > - With every `fixup` the no of commits gets changed so make sure to use the correct reference to HEAD

  ![2021-11-15_11-38.png](assets/2021-11-15_11-38_1636973563490_0.png)
  ![2021-11-15_11-38_1.png](assets/2021-11-15_11-38_1_1636973587219_0.png)

  - `fixup` vs `squash`

  `squash` prompts for new commit message but `fixup` uses the commit message of the commit that we will merge into

  ![2021-11-15_12-04.png](assets/2021-11-15_12-04_1636975393378_0.png)
  ![2021-11-15_12-05.png](assets/2021-11-15_12-05_1636975406231_0.png)
  ![2021-11-15_12-06.png](assets/2021-11-15_12-06_1636975428973_0.png)

- Overall flow

![2021-11-13_21-41.png](assets/2021-11-13_21-41_1636837556802_0.png)

</details>

## Yet to be added

  ```
  git config --global user.name dhayanand
  git config --global user.email dhayanands@hotmail.com
  git config --global credential.helper store
  git config --global init.defaultBranch main
  
  clone specific branch
  clone --single-branch -b develop <https://github.com/sysco-middleware/ansible-dojo-day0-part1>

  git rev-list --count branch-dev

  git log --oneline

  git log --graph

  git pull --allow-unrelated-histories

  git rebase

  git bisect

  git blame

  ```

