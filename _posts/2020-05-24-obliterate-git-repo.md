---
title: "How to reset the git remote to the initial commit in one line"
layout: single
published: true
---

The more I learn about git, the more I appreciate how insanely powerful the tool is. This is both a blessing and a curse, and the ways in which git will let you or your team mates shoot themselves in the foot are both a bit scary (considering that your valuable commits are at stake here) and childishly exciting.

Inevitably, one day the question ocurred to me, how can one obliterate a repo both locally and on the remote with a one-liner?

**Never, ever run these commands on a git repo**. You've been warned.

**TL;DR**
```bash
git rev-list --max-parents=0 HEAD | tail -1 | xargs git reset --hard && git push -f
```

<figure>
	<img src="/assets/images/daniel-tausis-loeqHoa1uWY-unsplash.jpg" width="100%">
	<figurecaption>This will be your repo, should you ever try to run the commands in this post.<br/> <i>Photo by <a href="https://unsplash.com/photos/loeqHoa1uWY">Daniel Tausis</a> on <a href="https://unsplash.com">Unsplash</a></i></figurecaption>
</figure>



Now, as you probably know, git comes with a vast set of configurable parameters and command line options to help adapt the tool to your workflow, so there's a multitude of ways to intentionally ruin a shared or private repo. There are also various ways to save a repo after disaster has struck, but that's not always the case. Bottom line is, if you're playing with git fire, you're gonna git burned.

So, without further ado, how can you use your git powers to satisfy your childish desire for destruction?

```bash
git rev-list --max-parents=0 HEAD | tail -1 | xargs git reset --hard && git push -f
```

That's _one_ receipe for disaster, and perhaps also a sudden need to find a new job. The command consists of four parts, and their purpose is as follows:

1. Find the root commit(s),
2. In case there are more than one, choose the oldest one,
2. Perform a hard reset to reset both working tree and index to the root commit,
3. Overwrite the remote's history with your modified, local history.

## Breaking down the details of the commands
Let's break them down, one by one.

```bash
git rev-list --max-parents=0 HEAD
```

This command will list commits that are reachable from `HEAD` and with 0 parents in reverse chronological order.

```bash
tail -1
```

This will simply grab the _tail_ of what's input to it. In this case, it will drop everything but the last line. I'm yet to experience a real-world situation where multiple root commits are reachable from `HEAD`, but it's quite simple to achieve this for the sake of demonstration. All you have to do is merge an orphan branch with the `--allow-unrelated-histories`.

```bash
xargs git reset --hard
```

First of all, git won't read piped input, so we need to use `xargs` to grab the piped input (i.e. commit hash) and make it appear to git as a command line argument. The actual git part simply resets the working tree and index to the commit from the previous command.

```bash
git push -f
```

The often misused act of force pushing. This will forcefully replace the remote's history with your local history.


# A final remark
As implied by the title of this post, these commands will severly screw up both the local and remote copy of your repo, and it will do so forcefully. If you're lucky, you can recover a previous state of your repo by inspecting the `reflog`. However, whether it will be possible to recover the repo or not depends on your git/reflog settings. It also requires you to not have manually deleted the commits. Furtermore, if the repo is shared and someone had pushed commits to it between your last pull and your force push, you'll not be able to restore those commits locally (since they never made it into your local repo).

Lastly, I'll say it again: **Never, ever do this to a repo**.

