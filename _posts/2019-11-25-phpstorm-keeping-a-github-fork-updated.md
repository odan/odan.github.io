---
title: PhpStorm - Keeping A GitHub Fork Updated
layout: post
comments: true
published: false
description:
keywords: PhpStorm git
---

I forked a GitHub repo `octo-org/octo-repo` to `johndoe/octo-repo` and want to keep it updated.

## Track

After I forked the repo to your Github account, I did this one time:

* Copy the GitHub repo url

![image](https://user-images.githubusercontent.com/781074/69576157-0b90b780-0fcc-11ea-9a14-c7a15e26d860.png)

* In PhpStorm open the menu: VCS > Git > Clone ...
* Paste the URL and enter a directory name. Click "Clone"
* To add the remote branch open the menu: VCS > Git > Remotes...
* Click the "+" button
* Paste the GitHub repo URL and enter the name "upstream"
* Save with "Ok"
* To fetch the remote branches from the upstream repo choose: VCS > Git > Fetch

## Update

Each time you want to update (pull changes), from your local master branch
you have to 1. fetch the upstream, 2. rebase upstream master.

Option 1: Manual

* From the main menu, choose VCS > Git > Pull. The Pull Changes dialog opens.
* Select the remote "upstream" from the dropdown.
* Under "Select to merge" choose "upstream/master"
* Leave all other options as it is
* Click "Pull"

Option 2: Automatic

* From the main menu, choose: VCS > Update Project (Ctrl+T).

## Rebase

* Checkout the local master branch you want to rebase.
* Select the "Remote Origin Master" upstream branch > Merge into using current using Rebase
* Push the changes to the Repository clone master.

It is best practice to always rebase your local commits when you pull before pushing them. 
As nobody knows your commits yet, nobody will be confused when they are rebased but the
additional commit of a merge would be unnecessarily confusing. 
Published commits are, however, usually merged, for example when branches are merged.

## Read more

* [Sync with a remote repository](https://www.jetbrains.com/help/phpstorm/sync-with-a-remote-repository.html)

 



 