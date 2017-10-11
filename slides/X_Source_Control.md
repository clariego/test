layout: true

.footer-picture[![Network to Code Logo](slides/media/Footer2.PNG)]
.footnote-left[(C) 2015 Network to Code, LLC. All Rights Reserved. ]
.footnote-con[CONFIDENTIAL]


---

class: center, middle, title
.footer-picture[<img src="slides/media/Footer1.PNG" alt="Blue Logo" style="alight:middle;width:350px;height:60px;">]

# Source Control

---

# Module Overview

.left-column[
- Source Control
- Git
- GitHub
]
.right-column[
<img src="slides/media/git/git.png" alt="Git" style="alight:middle;width:200px;height:100px;">

<img src="slides/media/git/github_cat.jpeg" alt="GitHub Cat" style="alight:middle;">
]


---

# Source Control (Version Control)

- “Version control is a system that records
changes to a file or set of files over time so that
you can recall specific versions later.” (http://git-scm.com/book/en/v2/Getting-Started-About-Version-Control)

- Collaborate with other engineers and developers

- Integrate changes from other developers into your project

---

class: middle

# Git
### Source Control

---

# Git

- Distributed source code management system
    - Work offline
    - No persistent connection to central server needed
    - Backup your work on-demand
- git essentially takes snapshots
- It records the entire state of your project at different
moments in time


---

# Git repository (repo)

- A directory that
  somebody has **initialized** with git
- Only the original creator needs to initialize
- Git **tracks** changes made to the files and folders in this directory*
  - *for files you specify
- Files can be code, configs, or anything else (such as this slide deck)


---

class: ubuntu

# Initial Setup

- Tell git who you are:
```
git config --global user.name "Joe Smith"
git config --global user.email joe_smith@mycompany.com
```
- You only have to do this once


---

# Working with a Git repository (repo)

There are two options to get started:

- Start your own with `git init`
- Copy someone else's with `git clone`

---

class: ubuntu

# git init
- Create your own git repository
```
ntc@ntc:~/ntc/training$ git init
Initialized empty Git repository in /ntc/training/.git/
```

- Files in `~/ntc/training/` now have the ability to be tracked

---

# Directory setup

- We'll use a switch config snippet called `switch1.cfg` in `~/ntc/training/`
```bash
vlan 10
   name web_vlan_10
!
vlan 20,30
!
interface Port-Channel10
   switchport trunk native vlan 20
   switchport mode trunk
!
interface Ethernet1
!
interface Ethernet2
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
   channel-group 10 mode active
!
interface Ethernet6
   channel-group 10 mode active
```

---

class: ubuntu

# git status

- `git status` gives useful information about the repo

```
ntc@ntc:~/ntc/training$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

  switch1.cfg

nothing added to commit but untracked files present (use "git add" to track)
```

- git informs that `switch1.cfg` is **untracked**

---

class: ubuntu

# git add

.left-column[
- `git add` to track a file

```
ntc@ntc:~/ntc/traning: git add switch1.cfg
ntc@ntc:~/ntc/traning:
```

- Updated status

```
ntc@ntc:~ntc/training$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

  new file:   switch1.cfg
```
]

--

.right-column[
**Two** things just happened
  1. `switch1.cfg` is now **tracked**
    - git notices if you make changes

  2. `switch1.cfg` is in the "staging area" or **staged**
    - git includes it in the next snapshot (commit)
]

---

class: ubuntu

# git commit

.left-column[
- Take a snapshot of your directory with `git commit`

```
ntc@ntc:~/ntc/training$ git commit -m "my first commit"
[master (root-commit) 65302a8] my first commit
 1 file changed, 22 insertions(+)
 create mode 100644 switch1.cfg
```
- The `-m` flag stands for "message"
  - git requires a message with each commit
  - `"my first commit"` is the message supplied
]

--

.right-column[
- Updated status

```
ntc@ntc:~/ntc/training$ git status
On branch master
nothing to commit, working directory clean
```

- We now see there are no **tracked** files that have **uncommitted** changes
]

---

class: ubuntu

# More Examples

.left-column[
- Now we'll make **two** changes to the repo.
  1. Modify the contents of ``switch1.cfg``
  2. Add ``switch2.cfg`` to the directory
]


---

class: ubuntu

# More Examples

.left-column[
- Now we'll make **two** changes to the repo.
  1. Modify the contents of ``switch1.cfg``
  2. Add ``switch2.cfg`` to the directory

- Updated status

```
ntc@ntc:~/ntc/training$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

  modified:   switch1.cfg

Untracked files:
  (use "git add <file>..." to include in what will be committed)

  switch2.cfg

no changes added to commit (use "git add" and/or "git commit -a")
```

]

.right-column[

- `switch1.cfg` is **tracked** and **modified** but **not staged**
  - Modifying a file removes it from the staging area until you `git add` it again
- `switch2.cfg` is **untracked**
]

---

class: ubuntu

# git diff

- What changes were made to the contents of `switch1.cfg`?

```
ntc@ntc:~/ntc/training$ git diff switch1.cfg
diff --git a/switch1.cfg b/switch1.cfg
index f348934..37daefe 100644
--- a/switch1.cfg
+++ b/switch1.cfg
@@ -1,5 +1,5 @@
 vlan 10
-   name web_vlan_10
+   name db_vlan_10
 !
 vlan 20,30
 !
```

- The line that starts with `+` was added
- The line that starts with `-` was removed
- "vlan 10" changed its name from "web_vlan_10" to "db_vlan_10"

---

class: ubuntu

# More Examples

.left-column[
- Start tracking `switch2.cfg`

```
ntc@ntc:~/ntc/training$ git add switch2.cfg
ntc@ntc:~/ntc/training$
```
]

--

.right-column[

- Updated status

```
ntc@ntc:~/ntc/training$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

  new file:   switch2.cfg

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

  modified:   switch1.cfg
```

- `switch2.cfg` is now **tracked** and **staged**
- `switch1.cfg` is still **tracked** and **modified** but **not staged**

]

---

class: ubuntu

# More Examples

.left-column[
- Make another commit (snapshot)

```
ntc@ntc:~/ntc/training$ git commit -m "added switch2.cfg"
[master 06211d9] added switch2.cfg
 1 file changed, 22 insertions(+)
 create mode 100644 switch2.cfg
```
]

--

.right-column[
- Updated status

```
ntc@ntc:~/ntc/training$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

  modified:   switch1.cfg

no changes added to commit (use "git add" and/or "git commit -a")
```

- `switch2.cfg` is **tracked** and has been **committed**
- `switch1.cfg` is still **tracked** and **modified** but **not staged**
  - Only staged files are included in the commit
]

---

class: ubuntu

# More Examples

.left-column[
- Stage `switch1.cfg` with `git add`

```
ntc@ntc:~/ntc/training$ git add switch1.cfg
ntc@ntc:~/ntc/training$
```

- Updated status

```
ntc@ntc:~/ntc/training$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

  modified:   switch1.cfg
```

]

--

.right-column[

- Finally, `switch1.cfg` is **staged** again
  - It's still also **tracked** and **modified**

- Commit the changes
```
ntc@ntc:~/ntc/training$ git commit -m "made changes to switch1.cfg"
[master 877b034] made changes to switch1.cfg
 1 file changed, 1 insertion(+), 1 deletion(-)
```
```
ntc@ntc:~/ntc/training$ git status
On branch master
nothing to commit, working directory clean
```

]



---

class: ubuntu

# git commit - Useful shortcut

.left-column[
- Automatically include all **tracked** files in a commit with the `-a` flag
  - Regardless of whether they are **staged** or **unstaged**

- Examples:

  -`git commit -a -m "committing all tracked files"`

  -`git commit -am "committing all tracked files"`
]

--

.right-column[

- `git add` only once at the the beginning for each file you want to **track**

- Use with caution
  - You may accidentally commit unwanted changes
]

---

# Remote Repositories

- Purposes
  1. Back up your repo (all of your commits)
  2. Allow others to copy and/or modify your repo
  3. Copy and/or modify other people's repos

- **GitHub** is a common remote repository

---

class: ubuntu

# git remote

- Connect our repo to Github
  - (Assumes some preliminary work on Github's website)

- Add the remote repository with `git remote`
```
ntc@ntc:~/ntc/training$ git remote add origin https://github.com/networktocode/training.git
ntc@ntc:~/ntc/training$
```

- git is now aware of a remote repository called "origin"
  - "origin" is an alias on your system for a particular repository that exists elsewhere, i.e. the link shown above
  - "origin" is a common default repository name

---

class: ubuntu

# git push

- Sync all the commits to the remote repo
```
ntc@ntc:~/ntc/training$ git push origin master
Counting objects: 8, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (7/7), done.
Writing objects: 100% (8/8), 866 bytes | 0 bytes/s, done.
Total 8 (delta 1), reused 0 (delta 0)
To git@github.com/networktocode/training.git
 * [new branch]      master -> master
```

- **origin** is the name (alias) we gave to the remote server
- **master** is the name of our branch (covered later)

---
class: ubuntu


# git clone

- Someone else can now get a copy of our repo with `git clone`
- Simulating new user, i.e. "user2" (on a totally different machine)

  ```
  user2@prod:~$ git clone https://github.com/networktocode/training.git
  Cloning into 'training'...
  remote: Counting objects: 8, done.
  remote: Compressing objects: 100% (6/6), done.
  remote: Total 8 (delta 1), reused 8 (delta 1), pack-reused 0
  Receiving objects: 100% (8/8), done.
  Resolving deltas: 100% (1/1), done.
  Checking connectivity... done.
  ```

- Automatically creates a directory called `training`
  - With files already committed

  ```
  user2@prod:~$ cd training
  user2@prod:~/training$ ls
  switch1.cfg switch2.cfg
  ```

---

# git clone vs. git init

- `git clone` and `git init` are two ways of starting a local repository

- Mutually exclusive

- Start your own local repo with `git init`

- Copy a remote repo with `git clone`

---

# git pull

- `git pull` is the counterpart to `git push`

- If many people contribute to a common remote repository, sync your local repo with the changes they made using `git pull`

- Examples
  - `git pull origin master`
  - After you clone a project, you will want to frequently do a *pull* to get the updates from your team members

---

# Branches

- Division of labor on a shared repo

- Keep new features separated while under development

- Commits only affect the branch you are working on

---

# Branching Strategy Example
.center[
<img src="slides/media/git/branch-strategy.png" alt="Branching Strategy Example" style="alight:middle;width:999px;height:405px;">
]

---

class: ubuntu

# Branches

.left-column[

- View branches with `git branch`

  ```
  user2@prod:~/training$ git branch
  * master
  ```
  - `master` is the only branch
  - `*` denotes the current working branch

- Create (and switch to) a new branch with `git checkout -b`

  ```
  user2@prod:~/training$ git checkout -b new_branch
  Switched to a new branch 'new_branch'
  ```
]
--

.right-column[

- View branches again

  ```
  user2@prod:~/training$ git branch
    master
  * new_branch
  ```

- Switch back to `master` with `git checkout`
```
user2@prod:~/training$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
```

]

---

# Branches

- `git push` and `git pull` work with branches on remote repos

  - Examples
    - `git push origin new_branch`

    - `git pull origin new_branch`

---

# Branches

- `git merge` merges two branches on a local repo
- Example
  - `git merge new_branch`

  - Merges `new_branch` with `master`

- Sometimes there are **merge conflicts**
  - Need to be resolved before next commit
  - git shows you with `git status`



---

class: middle

#GitHub
### Source Control

---

# GitHub

- Distributed Version Control System based on Git that is a web based hosting service
- Free Version for public files / code repositories
- Git + Code Review
- GitHub Enterprise
  - On-prem version of GitHub.com
- Social Coding
- An “online resume” that’s real

---

# GitHub

.left-column2[
- Registration is simple
- Create an account
- Manage SSH keys or use https
- Create your first repo!
- Start collaborating
]

.right-column2[
- <img src="slides/media/git/git_reg.png" alt="Navigate" style="     alight:middle;width:600px;height:435px;">
]

---

# Pull Request

- Tell others about changes you've pushed to a remote repo on GitHub.

- Changes could be on a **branch** you are working on
  - If you have write-access to the GitHub repo

- Or an a **fork** of the original repo (discussed more later)
   - If you have read-only access to the GitHub repo

---

# Pull Request

- Once a pull request is sent, interested parties can review the set of changes, discuss potential modifications, and even suggest follow-up commits if necessary.

- The repo maintainer then **merges** the changes into the **master** branch of the repo.

---

# Fork & Pull

- Contributor wants to add to a public GitHub repo

- Contributor has read-only access to the repo

- Contributor **forks** the repo on GitHub

- Contributor makes and pushes desired changes

- Maintainer **merges** changes

---

class: middle

# Contributor Point of View

---

# Step 1: Navigate to the Repository & Fork

.center[
<img src="slides/media/gitv2/git_01.png" alt="Navigate" style="alight:middle;width:850px;height:475px;">
]

---

# You now have YOUR own copy on GitHub

.center[
<img src="slides/media/gitv2/git_02.png" alt="Navigate" style="alight:middle;width:850px;height:475px;">
]

---

# Step 2a: Make a change to YOUR new repo

.left-column2[

- Making a change via the WEB UI.  This does a push back to YOUR repo.
]

.right-column2[

<img src="slides/media/gitv2/git_03.png" alt="fp_step4" style="alight:middle;width:700px;height:435px;">

]

---

class: ubuntu

# Step 2b: Make a change to YOUR new repo

- Making a change locally

Clone **YOUR** repo with `git clone`

```
ntc@ntc:~/training$ git clone https://github.com/GGabriele/training.git
Cloning into 'training'...
remote: Counting objects: 80, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 80 (delta 4), reused 0 (delta 0), pack-reused 70
Unpacking objects: 100% (80/80), done.
Checking connectivity... done.
```

- Navigate to the new directory (name of the repo) and show files with `ls`

```
ntc@ntc:~/$ cd training/
ntc@ntc:~/training$ ls
hostname_test.py  ping_test.py  README.md  snippets  switch1.cfg  switch2.cfg
```

---

class: ubuntu

# Create a new file (Step 2b)

- Create a new file called `switch3.cfg`

```
ntc@ntc:~/training$ cat switch3.cfg 
vlan 10
  name db_vlan_10
!
vlan 20,30
!
interface Port-Channel10
   switchport trunk native vlan 20
  switchport mode trunk
!
interface Ethernet1
!
interface Ethernet2
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
   channel-group 10 mode active
!
interface Ethernet6
   channel-group 10 mode active
```

---

class: ubuntu

# Add, Commit, and Push the new file (2b)

- Add the new file to the repo with `git add`

```
ntc@ntc:~/$ git add switch3.cfg
```

- Commit the new file with `git commit -m`

```
ntc@ntc:~/$ git commit -m "Adding switch3.cfg"
[master 7fa7001 Adding switch3.cfg
    1 file changed, 22 insertions(+)
    create mode 100644 switch3.cfg
```

- Push the changes with `git push`

```
ntc@ntc:~/$ git push origin master 
Username for 'https://github.com': GGabriele
Password for 'https://GGabriele@github.com':
counting objects: 11, done.
Compressing objects: 100% (7/7), done.
Writing objects: 100% (7/7), 849 bytes | 0 bytes/s, done.
Total 7 (delta 3), reused 0 (delta 0)
To https://github.com/GGabriele/training.git
    efdbcf5..7fa7001  master -> master
``` 

---

# Step 3: Initiate the Pull Request

.center[
<img src="slides/media/gitv2/git_04.png" alt="fp_step5" style="alight:middle;width:850px;height:475px;">
]

---

# Step 4: Review the Pull Request and Create

.center[
<img src="slides/media/gitv2/git_05.png" alt="fp_step6" style="alight:middle;width:850px;height:475px;">
]

---

# Step 5: Add Comments and Create

.center[
<img src="slides/media/gitv2/git_06.png" alt="fp_step7" style="alight:middle;width:850px;height:475px;">
]

---

# Pull Request Succeeds

.center[
<img src="slides/media/gitv2/git_07.png" alt="fp_step8" style="alight:middle;width:850px;height:475px;">
]

---

class: middle

# Maintainer Point of View

---

# Step 1: Browse to Correct Repository

.center[
<img src="slides/media/gitv2/git_08.png" alt="fp_step10" style="alight:middle;width:850px;height:475px;">
]

---

# Step 2: Examine List of Pull Requests

.center[
<img src="slides/media/gitv2/git_09.png" alt="fp_step11" style="alight:middle;width:850px;height:475px;">
]

---

# Step 3: View Changed Files

.center[
<img src="slides/media/gitv2/git_10.png" alt="fp_step12" style="alight:middle;width:800px;height:450px;">
]

---

# Step 4: Comment/Approve the Pull Request

.center[
<img src="slides/media/gitv2/git_11.png" alt="fp_step13" style="alight:middle;width:775px;height:475px;">
]

---

# Step 5: Confirm Merge

.center[
<img src="slides/media/gitv2/git_12.png" alt="fp_step13" style="alight:middle;width:775px;height:475px;">
]

---

# Final View of Merge

.center[
<img src="slides/media/gitv2/git_13.png" alt="fp_step14" style="alight:middle;width:800px;height:450px;">
]

---

# Summary

- git is a distributed Source (Version) control system
- git can be used for code or any type of files
- GitHub is a hosted service for remote repositories
- GitHub is also a powerful tool for collaboration
- Online Resume!

---

# Lab Time

- Lab 32 - Source Control with git and GitHub
  - Creating A GitHub Account
  - Making a Pull Request using Fork & Pull Method












