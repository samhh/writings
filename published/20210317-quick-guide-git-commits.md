---
slug: "quick-guide-git-commits"
date: "2021-03-17"
title: "A quick guide to Git commits"
---

Git has achieved a remarkable ubiquity amongst software engineers. We use it and version control more broadly to track changes over time. It's surprisingly useful as a historical documentative tool, perhaps more than something like Confluence could ever be. It heavily aids us in gatekeeping merges in codebases where we care about quality. It allows us to programmatically determine when a bug was introduced. With this in mind, we owe it to ourselves, our colleagues, and those who'll interact with our work after us to manicure our commits and leave behind a digital paper trail that makes semantic sense.

I said this was a quick guide in the title and it will be. I shan't go over the stuff that's very broadly known such as idiomatic grammatical tensing. I'll very quickly list the goals of good commit messages before proceeding:

- Commits should be "atomic". Where possible every commit should serve a single purpose. Builds, tests, et cetera should pass for any given commit.
- Commits should tell a story. Viewed in order, in for example a pull request, it should be possible to follow along with the intentions of the author.
- Commits should provide context. A dubious change should have the reasoning for said change documented within the commit message.

Everything I'm going to discuss is on the command-line interface (CLI). Presumably this is all achievable in any competent GUI if you're so inclined.

## Workflow

This is the part of the post that's particularly opinionated and whence the following suggestions are derived, namely my personal workflow. It is essentially as follows:

1. Pick up a ticket.
2. Do work on the ticket, potentially (likely) committing as you go.
3. Once it's ready, clean up everything.
4. Push it and make a PR or equivalent.

The third step is I believe the key that's missing from many peoples' workflows. Git is very flexible and modifying your commits is actually really easy, but it's not at all obvious how to do it. Assuming you're willing to follow this pattern I believe you'll find value in the rest of this post.

As an aside, it's absolutely not necessary, but if you'd like to gain a stronger intuition for what Git's doing under the hood with all its arcane commands I'd recommend watching [Git for ages 4 and up](https://www.youtube.com/watch?v=1ffBJ4sVUb4).

## Handy syntax

### Where's my HEAD gone?

Git always has access to an identifier that indicates "where we are" in the commit tree, namely `HEAD`. Note that the casing matters on case-sensitive operating systems.

You'll see it referenced if you run `git log`. It follows you around like a helpful stalker.

This can be handy when you want to dynamically reference where you are.

There are all sorts of other "references" you can use like this (such as commit IDs, branch names, etc), so keep in mind this flexibility whenever you see `<ref>` in a command.

### Parents

You can reference the parents of commits with the tilde (`~`), for example this references the commit two behind `<ref>`:

```
<ref>~2
```

### Ranges

Git supports referencing ranges of commits in all sorts of places with the following syntax (without the angle brackets):

```
<ref>..<ref>
```

## Viewing the log

We've all used `git log` for a visualisation of the commit tree, but there are some small tricks that can be really handy.

First and foremost, goodness me the default log output is noisy, no wonder everyone uses the web UI to look at it. Do yourself a favour and alias it to something saner in `~/.config/git/config`. My current alias is essentially as follows and can be viewed [here](https://github.com/samhh/dotfiles/blob/desktop-linux/home/.config/git/config):

```
$ git log --graph --pretty=format:'%Cblue%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold red)[%an]%Creset' --abbrev-commit --date=relative
```

Or for a shorter, simpler, less informative version:

```
$ git log --oneline
```

Lovely. Now that we have the gift of sight, let's learn how to work with this a bit. After all, how can we clean up our commits if we can't see in a pinch what they are?

Assuming we've branched off of `master`, we can run the following for a log of everything since `master` (excluding the commit `master` points to):

```
$ git log master..HEAD
```

We can also quickly view the last _n_ commits as follows, for example to view the last three:

```
$ git log -3
```

## Viewing a commit

How can we review our commits if we can't look at their contents? The (short) messages may be excellently written but they're worth little if the code they represent is in a poor state. We can view any commit as follows:

```
$ git show <ref>
```

As with log, we can also look at all the changes we've made using a range:

```
$ git show master..HEAD
```

## Simple renaming

Sometimes our commits make sense in terms of code but the messages themselves could do with some cleanup. In these cases we can make use of interactive rebasing. Let's rename some commits we've made since master:

```
$ git rebase -i master
```

This will bring up a temporary file in your $EDITOR. The commits are listed in the following format: <command> <ref> <name>

As you can see in the large comment block at the bottom, there are a lot of things you can do with interactive rebase, for example dropping and editing commits. For now let's make use of the "reword" command.

Replace the "pick" in one or many of the listed commits with "r" or "reword" and save and exit the file. For each commit for which you specified that you wanted to reword, a new temporary file will be opened with the current message for that commit. Simply edit the commit message as desired and save and exit the file when ready.

## Removing commits without losing changes

Sometimes when developing we'll want to undo all of our development commits without losing the changes themselves. This is trivially easy with the reset command:

```
$ git reset <ref>
```

For example, to reset everything since master and to reset the last commit respectively:

```
$ git reset master
```

```
$ git reset HEAD~1
```

All of the changes from these commits will now be sitting in our working tree as if they'd never been commit in the first place.

One word of warning - beware the `--hard` flag! This flag means the changes of said commits will be removed as well.

## Partial adding

It's often the case that we make lots of changes to a codebase but only want to partially commit them. This is easy enough when the changes can be cleanly separated by file, but what about semantically distinct changes within the same file? Enter partial adding, or as `man git add` puts it: "Interactively choose hunks of patch between the index and the work tree and add them to the index."

This one's really easy and very valuable; simply add a `-p`/`--patch` flag when calling `git add`. You'll be interactively prompted to review your code in "hunks". In this interactive session, for any given hunk, you've a few commands that you can run; all available commands can be expanded by replying `?`. Subjectively speaking the most useful commands are as follows:

- `y` - stage this hunk
- `n` - do not stage this hunk
- `s` - split this hunk
- `e` - edit this hunk more granularly than `s` allows

## Amending the previous commit

Sometimes you're happy with what you've done _except_ for the previous commit. In this case there's a more lightweight solution than interactive rebasing which is as follows.

To add the staged changes to the commit:

```
$ git commit --amend --no-edit
```

To add the staged changes to the commit (if any) and change the commit message:

```
$ git commit --amend
```

## Providing context

As described above we want to provide context and more information in a lot of commits. This doesn't belong in the initial short/subject message however which should remain concise. Instead, additional information - the sort of stuff you might typically (also) put into a ticket, on a wiki page, in a PR description - can exist alongside the relevant code changes in perpetuity as additional paragraphs in the commit message.

One way to do this is to run `git commit` without a subject argument which will bring up your $EDITOR. Simply put your subject message on the top line, and then newline separate your paragraphs like so:

```
Do a change

This change is important because of reasons.

It's implemented in terms of something because of reasons.
```

Another way, one much might more easily fit into how you commit code now, is to append additional message flags like so:

```
$ git commit 'Do a change' -m 'This change is important because of reasons.' -m 'It's implemented in terms of something because of reasons.'
```

As you can see, you can add a new `-m` flag for each paragraph of additional information you'd like to include.

