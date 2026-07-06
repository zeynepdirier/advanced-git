---
title: "Undoing Changes"
teaching: 0
exercises: 0
---

::::::::::::::::::::::::::::::::::::::: objectives

- How do I roll back a single change?
- How do I get back to a specific state?

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How do I undo changes?

::::::::::::::::::::::::::::::::::::::::::::::::::

## Git Revert

Reverting undoes a commit by creating a new commit. This is a safe way to undo changes, as it has no chance of re-writing the commit history. For example, the following command will figure out the changes contained in the 2nd to last commit, create a new commit undoing those changes, and tack the new commit onto the existing project.

Let's make a commit to revert first. On our `bean-dip` branch, let's add a line to our `bean-dip.md` file and commit it.

```bash
nano bean-dip.md
```

```markdown
# Bean Dip
## Ingredients
- beans
## Instructions
- Purchase the bean dip.
```

```bash
git add bean-dip.md
git commit -m "Add bean dip recipe"
```

After making that commit, maybe we second guess and decide we don't want that change after all. We can use `git revert` to back out that last commit.

```bash
git revert HEAD
```

We get a text editor window asking us for a commit message for the revert commit. Save and close the editor to complete the revert.

Let's run `git log --oneline` to see what happened.

```bash
git log --oneline
```

```output
$ git log --oneline
96f26c7 (HEAD -> bean-dip) Revert "Add bean dip recipe"
b8732e4 Add bean dip recipe
79cb366 (origin/bean-dip) Add bean dip recipe.
```

Reverting doesn't actually delete the previous commit - it just creates a new commit that undoes exactly the changes made in that commit.

::: callout

We can also revert to specific commits by providing the commit SHA instead of `HEAD`, or undo several commits by using `HEAD~n` where `n` is the number of commits to go back.

:::

![git revert](fig/08-revert.png){alt="A diagram showing a repository before and after reverting the last commit."}

Note that revert only backs out the atomic changes of the ONE specific commit (by default, you can also give it a range of commits but we are not going to do that here, see the help).

`git revert` does not rewrite history which is why it is the preferred way of dealing with issues when the changes have already been pushed to a remote repository.

## Git Reset

Resetting is a way to move the tip of a branch to a different commit. This can be used to remove commits from the current branch. For example, the following command moves the branch backwards by two commits.

```bash
git reset HEAD~1
```

```output
$ git reset HEAD~1
Unstaged changes after reset:
D       bean-dip.md
```

![git reset](fig/07-reset.png){alt="A diagram showing a repository before and after resetting the last commit."}


The two commits that were on the end of `hotfix` are now dangling, or orphaned commits. This means they will be deleted the next time `git` performs a garbage collection. In other words, you’re saying that you want to throw away these commits.

`git reset` is a simple way to undo changes that haven’t been shared with anyone else. It’s your go-to command when you've started working on a feature and find yourself thinking, “Oh crap, what am I doing? I should just start over.”

In addition to moving the current branch, you can also get `git reset` to alter the staged snapshot and/or the working directory by passing it one of the following flags:

__--soft__ – The staged snapshot and working directory are not altered in any way.

__--mixed__ – The staged snapshot is updated to match the specified commit, but the working directory is not affected. __This is the default option__.

__--hard__ – The staged snapshot and the working directory are both updated to match the specified commit.

It’s easier to think of these modes as defining the scope of a git reset operation.

Let's discard our changes completely using `--hard`:

```bash
git reset HEAD --hard
```

```output
$ git reset HEAD --hard
HEAD is now at bb3edf6 Add bean dip recipe
```

::: callout

`git reset` can also work on a single file:

```bash
git reset HEAD~2 foo.txt
```

:::

## Git Checkout: A Gentle Way

We already saw that `git checkout` is used to move to a different branch but can also be used to update the state of the repository to a specific point in the projects history.

```bash
git checkout main
git checkout HEAD~2
```

```output
$ git checkout HEAD~2
Note: switching to 'HEAD~2'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at c6ae196 Modify guacamole instructions to include food processor.
```

![git checkout](fig/09-checkout.png){alt="A diagram showing a repository before and after using git checkout to move to a previous commit."}

This puts you in a detached HEAD state. AGHRRR!

Most of the time, HEAD points to a branch name. When you add a new commit, your branch reference is updated to point to it, but HEAD remains the same. When you change branches, HEAD is updated to point to the branch you’ve switched to. All of that means that, in these scenarios, HEAD is synonymous with “the last commit in the current branch.” This is the normal state, in which HEAD is attached to a branch.

The detached HEAD state is when HEAD is pointing directly to a commit instead of a branch. This is really useful because it allows you to go to a previous point in the project’s history. You can also make changes here and see how they affect the project.

```bash
echo "Welcome to the alternate timeline, Marty!" > new-file.txt
git add .
git commit -m "Create new file"
echo "Another line" >> new-file.txt
git commit -a -m "Add a new line to the file"
git log --oneline
```

```output
$ git log --oneline
81c079c (HEAD) Add a new line to the file
1881c24 Create new file
c6ae196 Modify guacamole instructions to include food processor.
...
```

To save this alternate history, create a new branch:

```bash
git branch alt-history
git checkout alt-history
```

If you haven't made any changes or you have made changes but you want to discard them you can recover by switching back to your branch:

```bash
git checkout hotfix
```

https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting
Also OMG: http://blog.kfish.org/2010/04/git-lola.html


:::::::::::::::::::::::::::::::::::::::  challenge

## Squashing Commits and Amending

You are working on a pancake recipe and adding ingredients one by one, each as a separate commit.
You can track your history with `git log --oneline` at each step.

1. Create `pancake.md` and add the following ingredients as separate commits:
   - flour -> commit message: `Add flour`
   - milk -> commit message: `Add milk`
   - egg -> commit message: `Add eg` (this typo is intentional)
   - Fix the typo in the commit message before moving on.

::: hint

To correct only the commit message without touching the files, check `git commit --amend --help`.
:::

2. Squash all three commits into one.

::: hint
To squash multiple commits, look into `git rebase -i`.
:::

3. Oh, you forgot to butter. Add it to `pancake.md` and amend the existing commit without changing the commit message.

:::::::::::::::  solution

**Step 1:**

```bash
echo "flour" > pancake.md
git add pancake.md
git commit -m "Add flour"

echo "milk" >> pancake.md
git add pancake.md
git commit -m "Add milk"

echo "egg" >> pancake.md
git add pancake.md
git commit -m "Add eg"

git log --oneline
```
```output
<hash> (HEAD -> main) Add eg
<hash> Add milk
<hash> Add flour
```

```bash
git commit --amend -m "Add egg"

git log --oneline
```
```output
<hash> (HEAD -> main) Add egg
<hash> Add milk
<hash> Add flour
```

**Step 2:**

```bash
git rebase -i HEAD~3
```

In the editor, change `pick` to `s` for the last two commits:

```bash
pick <hash> Add flour
s <hash> Add milk
s <hash> Add egg
```

Save and exit. In the next editor, write a single commit message:
`Add ingredients to pancake recipe`

```bash
git log --oneline
```
```output
<hash> (HEAD -> main) Add ingredients to pancake recipe
```

**Step 3:**

```bash
echo "butter" >> pancake.md
git add pancake.md
git commit --amend --no-edit

git log --oneline
```
```output
<hash> (HEAD -> main) Add ingredients to pancake recipe
```
:::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::::::::::::::::

## Exercise: Undoing Changes

:::::::::::::::::::::::::::::::::::::::  challenge

## Exercise: Undoing Changes

1. Create a new branch called `soup-recipes`. Create a new file `soups/tomato-soup.md` and make 3-4 commits adding ingredients and instructions. Check the log to see the SHA of the last commit.
2. You realize the last instruction was wrong. Revert the last commit. Check the history.
3. The recipe isn't working out. Completely throw away the last two commits [DANGER ZONE!!!]. Check the status and the log.
4. Undo another commit but leave the changes in the staging area so you can review them. Check the status and log.
5. After reviewing, you decide to keep the changes. Add and commit them with a better commit message.

:::::::::::::::  solution

Step 1:

```bash
git checkout -b soup-recipes
mkdir -p soups
nano soups/tomato-soup.md
```

Create a file over 4 commits:
```markdown
# Tomato Soup
## Ingredients
- tomatoes (6)
## Instructions
- Chop tomatoes
- Add water and boil
```

```bash
git add soups/tomato-soup.md
git commit -m "Finish instructions"
git status
git log --oneline
```

Step 2:

```bash
git revert HEAD
git log
```

Step 3:

```bash
git reset HEAD~2 --hard
git status
git log
```

Step 4:

```bash
git reset HEAD~1
git status
git log
```

Step 5:

```bash
git add soups/tomato-soup.md
git commit -m "Add tomato soup with basic ingredients"
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

<!--- ![GitFlow 1](../fig/43-undo.png) --->

:::::::::::::::::::::::::::::::::::::::: keypoints

- `git reset` rolls back the commits and leaves the changes in the files
- `git reset --hard` roll back and delete all changes
- `git reset` does alter the history of the project.
- You should use `git reset` to undo local changes that have not been pushed to a remote repository.
- `git revert` undoes a commit by creating a new commit.
- `git revert` should be used to undo changes on a public branch or changes that have already been pushed remotely.
- `git revert` only backs out a single commit or a range of commits.

::::::::::::::::::::::::::::::::::::::::::::::::::
