# Lightning Talk "Improve your git workflow"


# Agenda

*Goal:* a easy to understand history in our git repositories

- How to commit to make everyone's life easier
- Hints for a better git setup
- Tool suggestions that work anywhere
- How to amend and rebase commits

---

# Motivation

Put yourself into your future self's shoes: Will the message make me understand the changes done?

&nbsp;

## Recent (bad) examples of commit messages in our repository:

- styling
- debug
- default None
- fixes
- add johns suggest
- Apply 1 suggestion(s) to 1 file(s)

&nbsp;

*=> More effort should go into writing rather than understanding the history.*

---

# When and how to "slice" code changes into commits?


## While progressing towards your goal:

- logically separate set of changes
- digestible size for a reviewer
- *commit early, commit often, commit logically separate*

*Hint:* Stage small chunks with *git add -p* or even single lines with *tig*

&nbsp;

## Before you open a merge request:

- squash (=merge multiple commits into one):
    - intermediate steps
    - failed attempts
    - progressions to final solution
- re-arrange commits in a logical order
- drop debug or temporary code

---

# The official commit message rules

- 1st line: subject (max. 50 chars)
- 2nd line: left blank for readability
- remaining lines: message body


&nbsp;
## Example commit message:

```
> commit 0ef8002be8198a509c70f8dd98f7bd54a6e408fa
> Date:   Tue Dec 22 13:51:25 2020 +0100
>
> Disable strict ssl-certificate-checking for npm
>
> We changed from http to https in
> https://gitlab.foo.com/some-team/bar/merge_requests/5714/
> for the npm repository. It seems though as if the certificate for the
> repo server is not present in the docker image and it fails with a
> untrusted server error, see:
> https://gitlab.foo.com/some-team/bar/-/jobs/3421479
>
> This is a quick workaround, better will probably be to have the pem file
> inside the docker container.
```

---

## Rules for the subject line

- Max 50 characters
- Capitalize only the first letter
- Don't put a period at the end
- Use the imperative mood

&nbsp;

Imperative mood means, this:

- *Fix* the ...

Instead of:

- *Fixed* the ...
- *Fixing* the ...

---

# How to write the subject line

> A good git commit message can be appended to the statement:
> "If applied, this commit will …."

&nbsp;

Example first words:

Add - Remove - Fix - Bump - Make - Start - Stop - Refactor - Reformat - Optimize - Document

&nbsp;

*Hint:* You can use a git commit message template to remind yourself of the format:

````
> # If applied, this commit will...
> Add unit tests for ...
````

## Setup in ~/.gitconfig:
```
> [commit]
>     template = /home/martin/dot/git/commit_template
```

---

# Rules for the body

- Wrap lines at 72 characters
- Describe what was done and why
- Add relevant info
    - urls (CI jobs, fogbugz, jira, sentry, ...)
    - sources
    - ticket trackers
    - debugging info
    - side effects / trade-offs
    - temporary code/command line magic used for bulk changes
    - failed attempts
    - describe what caused you a lot of time/frustration

---

# Tool suggestions

## tig: terminal git interface

https://github.com/jonas/tig

Available for linux/mac/windows

- go through commit history (messages, diff, stashs) using the arrow keys
- takes commands & arguments like git too: tig stash / tig status / tig blame some/file.py

*Useful keys:*

- t: browse repository tree at any given commit
- s -> 1: (un)stage line-by-line
- \[\]: change diff context size


&nbsp;
### Setup in ~/.tigrc:
```
> set main-view = line-number:yes,interval=1 id:yes date:default author:full commit-title:yes,graph,refs,overflow=no
> set stage-view = line-number:yes,interval=1 text
> set diff-view = line-number:yes,interval=1 text:yes,commit-title-overflow=no
```
*Hint:* Having the hash and line numbers shown is good info for git rebase commands.

---

## vim as commit message editor

- works anywhere
- has spell checking
- reminds you of the format rules

Useful commands:

- *:cq!* - Exit with an error code; ensures the git operation is canceled
- *gq* or *gqq* - Reflow line or selected lines to respect the maximum length


### Config extracts:


~/.vimrc:

```
> set colorcolumn=50
> set textwidth=72
> set spell
> set spelllang=en_us
>
> nnoremap <leader>f 1z=
> hi SpellBad cterm=underline ctermfg=red ctermbg=239
```

~/.gitconfig:

```
> [core]
>     editor = nvim -u /home/martin/dot/nvim/init.vim.git
> [merge]
>     tool = vimdiff3
>     conflictstyle = merge
> [mergetool "vimdiff3"]
>     cmd = nvim -f -d \"$LOCAL\" \"$REMOTE\" \"$MERGED\"
```

---

## git-interactive-rebase-tool

https://github.com/MitMaro/git-interactive-rebase-tool


- Cross-platform terminal editor for interactive rebase.
- Reorder, squash, drop commits intuitively

Will automatically start when you issue a rebase command, e.g.:

> git rebase -i HEAD~3

&nbsp;


### Config extracts:

~/.gitconfig:

```
> [sequence]
>     editor = interactive-rebase-tool
```

---

# Rebase and amend - usage examples

## Making sure your branch sits "on top" of another

Most often used to keep up-to-date with master but also helps while co-working on a feature:


> git rebase some-branch-name


- *Changes branch you are on*, not anything in the other branch
- Puts any new commits of the other branch underneath your branch/commits
- Avoids merge commits
- Ensures your merge request will not have any conflicts with the other

*Advantage:* Small merge conflict context per commit vs. one big one when using 'git merge'

&nbsp;

*Hint*:

alias rbm='BRANCH=\`git rev-parse --abbrev-ref HEAD\`; gco master && git pull && gco $BRANCH && git rebase master'

- Check out master
- Pull changes
- Go back to previous branch
- Rebase on master

=> You are up to date

----

## Short side track: git reset

There are 3 different ways to reset the state of your repository. In simple words:


- *--mixed*: is the default; the commits are gone, the file changes remain and are unstaged
- *--hard*: the file changes and the commits are completely gone; but can also be used to move to a repo state where more commits existed
- *--soft*: like --mixed but does not unstage it (rarely useful)

----

## Changing the last commit:


> git commit --amend

Will add whatever is staged to the last commit and allow you to re-edit the commit message.

&nbsp;

## Only content:
> git commit --amend --no-edit

Same as above, but message remains untouched.

&nbsp;

*Note:* Both change the commit id. Don't do it if someone else is depending on the original commits.

---

## The interactive rebase

Allows to rearrange, squash, split, drop or edit commits.

- You can always only rebase *a range of commits* starting from the most recent
- But you don't need to actually change more than one

Generally:

> git rebase -i <the *parent* of the last commit you want to edit>

Other examples:

````
> git rebase -i @~3           # rebase the last 3 commits
> git rebase -i --root        # Rebase up to first commit
> git rebase -i master        # rebase all commits since branching off
> git log master.. --reverse  # log of the commits since branching off
````

*Be aware*:

- Every commit in the range will get a new id
- Commit authors may no longer be the true author when squashing commits of more than one person


*Hints:*

- multiple small rebases are easier than one big one
- *@* is short for HEAD
- *~* and *^* refer to the parent of a commits, e.g. git show 8f327*~*
- only with *~* you can specify a range of commits

---

## Rebase to reorder, squash/fixup

### Goal

- Bringing relevant commits together
- Merging intermediate results into one

*Note:*

- in "git log / tig" the newest commit is on *top*
- in the rebase editor the newest commit is on the *bottom*
- if you mark a commit with squash or fixup it will be merged into the commit *above* it

You can use --reverse to have the same order with a log/tig command:

> git log --pretty="format:%h %s" --reverse

&nbsp;

### Process

- Make sure your repo is "clean", meaning nothing staged and no changes
- *Always make a copy* of your branch before you start rebasing! (for comparison & backup)

```
> gco -b example1-rebased
> git log --pretty="format:%h %s"  # state before
> git rebase -i --root  # Rebase up to first commit
> # Order the commits using the j/k keys, mark to fixup/squash with f/s keys
> # Use W key to save and close
> git log --pretty="format:%h %s"  # state after
```

Confirm the result with e.g.:

 - git diff example1  # should give no output
 - git log -p
 - tig

----

## Rebase to split or edit

The process is similar. You can keep the commit's changes in one commit or split them up in a desired bigger number.

*Note:* When the rebase gets to the “edit” commit it will stop *after* the commit was applied. Not before as one might
expect.

### Editing:

```
> # - Start a rebase, mark one commit as "edit", save & exit rebase-editor
> # - Make changes and stage them
> git commit --amend
> git rebase --continue
```

### Splitting:

```
> git reset HEAD^
> # stage the part you want to have in the first commit
> git commit -c <original hash>  # That way you can use the original message as template
> # repeat staging & commiting
> git rebase --continue
```

*Advice:* For commits where you add a new file, git does not allow partially staging it.

Read further: https://www.bryanbraun.com/2019/02/23/editing-a-commit-in-an-interactive-rebase/

----

## If a rebase goes wrong: git's undo list

````
> git reflog
> 0a0322c HEAD@{3}: commit: Use existing user record in SuperuserActionFormTestCase
> dfa642f HEAD@{4}: rebase -i (finish): returning to refs/heads/mb-foo
> e9bd32b HEAD@{7}: rebase -i (pick): Add testcase for SuperuserActionForm
> f0ba7eb HEAD@{8}: rebase -i (fixup): Remove storing object into class instance in clean
> 449f00e HEAD@{9}: rebase -i (start): checkout HEAD~5
> d170352 HEAD@{10}: commit: wip
````

Find commit to which you want to go back

````
> git rebase --abort
> git reset --hard d170352
````

---

## If a rebase is too hard: 'divide and conquer' of the final state may be an alternative

- facing too many conflicts?
- commits too big or too intermingled?

But you still want to have a cleaner history, then try this:

````
gco -b my-branch-neat
git reset $(git log master.. --oneline --pretty="format:%h"| tail -1)~
````

What does it do?

- take id of first commit of branch
- reset tree to parent of that commit
- unstage all changes


This is like if you had never made a commit on your branch but all changes are there. Now you can divide your final code state into
logical parts by staging and commiting newly. Any intermediate code states are however lost.

*Hint:*

Leave tig running in a separate terminal before you reset. That way you can still browse the commits and the
original messages for orientation.

----

# Commands used for examples

Get the repository:

git clone git@github.com:belugame/improve-git-workflow.git

## Example 1: Changing a commit

````
gco -b example1
sed -i 's/greet/salute/g' person.py
git add .
git commit --amend
````

## Example 2: Reordering commits

````
gco master
gco -b example2
gri --root
(sort A-B-C with j/k, confirm with  "W")
````

----

## Example 3: Squash/Fixup

````
gco master
gco -b example3
tig  #  to get right number
gri @~9
(squash/fixup A1-A2, use "c" and then "d" to see the commits diff, back with "q")
git diff example1  # to confirm no changes
````

## Example 4: Edit a commit

````
gco master
gco -b example4
git show
gri @-3
git show
sed -i 's/Jim/John/g' tests.py
git commit --amend --no-edit
git rebase --continue
````

----

## Example 5: Split a commit into multiple

````
gco -b example5
gri @~5
(mark C2 for "edit" with "e", "W")
git reset HEAD^
(tig stage single line)
git commit         # C2a Add locked
git add .
git commit         # C2b Add unlock
git rebase --continue
````

## Example 6: Conflict while rebasing

````
gco -b example6
gri @~4
(move B5 before B3 with "j", "W")
git status
vi person.py (fix typo in "print", remove "return")
git add .
git rebase --continue
vi person.py (remove "print", use "return")
git add .
git rebase --continue
````

----


# Aliases used

````
gri='git rebase -i '
gco='git checkout'
rbm='BRANCH=\`git rev-parse --abbrev-ref HEAD\`; gco master && git pull && gco $BRANCH && git rebase master'
````

----
