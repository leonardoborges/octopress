---
date: 2011-05-29
excerpt: I've been working fulltime with Git for a while now and I've noticed a pattern of commands that emerge every so often, but not often enough that would...
comments: true
layout: post
publish: true
categories: [git]
title: "A Few Useful Git Commands"
---

I've been working fulltime with [Git][1] for a while now and I've noticed a pattern of commands that emerge every so often, but not often enough that would make me remember them.


That's why I decided to post them here so I know where to look in the future. And of course they can turn out to be useful to someone else too.

So without further ado, here they are:

``` bash
#undo last commit
git reset HEAD^

#show files in a given commit
git show --pretty="format:" --name-only rev_number

#remove untracked files and directories
git clean -f -d

#track remote branch
git branch --track branch_name origin/master

# given you created a new local branch 'branch_name'
# pushes 'branch_name' to 'origin/branch_name', creating the remote branch for you
git push origin branch_name

#delete remote branch
git push origin :remote_branch_name
```


I'll keep this list up to date and add more useful stuff as need arises. Enjoy.

[1]: http://git-scm.com/