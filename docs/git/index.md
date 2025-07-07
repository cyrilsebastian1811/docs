# __:material-git: GIT__

## __Remotes__

A "remote" refers to another copy of the repository that is usually hosted on a remote server. Remotes are used to track the same project from different locations. This allows multiple people to work together on a project from different locations.

Here are some key points to understand about remotes in Git:
- Origin: By convention, the default remote is usually named "origin". This is typically the repository where you cloned from.

- Fetch and Push: You interact with remotes by "fetching" from them to get their data, and "pushing" to them to share your data. Fetching involves getting the latest changes from the remote without merging them with your local branches. Pushing involves sending your changes to the remote to share them with others.

- Tracking Branches: A tracking branch in Git is a local branch that is connected to a remote branch. When you push and pull on that branch, it automatically pushes and pulls to the remote branch that it is tracking.

- Managing Remotes: You can manage your remotes with various Git commands. For example,
```{ .bash }
git remote add <name> <url>                 # add a new remote
git remote remove <name>                    # remove a remote
git remote rename <old_name> <new_name>     # rename a remote
git remote show <name>                      # print information about a remote
```

## __Refs__

Git references (refs) are pointers to specific commits in the Git history. They are used to give human-readable names to commit hashes.

There are several types of refs in Git:

1. __Branches__: These are the most common type of ref. Each branch is a ref that points to the latest commit in a series of commits. The branch automatically moves forward as new commits are made.

2. __Tags__: Tags are refs that point to specific commits. They are typically used to mark specific points in the repository's history, such as releases. Unlike branches, tags do not move forward as new commits are made.

3. __Remote-tracking branches__: These are refs that track branches in remote repositories. They are automatically updated when you fetch or pull from the remote repository.

4. __HEAD__: This is a special ref that points to the current branch (or the current commit if the repository is in a detached HEAD state).

5. __Stash__: This is a special ref that points to the commit created when you run git stash. It allows you to save changes that you don't want to commit immediately.

6. __Refspecs__: Refspecs are used to map local and remote branches. A refspec maps a reference in the source repository to a reference in the destination repository. They are used by commands like git fetch, git push, and git pull to determine which commits to fetch or push.


### Refspecs

These are the various forms of refspecs:

=== "__Branches__"

    `+refs/heads/*:refs/remotes/origin/*` - Bitbucket stores branch refs under `refs/heads/`. The `*` is a wildcard that matches any branch.
    
     This refspec maps all branches on source repository to corresponding `orgin/*` branch in local repository, even if it does not result in a fast-forward update.

=== "__Tags__"

    `+refs/tags/*:refs/tags/*` - Bitbucket stores tag refs under `refs/tags/`. The `*` is a wildcard that matches any tag. The `*` is a wildcard that matches any tag name.

    This refspec maps all tags on the source repository to corresponding `tags/*` in the local repository, even if it does not result in a fast-forward update.

=== "__Pull Requests__"

    `+refs/pull-requests/*/from:refs/remotes/origin/PR-*` - This refspec is specific to Bitbucket. Bitbucket stores pull request refs under `refs/pull-requests/`. The `*` is a wildcard that matches any pull request number. The `from` ref points to the source commit of the pull request. 
    
    This refspec maps all pull-requests on source repository to corresponding `orgin/PR-*` branch in local repository, even if it does not result in a fast-forward update.

!!! example

    ```{ .bash }
    git fetch --no-tags --force --progress -- https://bitbucket.es.ad.adp.com/scm/awd/automation_scripts.git +refs/pull-requests/7/from:refs/remotes/origin/PR-7 # timeout=10
    ```

    This command fetches branches and/or tags (collectively, "refs") from one or more other repositories, along with the objects necessary to complete their histories. 



## __Merge Strategies__

In this section, we will explain and illustrate various merge strategies used in version control systems.

=== "Merge Commit `--no-ff`"

    The `--no-ff` option in Git stands for "no fast-forward". This is the default merge strategy in Git. It always creates a new merge commit and updates the target branch to it, even if the source branch is already up to date with the target branch.

    This strategy is useful when you want to preserve the history of commits on the source branch without flattening them into a single commit. It allows you to maintain a visual representation of the feature branch in your commit history.

    Here's an example of how to use it:

    ```bash
    git checkout master
    git merge --no-ff feature-branch
    ```

    In terms of a visual representation, consider the following scenario:

    ``` mermaid
    %%{init: {'logLevel': 'debug', 'theme': 'default', 'gitGraph': {'showBranches': true, 'showCommitLabel':true, 'rotateCommitLabel': true, 'parallelCommits': true} } }%%
        gitGraph
            commit id: "A" tag: "base"
            branch feature
            checkout main
            commit id: "B"
            commit id: "C" tag: "main tip"
            checkout feature
            commit id: "D"
            commit id: "E"
            commit id: "F" tag: "feature tip"
    ```

    If we are on `master` and we merge `feature` with `--no-ff`, we force a new commit and the history looks like this:

    ``` mermaid
    %%{init: {'logLevel': 'debug', 'theme': 'default', 'gitGraph': {'showBranches': true, 'showCommitLabel':true, 'rotateCommitLabel': true, 'parallelCommits': true} } }%%
        gitGraph
            commit id: "A" tag: "base"
            branch feature
            checkout main
            commit id: "B"
            commit id: "C" tag: "main tip"
            checkout feature
            commit id: "D"
            commit id: "E"
            commit id: "F" tag: "feature tip"
            checkout main
            merge feature id: "G" tag: "new merge commit"
    ```

    Merge commits are unique against other commits in the fact that they have two parent commits. When creating a merge commit Git will attempt to auto magically merge the separate histories for you. If Git encounters a piece of data that is changed in both histories it will be unable to automatically combine them. This scenario is a version control conflict and Git will need user intervention to continue. 

=== "Fast-forward Merge `--ff`"

    The `--ff` option in Git stands for "fast-forward". If the source branch is out of date with the target branch, create a merge commit. Otherwise, update the target branch to the latest commit on the source branch.

    This strategy is useful when you want to keep a linear commit history, as it avoids creating unnecessary merge commits when the target branch has not diverged from the point where the source branch was created.

    Here's an example of how to use it:

    ```bash
    git checkout master
    git merge --ff feature-branch
    ```

    In terms of a visual representation, consider the following scenario:

    ```mermaid
    %%{init: {'logLevel': 'debug', 'theme': 'default', 'gitGraph': {'showBranches': true, 'showCommitLabel':true, 'rotateCommitLabel': true, 'parallelCommits': true} } }%%
        gitGraph
            commit id: "A"
            commit id: "B"
            commit id: "C" tag: "main tip"
            branch feature
            commit id: "D"
            commit id: "E"
            commit id: "F" tag: "feature tip"
    ```

    If we are on `master` and we merge `feature` with `--ff`, `master` will be fast-forwarded to `feature`:

    ```mermaid
    %%{init: {'logLevel': 'debug', 'theme': 'default', 'gitGraph': {'showBranches': true, 'showCommitLabel':true, 'rotateCommitLabel': true, 'parallelCommits': true} } }%%
        gitGraph
            commit id: "A"
            commit id: "B"
            commit id: "C"
            branch feature
            commit id: "D"
            commit id: "E"
            commit id: "F" tag: "feature tip"
            checkout main
            commit id: "D "
            commit id: "E "
            commit id: "F " tag: "main tip"
    ```

    Here, `master` now points to the same commit as `feature`. If `master` had diverged, a new merge commit would have been created.


=== "Fast-forward Only Merge `--ff-only`"

    The `--ff-only` option in Git stands for "fast-forward only". If the source branch is out of date with the target branch, reject the merge request. Otherwise, update the target branch to the latest commit on the source branch.

    This strategy is useful when you want to ensure a linear commit history and prevent accidental merge commits. It guarantees that a merge commit will not be created.

    Here's an example of how to use it:

    ```bash
    git checkout master
    git merge --ff-only feature-branch
    ```

=== "Rebase and Merge `rebase + merge --no-ff`"

    The `rebase` and `merge --no-ff` strategy in Git involves two steps. Commits from the source branch onto the target branch, creating a new non-merge commit for each incoming commit. Creates a merge commit to update the target branch. The PR branch is not modified by this operation.

    This strategy is useful when you want to maintain a clean, linear history on the target branch but also want to preserve the context of a feature branch in the form of a merge commit.

    Here's an example of how to use it:

    ```bash
    git checkout feature-branch
    git rebase master
    git checkout master
    git merge --no-ff feature-branch
    ```

    In terms of a visual representation, consider the following scenario:

    ```mermaid
    %%{init: {'logLevel': 'debug', 'theme': 'default', 'gitGraph': {'showBranches': true, 'showCommitLabel':true, 'rotateCommitLabel': true, 'parallelCommits': true} } }%%
        gitGraph
            commit id: "A"
            commit id: "B"
            branch feature
            checkout main
            commit id: "C"
            commit id: "D" tag: "main tip"
            checkout feature
            commit id: "E"
            commit id: "F"
            commit id: "G" tag: "feature tip"
    ```

    After the rebase, the situation will look like this:

    ```mermaid
    %%{init: {'logLevel': 'debug', 'theme': 'default', 'gitGraph': {'showBranches': true, 'showCommitLabel':true, 'rotateCommitLabel': true, 'parallelCommits': true} } }%%
        gitGraph
            commit id: "A"
            commit id: "B"
            commit id: "C"
            commit id: "D" tag: "main tip"
            branch feature
            commit id: "E'" type: REVERSE
            commit id: "F'" type: REVERSE
            commit id: "G'" tag: "feature tip" type: REVERSE
    ```

    After the `merge --no-ff`, `master` will be updated with a new merge commit:

    ```mermaid
    %%{init: {'logLevel': 'debug', 'theme': 'default', 'gitGraph': {'showBranches': true, 'showCommitLabel':true, 'rotateCommitLabel': true, 'parallelCommits': true} } }%%
        gitGraph
            commit id: "A"
            commit id: "B"
            commit id: "C"
            commit id: "D"
            branch feature
            commit id: "E'" type: REVERSE
            commit id: "F'" type: REVERSE
            commit id: "G'" tag: "feature tip" type: REVERSE
            checkout main
            merge feature id: "H" tag: "main tip"
    ```

=== "Rebase and Fast-forward `rebase + merge --ff-only`"

    The `rebase` and `merge --ff-only` strategy in Git involves two steps. First, the commits from the source branch are replayed onto the target branch, creating a new non-merge commit for each incoming commit. Then, the target branch is fast-forwarded to the latest commit. The source branch is not modified by this operation.

    This strategy is useful when you want to maintain a clean, linear history on the target branch and avoid creating a merge commit.

    Here's an example of how to use it:

    ```bash
    git checkout feature-branch
    git rebase master
    git checkout master
    git merge --ff-only feature-branch
    ```

    In terms of a visual representation, consider the following scenario:

    ```mermaid
    %%{init: {'logLevel': 'debug', 'theme': 'default', 'gitGraph': {'showBranches': true, 'showCommitLabel':true, 'rotateCommitLabel': true, 'parallelCommits': true} } }%%
        gitGraph
            commit id: "A"
            commit id: "B"
            branch feature
            checkout main
            commit id: "C"
            commit id: "D" tag: "main tip"
            checkout feature
            commit id: "E"
            commit id: "F"
            commit id: "G" tag: "feature tip"
    ```

    After the rebase, the situation will look like this:

    ```mermaid
    %%{init: {'logLevel': 'debug', 'theme': 'default', 'gitGraph': {'showBranches': true, 'showCommitLabel':true, 'rotateCommitLabel': true, 'parallelCommits': true} } }%%
        gitGraph
            commit id: "A"
            commit id: "B"
            commit id: "C"
            commit id: "D" tag: "main tip"
            branch feature
            commit id: "E'" type: REVERSE
            commit id: "F'" type: REVERSE
            commit id: "G'" tag: "feature tip" type: REVERSE
    ```

    After the `merge --ff-only`, `master` will be fast-forwarded to the latest commit:

    ```mermaid
    %%{init: {'logLevel': 'debug', 'theme': 'default', 'gitGraph': {'showBranches': true, 'showCommitLabel':true, 'rotateCommitLabel': true, 'parallelCommits': true} } }%%
        gitGraph
            commit id: "A"
            commit id: "B"
            commit id: "C"
            commit id: "D"
            branch feature
            commit id: "E'" type: REVERSE
            commit id: "F'" type: REVERSE
            commit id: "G'" tag: "feature tip" type: REVERSE
            checkout main
            commit id: "E' " type: REVERSE
            commit id: "F' " type: REVERSE
            commit id: "G' " tag: "main tip" type: REVERSE
    ```

    Here, `master` now points to the same commit as `feature`. If `master` had diverged, the merge would have been rejected.

=== "Squash Merge `--squash`"

    The `--squash` option in Git combines all commits from the source branch into one new non-merge commit on the target branch. This strategy is useful when you want to condense all the changes into a single commit, which can make the history cleaner and easier to understand.

    Here's an example of how to use it:

    ```bash
    git checkout master
    git merge --squash feature-branch
    git commit -m "Squashed commit of the feature-branch changes"
    ```

    This will combine all the changes from `feature-branch` into a single commit and apply it to `master`.

    In terms of a visual representation, consider the following scenario:

    ``` mermaid
    %%{init: {'logLevel': 'debug', 'theme': 'default', 'gitGraph': {'showBranches': true, 'showCommitLabel':true, 'rotateCommitLabel': true, 'parallelCommits': true} } }%%
        gitGraph
            commit id: "A" tag: "base"
            branch feature
            checkout main
            commit id: "B"
            commit id: "C" tag: "main tip"
            checkout feature
            commit id: "D"
            commit id: "E"
            commit id: "F" tag: "feature tip"
    ```

    After the `merge --squash`, the situation will look like this:

    ```
    A---B---C feature
     /
    D---E---F---G master
    ```

    Here, `G` is the new commit made by the `merge --squash` operation. This commit includes all the changes made in the `A`, `B`, and `C` commits from the `feature` branch.

=== "Squash and Fast-forward Only Merge `--squash --ff-only`"

    The `--squash --ff-only` option in Git combines all commits from the source branch into one new non-merge commit on the target branch if the source branch is up to date with the target branch. If the source branch is out of date with the target branch, it rejects the merge request.

    This strategy is useful when you want to condense all the changes into a single commit and maintain a linear commit history, but also prevent accidental merge commits.

    Here's an example of how to use it:

    ```bash
    git checkout master
    git merge --squash --ff-only feature-branch
    git commit -m "Squashed commit of the feature-branch changes"
    ```

    This will combine all the changes from `feature-branch` into a single commit and apply it to `master` if `master` has not diverged from the point where `feature-branch` was created. If `master` has diverged, the merge will be rejected.

    In terms of a visual representation, consider the following scenario:

    ```
    A---B---C feature
     /
    D---E---F master
    ```

    If we are on `master` and we merge `feature` with `--squash --ff-only`, `master` will be updated with a new commit that includes all the changes from `feature`:

    ```
    A---B---C feature
     /
    D---E---F---G master
    ```

    Here, `G` is the new commit made by the `merge --squash --ff-only` operation. This commit includes all the changes made in the `A`, `B`, and `C` commits from the `feature` branch. If `master` had diverged, the merge would have been rejected.