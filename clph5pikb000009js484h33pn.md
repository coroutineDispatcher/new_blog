---
title: "Using git from Android Studio. A quick guide."
datePublished: Sun Oct 31 2021 09:30:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5pikb000009js484h33pn
slug: using-git-from-android-studio-a-quick-guide

---


We all know how important version control is. One can save a lot of time in case conflicts occur or things go really bad. But we can still argue which tool is the best for using Git. That's because Git in general is abstract, and visualizing it is somehow hard, as this also needs to go along with everyone's head. I would not blame anyone if they fantasize git differently in their heads, and then argue about the tool. In my opinion, everyone has the right to argue on toolings just because of that.

# What are some of the preferences though?
Here I am listing only the tools that I have experience with.

## Terminal

Sometimes I try to keep it very conservative. I just use the terminal. It's over there, right below my IDE and I am the laziest person ever lived. The commands are usually muscle memory, even though I have to write something like this:

```
git status
git add whatever/
git status
// what did I just do?
git status
git status
git status
// ah yea
git commit -m "Some related message"
git status
// did I commit?
git status
git status
git push
```

And it is still easier than using any tool. The reason for just using the terminal is actually very logical for me: Different companies, use different tools. If one doesn't know how to use git in the terminal, switching tools would be the hardest thing ever. Also, the terminal takes changes with the branch once you checkout in a new one, and that would be a huge red flag. Git visualization tools forbid such things.

## Sourcetree.
Another option for me has been Sourcetree. During merges or rebases, I do not feel comfortable with the terminal, especially when it brings me a file to edit after doing `git merge <branch_name>`. Therefore Sourcetree has been my number one solution for merging, rebasing branches as well as cherry-pick-ing and tagging commits. The UI is very friendly. However Sourcetree has problems sometimes positioning the view in the right commit after switching a branch, and it looks very laggy.

## Android Studio
Being an Android Developer, I do not feel well about the fact that I have always ignored this section in the IDE:

![](/images/git_as.jpg)

Therefore, I thought I should give it a try. I closed Sourcetree and then I started working with Android Studio. I can definitely say that the one and only issue I faced was
the word `Update` which in this case stands for `Pull` (from remote). And that actually got me thinking: Isn't that even more accurate? Since this debate between `Pull` and `Merge` as terms would never be resolved, why not actually separate them as concepts completely? Update would be the new pull and merge would continue to be merge. Even with introducing this word though, the debate still stands. However, the most important part would be to know what you are doing.

### Let's start with Commits

Usually Android Studio tends to `git add` files by default. Just be careful when creating new files, or deleting and reverting them. So let's commit one file from Android Studio. The project is in Flutter, but it is sufficient for the example. 

![](/images/git_commit_as.png)

### Cherry-pick/Patches and commits

In case you need changes from only one commit, the Android Studio visualization is not different from most of the other Git clients over there. Just right-click to any of the commits you would like to work with and all the options you need would be there:

![](/images/git_branch_operations.png)

{{< admonition >}}
As you may already know, the commit has a hash which Android Studio calls it `Revision Number`.
{{< /admonition >}}

Applying patches is one of the coolest things from Android Studio. You can immediately apply a patch from the clipboard.

![](/images/git_as_patch.png)

In case what is copied on the clipboard is not a git patch, you won't be able to paste.

### Reverting small/big changes

Sometimes you get lost with just one method. And you just want to revert it and start from the beginning. This is very user-friendly from Android Studio. Just click the highlighted bar on the left, close to the numbers that count the lines of code and that would be it:

![](/images/git_as_revert.png)

But sometimes you just do not want to revert, but at the moment you want to see how it is in the base branch and compare it with your work. 

![](/images/git_as_compare.jpg)

It is pretty helpful.

### Pulling/Updating or just merging remote into yours

The whole git section can be found by default in the down toolbar of the IDE, however, you can easily move it to whichever place you prefer just by drag&drop.
This is the section to view everything in your local and remote configurations. To update from any branch, just hit Update as shown in the image below:

![](/images/git_as_update.png)

### Checkout

As you can see, I am currently in `some_branch_name`. If I would want to checkout to a (new) branch, it's just one click away:

![](/images/git_checkout_as.png)

### Fetching

In case you just want to see if there is something new on remote, but you do not care about updating, fetch can be found right here:

![](/images/git_as_fetch.png)

### Tagging

Tagging is also simple in Android Studio, as seen in the `Cherry-pick/Patches and commits` section (see image). However, there is another way. From the upper toolbar of the IDE, you can have a dialog for tagging any commit you like, but you would need to copy the commit hash before doing that:

![](/images/git_new_tag.png)

Anyhow, in comparison with SourceTree, it lacks just one small thing. You cannot immediately push the tag from this dialog:

![](/images/git_tag_no_push.png)

Then you would have to be careful not to forget pushing the tag when pushing changes to remote:

![](/images/git_push_tag_current_branch.png)

### Let's have some fun now. Conflicts.

After resolving conflicts with the Android Studio Conflict resolver, I can say that I am not going to drop using git from AS anymore. In case one needs to know a little bit more about merging or rebasing, they can find it [here, in this article that I wrote some time ago](https://coroutinedispatcher.com/posts/git_me_baby_one_more_time/).

![](/images/git_merge_conflicts.png)

Just click the `Merge` button to open the conflict resolver. Now another window will pop up:

![](/images/git_conflict_resolver.png)

In this case, you know 100% what you are doing, what the others have been doing, and what you need to do after.

### Ok, I still need the terminal for three things:

```
git merge --abort
git rebase --abort
git stash "stash_message_or_empty"
```

These just seem easier from the terminal, and I will continue to do so, but that's just me, being lazy to find where the buttons are.

## Closing notes
Hopefully, I have given a general idea on using Git from Android Studio. It is super easy to tackle, user-friendly, and smooth. What other git clients do you prefer?
