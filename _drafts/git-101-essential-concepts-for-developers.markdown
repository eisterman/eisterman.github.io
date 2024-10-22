---

alayout: post
title: "GIT 101: Essential Concepts for Developers"
categories: git guide vcs
mermaid: true

---

Git is the most widespread distributed Version Control System (VCS) used worldwide. Its purpose is to help maintain and manage a set of files that change over time by recording all the changes made to it and assisting in the concurrent progression of multiple people on the same set of files.

While it is primarily used in programming for source code management, it can also be used for managing any text file that changes over time (unfortunately, it is useless for files like Word or Excel that are not in "raw text" format).

## Theory - Repository Features

```mermaid
gitGraph
    commit
    commit
    branch develop
    branch feat1
    checkout develop
    commit
    commit
    checkout master
    merge develop
    checkout feat1
    commit 
    commit
    checkout master
    commit
    commit type: HIGHLIGHT tag: "HEAD"
```

The **repository** is the name given to a project that is managed using git.

The fundamental idea of GIT is that every change in the state of the source code can be represented by a list of "differences," specifically added and removed lines. For example, if you modify a line, it will count as a removal and an addition. These "change packages" are called **commits** and are the fundamental building block that git uses to function. Any git functionality can be traced back to how to create/manage/modify/merge/inspect commits.

In a way that almost resembles the more modern blockchain, each commit contains a list of links to the previous commits, if present, and is represented by a unique hash.

This allows the commit tree to be reconstructed through these links, providing a kind of certainty that previous commits cannot be modified without recreating all the commits, allowing, even in the event of disasters/"back in time" modifications, not to risk randomly losing data.

Each repository can thus be represented as a tree of interconnected commits.

In the graph all the points with an hash like "0-99344e5" are commits.



To help keep track of progress, there are **branches**, including a special one called master/main.

A branch is nothing more than a "pointer" to a specific commit, which can be moved to another commit, such as a newly created commit on top of the current one.

Thanks to this property, branches are usually used to represent "work branches," i.e., those commits being worked on to accomplish something. A classic example of a repository configuration is to have a master branch with the "production"/final version of the software, and various branches for working on different features which, once completed, will be merged into master.

In the graph the branches are the horizontal lines containing commits.



Now let's mention the last essential feature of the theory, namely the **merge**.

When you have multiple branches with a different state, you may find yourself needing to merge changes from one branch into the other. This process is called merge and creates a special commit, a child of both branch heads, in which the differences between the two branches are unified, possibly with human help if necessary.

In the graph the point where two branch unite is a **merge commit**, and even if in the graph the hash is not reported, as you will see in a real repository these is a full-fledged commit with hash and everything.



After this overview of GIT fundamentals in local, it is now time to look at the "distributed" aspect of GIT, namely the possibility to synchronize your code with a remote repository on platforms like GitHub and GitLab.

## Theory - Distributed Repository

A GIT repository has its benefits offline for managing and keeping track of your code changes, but its strength becomes even more evident when working on a project together with multiple people or multiple machines.

The fundamental principle is that each repository can have **remotes**, which are special links, such as `git@github.com:user/projectrepo.git`, which git can use to read the remote state of the repository. The vast majority of repos have a single remote called **origin**.

The idea is to download the list of remote branches and online commits from the remote and work on them offline, which will appear as special branches with the name `origin/branchname`.

--- example graph of a commit tree with local and remote repos.

Besides fetching information from the remotes, you can also push it, uploading your commits and the position of your branches online. It will become (hopefully) clearer at the end of the explanation of the commands.

## Commands - How to do!

### Repository Creation

The first important commands are those for creating a repo.

The typical commands are:

```bash
git init  # initializes an empty repository in the current folder
git clone CLONE_URL  # creates a new repository by copying the state of a remote repository
```

The most classic way to create a repo is generally to create a repo on GitHub and clone it, so you already have all the logic preconfigured online.

If you proceed to clone someone else's repository for which you have read but NOT write permissions, the repository can be modified locally but not sent online. To create your own version of someone else's repo, there is the **fork**, which I will explain later.

It's important to remember that git has an extremely comprehensive first-class manual, sometimes overly so, which can be accessed with `git --help`. The manual is available for each subcommand, such as `git clone --help` or `git checkout --help`.

### Status and Information

The second set of commands are for information

##### `git status`

This command allows you to observe the current state of the commit you're on, referred to in the system as `HEAD`.

```
fpasqua@EisterBox:~/git/eisterman.github.io$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   test

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   _posts/2024-07-07-welcome-to-jekyll.markdown

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        _posts/GIT.md
```

With this command, we can see at a glance the most important information about the current state of the repo, in particular:

- `On branch`, i.e., which branch we are working on and if there are differences between the local branch and the remote one

- `Changes to be committed`, i.e., the files which changes have been selected for inclusion in the next created commit

- `Changes not staged for commit`, i.e., files that have been modified but will not be added to the next created commit

- `Untracked files`, i.e., files that are not present in GIT and thus are not tracked. Practically the files ignored by GIT.

Conveniently, git status also shows some useful commands for adding/removing from commits, so you don't have to search for them each time.

##### `git log` and `git tree`

This extremely powerful command allows you to see list and details of commits, the position of the various branches and, with the right options, the commit graph.

The simplest version is `git log` without options that only shows a generic list of commits in chronological order, without showing the relative links between the various trees

```
commit 39b3fea912a4ffe687b6d22199b2ada4d8b65778 (HEAD -> hotfix_release, origin/hotfix_release)
Author: Federico Pasqua (eisterman) <federico.pasqua.96@gmail.com>
Date:   Mon Jul 8 10:47:37 2024 +0200

    re.parkadmin.charge: Add sum endpoint to obtain the precalculated sum of the charge requested by filter.

commit bf35aae8e50e45850b3c1a1f42f070f460f1d9c4
Author: Production RE 1 <robot@rossinienergy.com>
Date:   Fri Jul 5 14:47:08 2024 +0000

    re.parkadmin.chargemap: Use GET instead of POST

commit 6bc42a94599cd5be1f3e12c9865297a10c94ebd0
Author: RE Staging1 <robot@rossinienergy.com>
Date:   Fri Jul 5 13:33:42 2024 +0000

    MIGRATION 036 - Chargemap Business Admin
```

An important piece of information is understanding at what state the repository is currently. This is indicated by `HEAD`, which represents the current state. Whenever you see `HEAD` around, it generally refers to the currently selected commit whose state is set in the repo.

The raw Git Log is not the easiest way to understand what's happening, particularly due to the lack of connections between commits, which is why `git log --graph` exists, showing the same list as before but with the connections between commits

```
* commit 6bc42a94599cd5be1f3e12c9865297a10c94ebd0
| Author: RE Staging1 <robot@rossinienergy.com>
| Date:   Fri Jul 5 13:33:42 2024 +0000
|
|     MIGRATION 036 - Chargemap Business Admin
|
*   commit f2367b5c1c78ddc0ebc0a8d2efa54232a51c9f1c
|\  Merge: fbd610a f99a3da
| | Author: Federico Pasqua (eisterman) <federico.pasqua.96@gmail.com>
| | Date:   Fri Jul 5 15:27:18 2024 +0200
| |
| |     Merge branch 'refs/heads/chargemap_integr' into hotfix_release
| |
| * commit f99a3dad16fc7ae81a0d814c6566b9e4687dc84a (origin/chargemap_integr)
| | Author: Federico Pasqua (eisterman) <federico.pasqua.96@gmail.com>
| | Date:   Mon Jul 1 14:04:00 2024 +0200
| |
| |     re.parkadmin.cmb: Add export endpoint proxy
| |
```

It's still quite cluttered with unnecessary information in 90% of cases, so here's a command that shows a simple and immediately understandable tree:

```bash
git log --graph --full-history --all --color --pretty=format:"%x1b[31m%h%x09%x1b[32m%d%x1b[0m%x20%s"
```

This log variant allows having a compact tree with only branches, Commit Hash, and Commit messages:

```
* 39b3fea        (HEAD -> hotfix_release, origin/hotfix_release) re.parkadmin.charge: Add sum endpoint to obtain the precalculated sum of the charge requested by filter.
* bf35aae        re.parkadmin.chargemap: Use GET instead of POST
* 6bc42a9        MIGRATION 036 - Chargemap Business Admin
*   f2367b5      Merge branch 'refs/heads/chargemap_integr' into hotfix_release
|\
| * f99a3da      (origin/chargemap_integr) re.parkadmin.cmb: Add export endpoint proxy
| * 997867e      re.parkadmin: Add ChargeMap Business Admin integration model fields
* | fbd610a      re.charge: Add Grace Period during the reboot time of the Pilotage, to not close charges for RP that take too much time to get back online.
|/
| * 81d8f6e      (origin/fix_csauth_v1public) re.public.auth: Add qrcode_allowed to FeaturedUser to replace the old legacy retrieve_user_charge_permissions
|/
* 776f16c        re.reports.yearly: Send Yearly Report only for the park with at least one CS with QRCode
```

To avoid having to type it every time, a good idea is to create a bash alias or set it as a global git alias. I prefer the second option.

By running the command

```
git config --global alias.tree "log --graph --full-history --all --color --pretty=format:\"%x1b[31m%h%x09%x1b[32m%d%x1b[0m%x20%s\""
```

The configuration is inserted into your `~/.gitconfig` file that allows recalling this long command simply with `git tree`. A really powerful tool for daily work.

##### `git show`

This command allows viewing the content of a commit, i.e., all its metadata and finally the `diff`, which is the list of additions and removals that the commit contains. The name comes from the utility used to generate these files and the related format, i.e., `diff`!

```
roberto@production-api-charge-re:~/backend_rossinienergy$ git show f99a3da
commit f99a3dad16fc7ae81a0d814c6566b9e4687dc84a (origin/chargemap_integr)
Author: Federico Pasqua (eisterman) <federico.pasqua.96@gmail.com>
Date:   Mon Jul 1 14:04:00 2024 +0200

    re.parkadmin.cmb: Add export endpoint proxy

diff --git a/re_restapi/views/parkadmin/current/chargemap.py b/re_restapi/views/parkadmin/current/chargemap.py
new file mode 100644
index 0000000..672bc61
--- /dev/null
+++ b/re_restapi/views/parkadmin/current/chargemap.py
@@ -0,0 +1,49 @@
+import logging
+import requests
+
+from rest_framework import serializers, status
+from rest_framework.decorators import api_view, permission_classes
+from rest_framework.response import Response
+from drf_spectacular.utils import extend_schema, inline_serializer, OpenApiTypes
+from re_restapi.libs.permissionviewset import *
```

Among the information in the metadata, we have the author, date, and commit message. From `diff` onwards, there's the representation in diff format of the changes contained in the commit.

### Commit Creation

Creating commits is the crux of daily work in git.

The logical process behind it is to break the work into steps:

1. Make sure you are on the correct branch where you need to work, for example, `master`.

2. Make your changes, test, etc.

3. Stage the modification for the future commit, i.e., select those you need to include in the new commit

4. Create the commit by adding a commit message that WELL describes the changes you just made. Always remember that while you may want to make commit messages like "bugfix," when you have a disaster and need to go back on your steps, unclear messages like this force you to read _commit by commit_ the changes, losing millennia of time!

Now let's do an example of creating a commit.

Imagine having a repo where we are on master and need to modify two files, `A.py` and `B.py`. If you want to try, you can go to an empty folder and type this in the terminal to initialize the branch to my same initial state:

```bash
git init
touch A.py B.py
git add A.py B.py
git commit -m "Initial Commit"
```

Continuing with the tutorial, you'll understand what is being done in this snippet.

After making sure with `git status` that we are in the right place, we make our changes.

`git status` at this point will show us something like this:

```
fpasqua@EisterBox:~/git/test$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   A.py
        modified:   B.py

no changes added to commit (use "git add" and/or "git commit -a")
```

At this point, we start to be able to do two things: `git add` and `git restore`.

##### `git restore`

This command allows discarding changes made and not committed to a git-tracked file. For example, if I did `git restore B.py`, all the changes I made to `B.py` that make it different from the current state of the repository would be discarded.

It can be convenient in certain situations.

##### `git add`

This essential command allows an altered file to be flagged for commit, i.e., make it **staged**.

If we do `git add A.py`, the subsequent `git state` will give us:

```
fpasqua@EisterBox:~/git/test$ git add A.py
fpasqua@EisterBox:~/git/test$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   A.py

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   B.py
```

Here we see how `A.py` is included in the next commit while B.py is not.

We also notice that `git status` suggested a new command to unstage.

##### `git restore --staged`

This variant of `git restore` allows removing a file from a commit without removing its changes. Practically, it's the exact inverse of `git add`. This command doesn't result in any data loss, so it can be used with confidence, paying attention that there is the `--staged` option when used, otherwise, modifications within the file will be lost!

Continuing with our example, we can now create a commit.

##### `git commit`

Creating a commit is done by simply executing `git commit`, which will open a window in a text editor where you can enter the commit message.

Once you save the temporary file, git will create the commit.

A variant to this procedure, which is very handy if the commit message isn't so long as to require a text editor, is `git commit -m "commit message text"`, which allows you to provide the commit message directly via command line.

```
fpasqua@EisterBox:~/git/test$ git commit -m "Modify A.py adding feature X"
[master 31ebe42] Modify A.py adding feature X
 1 file changed, 1 insertion(+)
fpasqua@EisterBox:~/git/test$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   B.py

no changes added to commit (use "git add" and/or "git commit -a")
```

And by doing `git tree`, we can notice the new commit on the master branch:

```
fpasqua@EisterBox:~/git/test$ git tree
* 31ebe42        (HEAD -> master) Modify A.py adding feature X
* bd883a1        Initial Commit
```

These are the fundamental infos to create a commit on a selected branch.

But if we could only create commits, it would become difficult to manage the work of multiple people, and GIT is designed precisely to handle that.

Let's dive deeper into branches and their superpowers.

### Branch

As we mentioned earlier, a branch can be imagined as a "pointer" pointing to a commit and indicating where we are working.

There are countless approaches to managing branches, but the simplest to understand, which isn't necessarily the best, especially for projects with more people, is called [Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow), where the master branch keeps track of the final version of the software, and there are separate branches for each feature, which are then merged into master when the feature is ready:

--- feature branch graph with master + 1 merged branch and 1 to be merged

The fundamental operations that can be performed on branches are:

1. **Checkout**: select a branch as the one in use. The repository's content will be put in the current state of the branch.

2. **Creation**: create a new branch pointing to an already existing commit.

3. **Merge**: insert the commits of branch B into A, merging the two paths and creating a special "merge commit" in A.

4. **Reset**: reset the state of a branch to a previous commit.

5. **Rebase**: The Secret Art of rewriting History. An extremely powerful operation that allows rewriting an entire branch's structure.

##### `git checkout`

Probably the most important operation in all of GIT. This command allows selecting the current state of the repository.

The syntax is `git checkout <commit|branch>`, meaning you can even select a single commit to revert the repo to that state. This special state where you checkout a commit rather than a branch is called *Detached Head*, because HEAD, the current state of the repo, is not directly linked to a branch, and any new commit risks getting lost in the labyrinths of Git's graph. This is because Git considers "useful" only the commits connected to the history of a branch.

Checking out on a branch is a common operation, allowing putting the repo in the state associated with that branch and possibly making and committing changes on it.

Remember in `git status` where it says `On branch master`? That's the currently checked-out branch, and where new commits will be made.

--- image branch checkout with HEAD moving from master to feature1 branch

##### `git checkout -b`

Checkout is the same command used to CREATE a new branch.

```
fpasqua@EisterBox:~/git/test$ git tree
* 31ebe42        (HEAD -> master) Modify A.py adding feature X
* bd883a1        Initial Commit
fpasqua@EisterBox:~/git/test$ git checkout -b feature1
Switched to a new branch 'feature1'
fpasqua@EisterBox:~/git/test$ git tree
* 31ebe42        (HEAD -> feature1, master) Modify A.py adding feature X
* bd883a1        Initial Commit
fpasqua@EisterBox:~/git/test$
```

As we can see from `git tree`, HEAD now points to the new feature1 branch instead of master! This means that if we now create a new commit by modifying a file `B.py` and committing, the new commit would be associated with feature1!

```
fpasqua@EisterBox:~/git/test$ git add B.py
fpasqua@EisterBox:~/git/test$ git commit -m "Modify B.py for feature 1"
[feature1 86b5f83] Modify B.py for feature 1
 1 file changed, 1 insertion(+), 1 deletion(-)
fpasqua@EisterBox:~/git/test$ git tree
* 86b5f83        (HEAD -> feature1) Modify B.py for feature 1
* 31ebe42        (master) Modify A.py adding feature X
* bd883a1        Initial Commit
```

In this way, you can work on multiple paths without mixing the changes of the various features, enabling easy troubleshooting of what's broken and WHY it broke.

Let's now assume that there are more commits in master:

```
fpasqua@EisterBox:~/git/test$ git checkout master
Switched to branch 'master'
fpasqua@EisterBox:~/git/test$ touch newfile
fpasqua@EisterBox:~/git/test$ git add newfile
fpasqua@EisterBox:~/git/test$ git commit -m "Add New File!"
[master 4c0a83f] Add New File!
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 newfile
fpasqua@EisterBox:~/git/test$ git tree
* 4c0a83f        (HEAD -> master) Add New File!
| * 86b5f83      (feature1) Modify B.py for feature 1
|/
* 31ebe42        Modify A.py adding feature X
* bd883a1        Initial Commit
```

Now, let's imagine that feature1 is complete and should be reintroduced into the master. This operation is called merge.

##### `git merge`

The merge works according to a predefined logic, which can also be changed with arguments, but it generally works like this:

1. `git checkout branch-that-receives` to go into the branch where I need to introduce `feature-branch`

2. `git merge feature-branch` to initiate the merge. A text editor will open to modify the default commit message (which generally is fine)

3. If the merge can be done by *fast-forwarding*, i.e., without creating a merge commit, it is done this way

4. If FF is not possible, a merge commit is created in `branch-that-receives` with all the modifications of `feature-branch`

5. If there are conflicts, such as portions of files modified by both branches, **merge conflict** mode activates, requiring human intervention.

--- merge and fast-forwarding images

Here's an example where there are no conflicts, but fast-forwarding can't be done:

```
fpasqua@EisterBox:~/git/test$ git tree
* 4c0a83f        (HEAD -> master) Add New File!
| * 86b5f83      (feature1) Modify B.py for feature 1
|/
* 31ebe42        Modify A.py adding feature X
* bd883a1        Initial Commit
fpasqua@EisterBox:~/git/test$ git merge feature1
Merge made by the 'ort' strategy.
 B.py | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
fpasqua@EisterBox:~/git/test$ git tree
*   13b099b      (HEAD -> master) Merge branch 'feature1'
|\
| * 86b5f83      (feature1) Modify B.py for feature 1
* | 4c0a83f      Add New File!
|/
* 31ebe42        Modify A.py adding feature X
* bd883a1        Initial Commit
```

As we can see, the feature1 branch remains completely untouched.

If instead Fast Forward is possible as in the case of `feature2` and `master`, unless specified otherwise in the merge options, no merge commit is created, but the target branch is fast-forwarded ahead:

```
fpasqua@EisterBox:~/git/test$ git tree
* baca0a1        (feature2) Add C.py for feature 2
*   13b099b      (HEAD -> master) Merge branch 'feature1'
|\
| * 86b5f83      (feature1) Modify B.py for feature 1
* | 4c0a83f      Add New File!
|/
* 31ebe42        Modify A.py adding feature X
* bd883a1        Initial Commit
fpasqua@EisterBox:~/git/test$ git merge feature2
Updating 13b099b..baca0a1
Fast-forward
 C.py | 2 ++
 1 file changed, 2 insertions(+)
 create mode 100644 C.py
fpasqua@EisterBox:~/git/test$ git tree
* baca0a1        (HEAD -> master, feature2) Add C.py for feature 2
*   13b099b      Merge branch 'feature1'
|\
| * 86b5f83      (feature1) Modify B.py for feature 1
* | 4c0a83f      Add New File!
|/
* 31ebe42        Modify A.py adding feature X
* bd883a1        Initial Commit
```

This command is extremely powerful and generally is the basis of simpler git flows in use.

Merge is an extremely vast command with dozens of options.

##### Git Merge Conflict

If during a merge of two divergent branches, where fast-forward is not possible, files are modified in both branches, git cannot autonomously decide which version of the file is correct. Maybe the user needs to combine some modifications from branch 1 and branch 2. In this case, the merge is blocked, and the repository is put in **Merge Conflict** state, extremely feared by beginners.

If the same file is modified in different parts by the two branches, git's default strategy is still able to merge, but if a modification is detected by both branches to the same portion of a file, a conflict is triggered.

Let's create a dedicated example. Imagine having this file called `D.py` in our branch:

```python
print("Numbers:")
for i in range(10):
    print(i)
print("Fibo:")
a = 1
b = 1
for x in range(5):
    a,b = b,a+b
    print(b)
```

Imagine that in two different branches, modifications are made to different lines of this same file.

We will have this tree:

```
fpasqua@EisterBox:~/git/test$ git tree
* bd642ef        (HEAD -> master) D.py: Print numbers on a line
| * 8d31cf8      (d_modify) D.py: Fix Fibo sequence
|/
* f6f37ea        First D.py version
[...]
```

We can see two branches, `master` and `d_modify`, with two commits. These commits act on different portions of the same file, as we can observe by doing `git show` of the commits at the top of the branches.

Remember that in this case, doing `git show <branch>` is equivalent to doing `git show <commit_hash>` with the commit that the branch label points to.

```diff
fpasqua@EisterBox:~/git/test$ git show master
commit bd642ef1b280cb7d5fedb1a6527345fea6df3424 (HEAD -> master)
Author: Federico Pasqua (eisterman) <federico.pasqua.96@gmail.com>
Date:   Sun Sep 22 15:51:21 2024 +0200

    D.py: Print numbers on a line

diff --git a/D.py b/D.py
index 8868120..581d593 100644
--- a/D.py
+++ b/D.py
@@ -1,6 +1,7 @@
 print("Numbers:")
 for i in range(10):
-    print(i)
+    print(i, end=', ')
+print()
 print("Fibo:")
 a = 1
 b = 1
```

-

```diff
fpasqua@EisterBox:~/git/test$ git show d_modify
commit 8d31cf803278978093f8612c1b73490497338e33 (d_modify)
Author: Federico Pasqua (eisterman) <federico.pasqua.96@gmail.com>
Date:   Sun Sep 22 15:50:16 2024 +0200

    D.py: Fix Fibo sequence

diff --git a/D.py b/D.py
index 8868120..75d2edb 100644
--- a/D.py
+++ b/D.py
@@ -1,10 +1,12 @@
 print("Numbers:")
-for i in range(10):
-    print(i)
+for j in range(10):
+    print(j)
 print("Fibo:")
-a = 1
+a = 0
 b = 1
+print(1, end=', ')
 for x in range(5):
     a,b = b,a+b
-    print(b)
+    print(b, end=', ')
+print()
```

As we can see, both modify, in the first few lines, the same portions of the file `D.py`. What happens if we try to merge d_modify into master?

```
fpasqua@EisterBox:~/git/test$ git merge d_modify
Auto-merging D.py
CONFLICT (content): Merge conflict in D.py
Automatic merge failed; fix conflicts and then commit the result.
fpasqua@EisterBox:~/git/test$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   D.py

no changes added to commit (use "git add" and/or "git commit -a")
```

Now we open D.py and we'll see this:

```python
print("Numbers:")
<<<<<<< HEAD
for i in range(10):
    print(i, end=', ')
print()
=======
for j in range(10):
    print(j)
>>>>>>> d_modify
print("Fibo:")
a = 0
b = 1
print(1, end=', ')
for x in range(5):
    a,b = b,a+b
    print(b, end=', ')
print()
```

The changes that could be auto-merged have been merged, but those in conflict haven't.

To resolve the conflict, you need to replace the part enclosed between the `<<<<<<` and `>>>>>>>` lines with the code you want to be present after the merge.

You can see how we are given the code versions in the 2 branches participating in the merge, with HEAD referring to the currently checked-out branch, so we have in the first part the code originating from `master`, and in the second one from `d_modify`.

This operation must be performed manually based on what the developers expect, and in this case, what I wanted was:

```python
print("Numbers:")
for j in range(10):
    print(j, end=', ')
print()
print("Fibo:")
a = 0
b = 1
print(1, end=', ')
for x in range(5):
    a,b = b,a+b
    print(b, en
```

and write it into the file.

Now that the conflict is resolved, it needs to be marked as such and continue the merge process:

```
fpasqua@EisterBox:~/git/test$ git add D.py
fpasqua@EisterBox:~/git/test$ git status
On branch master
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
        modified:   D.py

fpasqua@EisterBox:~/git/test$ git commit
[master 4c13b7a] Merge branch 'd_modify'
fpasqua@EisterBox:~/git/test$ git tree
*   4c13b7a      (HEAD -> master) Merge branch 'd_modify'
|\
| * 8d31cf8      (d_modify) D.py: Fix Fibo sequence
* | bd642ef      D.py: Print numbers on a line
|/
* f6f37ea        First D.py version
[...]
```

And the merge conflict is resolved.

##### `git reset`

Sometimes it might happen that it's better to change the branch state backward to solve a problem. Git Reset allows reseting the currently selected branch's (HEAD) state to a previous state.

There are 4 types of Reset:

1. **--soft** :
   
   - Moves the HEAD to a specified commit.
   - Keeps changes in the working directory and index (staging area).
   - Use case: Undo commit(s), but keep changes staged.

2. **--mixed** (default if no mode is specified):
   
   - Moves the HEAD to a specified commit.
   - Keeps changes in the working directory.
   - Unstages changes by resetting the index.
   - Use case: Undo commit(s) and unstage changes for re-committing.

3. **--hard** :
   
   - Moves the HEAD to a specified commit.
   - Resets both the index and working directory to the specified commit.
   - Discards all local changes.
   - Use case: Completely revert changes to a previous state.

4. **--merge** :
   
   - Moves the HEAD to a specified commit.
   - Keeps changes in the working directory that can be merged.
   - Resets the index but preserves uncommitted changes to be merged.
   - Use case: Safely update the current branch with a new base commit while keeping relevant changes.

5. **--keep** :
   
   - Moves the HEAD to a specified commit.
   - Keeps changes in the working directory if they do not conflict with the reset.
   - Use case: Update the HEAD while retaining work in progress that does not conflict.

Each reset mode has a specific use case. Personally, I've rarely had to use resets other than `reset --hard`, but this is probably a skill-issue on my part for not focusing enough on understanding when to use each type of git reset.

For example, let's imagine we want to reset our repository's state before the merge.

```
fpasqua@EisterBox:~/git/test$ git tree
*   4c13b7a      (HEAD -> master) Merge branch 'd_modify'
|\
| * 8d31cf8      (d_modify) D.py: Fix Fibo sequence
* | bd642ef      D.py: Print numbers on a line
|/
* f6f37ea        First D.py version
* baca0a1        Add C.py for feature 2
[...]
fpasqua@EisterBox:~/git/test$ git reset --hard bd642ef
HEAD is now at bd642ef D.py: Print numbers on a line
fpasqua@EisterBox:~/git/test$ git tree
* 8d31cf8        (d_modify) D.py: Fix Fibo sequence
| * bd642ef      (HEAD -> master) D.py: Print numbers on a line
|/
* f6f37ea        First D.py version
* baca0a1        Add C.py for feature 2
[...]
```

In this way, we have rolled back the merge, removing its commit.

##### Recovering from a git reset

Reset is an extremely destructive operation if done the wrong way, however, there is a way to recover lost commits from a reset.

If you remember the commit hash from before the reset, or through the command that indiscriminately lists all the commit movements in the repo, even the orphaned ones (`git reflog`), it's possible to recover the previous state thanks to another reset:

```
fpasqua@EisterBox:~/git/test$ git reflog
bd642ef (HEAD -> master) HEAD@{0}: reset: moving to bd642ef
4c13b7a HEAD@{1}: reset: moving to 4c13b7a
f6f37ea HEAD@{2}: reset: moving to f6f37ea
4c13b7a HEAD@{3}: commit (merge): Merge branch 'd_modify'
bd642ef (HEAD -> master) HEAD@{4}: checkout: moving from d_modify to master
8d31cf8 (d_modify) HEAD@{5}: commit (amend): D.py: Fix Fibo sequence
1d8bf5c HEAD@{6}: checkout: moving from master to d_modify
bd642ef (HEAD -> master) HEAD@{7}: reset: moving to bd642ef
a32e387 HEAD@{8}: merge d_modify: Merge made by the 'ort' strategy.
bd642ef (HEAD -> master) HEAD@{9}: commit: D.py: Print numbers on a line
```

We can see that the last commit where an operation was registered before the reset was `4c13b7a`. So we can inspect this orphaned commit to see if it’s where we want to return

```diff
fpasqua@EisterBox:~/git/test$ git show 4c13b7a
commit 4c13b7a02077adb301a23c359418413bd046c129
Merge: bd642ef 8d31cf8
Author: Federico Pasqua (eisterman) <federico.pasqua.96@gmail.com>
Date:   Sun Sep 22 16:06:36 2024 +0200

    Merge branch 'd_modify'

diff --cc D.py
index 581d593,75d2edb..ea40bd3
--- a/D.py
+++ b/D.py
@@ -1,11 -1,12 +1,13 @@@
  print("Numbers:")
- for i in range(10):
-     print(i, end=', ')
+ for j in range(10):
 -    print(j)
++    print(j, end=', ')
 +print()
  print("Fibo:")
- a = 1
+ a = 0
  b = 1
+ print(1, end=', ')
  for x in range(5):
      a,b = b,a+b
-     print(b)
+     print(b, end=', ')
+ print()
```

And it looks like the commit from which we started with reset! Therefore, by doing:

```
fpasqua@EisterBox:~/git/test$ git reset --hard 4c13b7a
HEAD is now at 4c13b7a Merge branch 'd_modify'
fpasqua@EisterBox:~/git/test$ git tree
*   4c13b7a      (HEAD -> master) Merge branch 'd_modify'
|\
| * 8d31cf8      (d_modify) D.py: Fix Fibo sequence
* | bd642ef      D.py: Print numbers on a line
|/
* f6f37ea        First D.py version
* baca0a1        Add C.py for feature 2
*   13b099b      Merge branch 'feature1'
|\
| * 86b5f83      Modify B.py for feature 1
* | 4c0a83f      Add New File!
|/
* 31ebe42        Modify A.py adding feature X
* bd883a1        Initial Commit
```

We have successfully recovered the commits lost with the reset.

This operation relies on the fact that Git doesn’t delete orphaned commits internally immediately, but once every so often (garbage collection); thus, one can recover the damage done by a wrong reset for a while.

This is only possible with the commit data, not with data in the staging area or working directory lost from a reset.

##### `git rebase`

Rebase is, at the same time, a simple yet extremely powerful operation, with the use case effectively being to "rewrite history," i.e., modify/recreate an entire chain of commits by changing certain pieces, like merging multiple commits together.

For now, I’ll omit it from the guide, but hopefully, I can write a more detailed article about it in the future.

### Remote interaction

<work in progress>

## Conclusions

Git is an extremely powerful tool with many things to learn and a learning curve that can seem quite steep, but it offers many advantages.

I hope that this collection of information and tips can be useful for someone!
