---
title: "Git me baby one more time"
date: 2021-04-03T12:15:57+02:00
draft: false
---

&nbsp;

![Photo by Yancy Min @Unsplash](/images/git_me_baby.jpg)

&nbsp;

Long time no see. Nothing new on Android from my side this days. I have mostly been focused in learning some new tech stack and some automation. But today I would like to talk about some basic git commands/concepts and situations.

## Let's immediately jump to some ancient debate: Rebase vs Merge

For any reader who doesn't know the difference, I am trying to illustrate it in the below diagram. Let's consider these 2 branches:

&nbsp;

![Branch_A_B_Illustration](/images/git_rebase_vs_merge.png)

&nbsp;

### git merge

Let's start with merge first. From the official git documentation:

{{< admonition type=info title="Name">}}
git-merge - Join two or more development histories together
{{</ admonition >}}

{{< admonition type=info title="Description">}}
Incorporates changes from the named commits (since the time their histories diverged from the current branch) into the current branch.
{{</ admonition >}}

In other words, you don't care what happened to previous commits of branch B. You just take the last one, and put it together with the current one, as a whole:

&nbsp;

![git_merge](/images/git_merge.png)

&nbsp;

Think of the merging point as a huge commit that has basically everything, your current work, including the one you merged. If you think further, that's what happens after merging a Pull Request (or a Merge Request).

For more information, you can refer to the official [documentation](https://git-scm.com/docs/git-merge).

### git rebase

Let's try the same scenario, but now talking about rebase. From the official docs:

{{< admonition type=info title="Name">}}
git-rebase - Reapply commits on top of another base tip
{{</ admonition >}}

{{< admonition type=info title="Description">}}
If \<branch\> is specified, git rebase will perform an automatic *git switch* \<branch\> before doing anything else. Otherwise it remains on the current branch.
{{</ admonition >}}

It is practically vague, from only those two short notations. Therefore, a longer description available can be found [here](https://git-scm.com/docs/git-rebase).

How I understand it: Every single commit of the history of your current branch, is put on the top of the branch (latest commit) you are rebasing on. In other words, you have every commit in the stack:

&nbsp;

![git_rebase](/images/git_rebase.png)

&nbsp;

{{< admonition type=hint title="Hint">}}
When rebase is finished, make sure to force push.
This is not necessary in merging strategy though.
{{</ admonition >}}

Now the debate: when to use what? Frankly I don't see any huge difference. However, I personally prefer rebasing, since I can track history while if we use merge, we have no idea who did what and how many small other merges might be inside your own merge. *Merge can be super confusing sometimes.*

## git reset

In situations where I know exactly the commit(s) that I screwed up and the last commit that the project was working all right, git reset helps a lot. Let's first see what it does:

{{< admonition type=info title="Name">}}
git-reset - Reset current HEAD to the specified state
{{</ admonition >}}

I think this one doesn't need additional description. Wait? What is the HEAD? HEAD is just the current state. When you pull from remote, the HEAD is already in the same place as develop, when you commit from a new branch, checked out from it, the head shows you exactly where you are. HEAD is the pointer of git (if you want to make a very nerdish comparison). 

So, you added a commit that failed your project to build. Good! Pick up the command line and start resetting:

```bash
git reset --hard HEAD^
```
What this one does, is resets your HEAD 1 commit before. Thus now we basically "revert" the previous commit and your project builds again.
Great. But what the git is `--hard` ? `--hard` means uncommit + unstage + delete the changes. Basically, reset everything from that desired commit. This command is not alone, but I will leave you to explore the rest. A very short and nice explanation can be found [here](https://stackoverflow.com/a/50022436/8914336).

Ok, but what if we screwed 4 commits "ago". We can either do `git reset --hard HEAD~4` or if we are unsure to count backwards, we can always reset to the desired commit just by knowing the commit hash:

```bash
git reset --hard 9ece14fdfdeb44ac95fecd8f8abe4bb639100b9d
```
**Every commit has a hash.**

More information on `git reset` can be found [here](https://git-scm.com/docs/git-reset).

## git cherry pick

Cool. Now I'm going to talk about my favorite git command: cherry pick. How cool it is to pick some cherries now that spring is here. But that has nothing to do with real cherries. Although the concept is pretty similar. Once you are eating a plate of apples and kiwis, you might also want to sweaten your taste, very little. So you pick a cherry. First let's clarify the concept:

&nbsp;

{{< admonition type=info title="Name">}}
git-cherry-pick - Apply the changes introduced by some existing commits
{{</ admonition >}}

Well, to me, the name isn't telling too much. Let's see the description as well. 

{{< admonition type=info title="Description">}}
Given one or more existing commits, apply the change each one introduces, recording a new commit for each. This requires your working tree to be clean (no modifications from the HEAD commit).
{{</ admonition >}}

&nbsp;

Two are the basic scenarios where I use cherry pick:

1 - I need some unmerged-to-main work from some other branch (Let's say that my teammate just created a data class that I also need). 

&nbsp;

![git_cherry_pick](/images/cherry_pick.png)

&nbsp;

{{< admonition type=warning title="Warning">}}
That data class commit we picked must be the only change made in it, otherwise you might get other stuff flying around in your HEAD.
{{</ admonition >}}

2 - When the branch is too old, and rebasing gives you a headache.

Of course that in that scenario, cherry pick is far easier than just struggling and struggling to solve and resolve conflicts quick. Therefore, renaming the old branch, creating a new one on top of `main`/`develop` and cherry-picking commits from the old one is a real life hack.

&nbsp;

When you learn how easy git is, you learn how endless it is. Learning some git tricks would neve hurt and will save you a lot of time. Principal advice: Learn git by command line. This will help you understand every UI tool built for git, and if you switch your company one day, you won't need to learn another git client tool to use it. However, if you are already experienced on git, probably it doesn't make too much sense remembering those commands all the time, but rather use a client for that. Anyways, I always combine it. I still commit, add and push from the command line, but when it comes to making a longer command, rather than remembering the syntax, I just open source tree and let it do the magic for me.

Compliments of [@kofse](https://twitter.com/kofse), who has been so patient in teaching me a lot about git, and [KlausNie](https://github.com/KlausNie) who saves my a** most of the time during git screw ups.
