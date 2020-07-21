---
title: Merging an upstream repository
layout: post
comments: true
published: false
description: 
keywords: 
---

In your local clone of your forked repository, you can add the original 
GitHub repository as a "remote". ("Remotes" are like nicknames for the 
URLs of repositories - origin is one, for example.) Then you can fetch 
all the branches from that upstream repository, and rebase your work 
to continue working on the upstream version. In terms of commands 
that might look like:

Change the current working directory to your local project.

```
cd /path/to/my/project
```

Add the remote, call it "upstream":

```
git remote add upstream https://github.com/jdoe/coolapp.git
```

Show all remote repositories in verbose mode

```
git remote -v
```

Fetch all the branches of that remote into remote-tracking branches, 
such as upstream/master:

```
git fetch upstream
```

List all remote branches:

```
git branch -r
```

Check out the branch you wish to merge to. Usually, you will merge into master.

```
git checkout master
```

Pull the desired branch from the upstream repository. 
This method will retain the commit history without modification.

```
git pull upstream master
```

If there are conflicts, resolve them. For more information, 
see "Addressing merge conflicts".

Commit the merge.

Review the changes and ensure they are Okay.

Push the merge to your GitHub repository.

```
git push origin master
```


## Links

* <https://help.github.com/articles/configuring-a-remote-for-a-fork/>
* <https://help.github.com/articles/merging-an-upstream-repository-into-your-fork/>
* <https://stackoverflow.com/questions/11266478/git-add-remote-branch?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa>