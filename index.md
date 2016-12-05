---
layout: post
title:  "Come on, git"
date:   2016-12-05 09:17:00 -0800
---
Git is probably the biggest common-among-developers-of-all-languages programs that we encounter on a daily basis outside of our terminals, yet many of us use it in a very straight forward manner and hope we don’t ever run into issues with it.

But how many of us have errantly committed to master by mistake, or committed a bunch of work, forgetting to pull from origin first so we end up with the obnoxious

```
↳ git push origin master
To ../origin
 ! [rejected] master -> master (fetch first)
error: failed to push some refs to ‘../origin’
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., ‘git pull …’) before pushing again.
hint: See the ‘Note about fast-forwards’ in ‘git push — help’ for details.
```

Then you do a git pull origin master but you end up with a merge conflict or two. Or it creates a merge commit and you were told merge commits aren’t good. Or you follow the above message like a good developer and then you get

```
↳ git pull origin master
From ../origin
 * branch            master     -> FETCH_HEAD
fatal: refusing to merge unrelated histories
```


(ノಠ益ಠ)ノ彡┻━┻ I LITERALLY JUST DID WHAT IT TOLD ME TO DO AND IT IS SAYING IT CAN’T DO IT?!

Much like Linus, git is not known for being nice or subtle. (Which is a sad thing because software can and should be friendly.)

So lets step through some common git error messages and see how we can fix them!

## Refusing to merge unrelated histories

This is likely due to a scenario where someone made a remote repo and pushed their initial commit(s) to it, and instead of cloning it, you created a new local repo and added the remote repo as origin — you’ve now got two different “initial” commits, which, unlike the Highlander movies, there really can only be one.

```
remote git commit stack      local git commit stack
        +----------+                 +----------+
HEAD -> | 55fe3bf  |          HEAD-> | fc4daf3  |
        +----------+                 +----------+
```

As we can see from my crude graphic, the remote git repository has a different commit history than the local; there is no common commit between the two of them.
However we can resolve this with a git pull --rebase. This will pop off any local commits you have in your repo, pull the remote ones, place them on the stack, and then push your commits onto the stack after.

```
remote git commit stack      local git commit stack
                                     +----------+
                             HEAD -> | fc4daf3  |
        +----------+                 +----------+
HEAD -> | 55fe3bf  |                 | 55fe4bf  |
        +----------+                 +----------+
```

So now we have our conflicting commit as the second commit in the branch and are sharing 55fe4bf. Pushing this branch back up to origin will be successful now!

## Commit to the wrong branch

If anyone works in anything similar to git flow (although I would argue git flow is way more complex than you likely need), you’ve had the point hammered into your head that you should never work on master (or develop. Or whatever branch it is.) But, like, hey, we’re all human.

### If your changes are staged<sup>[1](#1)</sup>, but not committed<span id="1-source"></span>
This one is the easy one! Make a new branch and check it out, then commit your changes!

```
git branch [branchname]
git checkout [branchname]
```

(or do both commmands at the same time with `git checkout -b [branchname])

### If your changes are committed, but not pushed<span id="2-source"></span>
1. Checkout the branch you committed to by mistake
2. Using git log find the commit before all the stuff you added, and copy it's hash.
3. `git reset [commit hash]`<sup>[2](#2)</sup>
4. `git checkout -b [new branch name]`
5. `git add . && git commit`
 
`git reset [commit hash]` will place the `HEAD` pointer (which tells git what the last commit in a branch was) at this commit.  Anything _after_ that commit in the log, is no longer part of this branch, and will be unstaged. This makes it so all your changes are just now uncommitted work on that branch.  You checkout a new branch, add the files back to staging and then commit them to your new branch.

#### Update
A [reader asked](https://github.com/toddself/git-undo.com/issues/2) if you can do this without having to restage the commits and re-write a commit message.  

This can be done by making a new branch off your current one, and then resetting the head of the current one back to where you wanted it:

1. `git checkout -b my-new-branch`
1. `git checkout -`<span id="3-source"><sup>[3](#3)</sup></span>
1. `git reset --hard [commit hash]`

I'm not a huge fan of this since it presents some additional pitfalls that you might stumble on.  `reset --hard` causes the commits to disappear from your current branch.  If you do something wrong or clumbsy, it's possible to remove the commits entirely, and then you might have to retrieve them from the reflog (but that's for another time).

### If your changes are committed and pushed
Sorry, bub. You really shouldn't ever edit remote repository history on shared branches, as this will cause everyone to have a very sad day. You're just going to have to take your lumps on this one. 

Why?

Because if someone has pulled down your changes locally and starts working on a new branch based on that version, and then you remove those commits from the branch, their local branch copy will no longer jive with the history on the remote branch. (But, yes! That can be fixed as well! However! Don't force people to do it!)

#### 1
_Wait.  What the fuck is "staged, but not committed?"_

When you're using git and you are `git add`ing files (If you're using `git commit -a` that is telling git to automatically add all changed files, so you're implicitly `git add`ing there as well!), you are actually moving them into what's called `staging`.  This is a list of files, at the specific version when you added them, that you are going to commit.

[🔙](#1-source)

#### 2
_I've seen a lot of git commands use `HEAD~[some number]` or even more esoteric shit like `HEAD^^`. What the shit is that?_

Those are all short-hand for specific commits.  `HEAD~[some number]` means [some number] commits before the commit that `HEAD` is pointed at.

```
+---------+
| 23dae4a | <- HEAD
+---------+
| ff092aa |
+---------+
| 452a55a |
+---------+
```

In the example above, `HEAD~1` is the commit with the hash `ff092aa`.  `HEAD~2` is `452a55a`.  `HEAD^` is an even lazier way to type `HEAD~1`.  `HEAD^^` is `HEAD~2`.

[🔙](#2-source)

#### 3
_What the shit is `git checkout -`?_

`git checkout -` works the same as `cd -` does in unix-y places -- it returns you to where you last where!
If I'm on `master` and I do `git checkout -b my-new-branch` and then `git checkout -`, it'll bring you right back to master.

[🔙](#3-source)
