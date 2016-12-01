---
layout: post
title:  "Come on, git"
date:   2016-12-02 09:00:00 -0800
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

### If your changes are staged, but not committed

