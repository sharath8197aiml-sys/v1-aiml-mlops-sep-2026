# Git Remote-to-Local Synchronization Guide

## Your repository setup

- **Local repository (Windows):** `D:\MLOPS_Shivam_prasad\MLOPS_Github_Code\v1-aiml-mlops-sep-2026`
- **Your GitHub fork (`origin`):** `https://github.com/sharath8197aiml-sys/v1-aiml-mlops-sep-2026`
- **Instructor's original repository (`upstream`):** `https://github.com/shivam13juna/v1-aiml-mlops-sep-2026`
- **Working branch:** `master`

There are **two different synchronization goals**. Choose the scenario that matches what you want to update.

## Scenario 1 — Download changes from your GitHub fork to your laptop

Use this when your fork on GitHub contains commits you want locally.

### Step 1: Open PowerShell in your repository

```powershell
cd D:\MLOPS_Shivam_prasad\MLOPS_Github_Code\v1-aiml-mlops-sep-2026
```

### Step 2: Inspect your branch and working tree

```powershell
git status
git branch --show-current
git remote -v
```

Confirm the working tree is clean, your branch is `master`, and `origin` points to your fork. If you have uncommitted changes, commit or stash them first; do not overwrite them.

### Step 3: Pull from your fork

```powershell
git switch master
git pull --ff-only origin master
```

`git pull` fetches remote commits and integrates them. `--ff-only` only updates your local branch when no merge commit is required. If your local and remote branches diverged, Git stops instead of silently creating a merge.

### Step 4: Verify

```powershell
git status
git log -1 --oneline
```

A clean working tree and an up-to-date tracking branch indicate successful synchronization at the time of the pull.

## Scenario 2 — Download instructor updates and publish them to your fork

Use this when `upstream` has new commits you want in both your laptop and your own GitHub fork.

### Step 1: Inspect local changes

```powershell
git status
```

**Do not start a merge with uncommitted work.** Commit your changes or stash them safely first.

### Step 2: Switch to master and update it from your fork

```powershell
git switch master
git pull --ff-only origin master
```

If this command refuses to fast-forward, inspect the divergence before proceeding.

### Step 3: Create an optional safety branch

```powershell
git branch backup-master-before-sync
```

If that name already exists from a previous synchronization, use a new descriptive name (for example, `backup-master-2026-10-16`). This branch preserves a reference to your current commit.

### Step 4: Fetch the instructor's updates

```powershell
git fetch upstream
```

This downloads commits and updates `upstream/master`; **it does not change your working files**.

If the `upstream` remote is missing, add it **once**:

```powershell
git remote add upstream https://github.com/shivam13juna/v1-aiml-mlops-sep-2026.git
```

Use `git remote -v` to verify. Do not repeat `git remote add upstream` after it exists.

### Step 5: Compare commits before merging

```powershell
# Commits the instructor has that your master does not
git log --oneline master..upstream/master

# Commits your master has that the instructor does not
git log --oneline upstream/master..master
```

For example, if the first command shows three commits, your local branch has not yet incorporated those three upstream commits. If the second shows eight commits, those eight are present locally but not in upstream. This is the basis of GitHub's **ahead/behind** message.

### Step 6: Merge instructor changes into your local master

```powershell
git merge upstream/master
```

- **No conflicts:** Git completes the merge (or fast-forwards). Continue to verification.
- **Conflicts:** Git pauses the merge. Review and resolve each file before committing.

### Step 7: Resolve conflicts, if any

Check the affected files:

```powershell
git status
```

For a normal content conflict, use VS Code's merge editor to select or combine the correct changes, save the file, then stage **only the files you resolved**:

```powershell
git add path/to/resolved-file
```

For the **modify/delete** conflict you encountered earlier, upstream deleted `.github/workflows/continuous_integration.yaml` while your branch modified it. Decide deliberately:

**Keep your modified workflow file:**

```powershell
git add .github/workflows/continuous_integration.yaml
```

**Accept upstream's deletion instead:**

```powershell
git rm .github/workflows/continuous_integration.yaml
```

Then inspect the status and finish the merge:

```powershell
git status
git commit
```

**Caution:** Running `git add .` without reviewing a modify/delete conflict can silently choose to retain a file upstream intended to delete. Also inspect `.github/workflows/` after merging; keeping both old and new GitHub Actions workflows may trigger duplicate CI/CD runs.

To abandon an unresolved merge and return to the pre-merge state:

```powershell
git merge --abort
```

Only use `--abort` while a merge is in progress; it does not undo a completed merge commit.

### Step 8: Verify the integrated code

```powershell
git status
git log -1 --oneline
git log --oneline master..upstream/master
Get-ChildItem .github\workflows
```

After a successful merge, the third command normally prints nothing because `master` contains the fetched upstream commits. Review your workflows and run the project's relevant tests (for example, `python -m pytest`) before pushing.

### Step 9: Push the merged changes to your GitHub fork

```powershell
git push origin master
```

This publishes your local `master` (including the upstream merge) to **your fork**, not the instructor's repository.

### Step 10: Verify final status

```powershell
git status
git remote -v
```

## Quick reference

| Goal | Command |
|---|---|
| Inspect local changes | `git status` |
| Confirm current branch | `git branch --show-current` |
| Check remote destinations | `git remote -v` |
| Your fork → laptop | `git pull --ff-only origin master` |
| Download original repo updates | `git fetch upstream` |
| View incoming instructor commits | `git log --oneline master..upstream/master` |
| Integrate instructor commits | `git merge upstream/master` |
| Laptop → your fork | `git push origin master` |
| Inspect unresolved conflicts | `git status` |
| Cancel an in-progress merge | `git merge --abort` |

## Copy-and-paste routine (when working tree is clean)

```powershell
cd D:\MLOPS_Shivam_prasad\MLOPS_Github_Code\v1-aiml-mlops-sep-2026

git status
git switch master
git pull --ff-only origin master

git fetch upstream
git log --oneline master..upstream/master
git merge upstream/master

# STOP HERE if Git reports merge conflicts.
# Resolve conflicts, stage resolutions, and commit the merge first.

git status
git push origin master
git status
```

### Important distinctions

- **`git fetch`** downloads remote history without merging it into your working branch.
- **`git pull --ff-only`** fetches and updates your branch only when a fast-forward is possible.
- **`git merge`** incorporates commits from another branch and may require conflict resolution.
- **`git push`** publishes your local commits to a remote branch.
- **`origin`** is your fork; **`upstream`** is the instructor's original repository.

### Common problems

**`error: remote upstream already exists.`** You already configured it. Check `git remote -v` instead of adding it again.

**`fatal: Not possible to fast-forward, aborting.`** Your local and remote branches have diverged. Inspect `git status` and the commit differences before deciding how to integrate them.

**`You have unmerged paths.`** A merge is in progress. Resolve the listed files, stage them, and commit before pushing.

**`Everything up-to-date` on push.** Your local branch has no commits that need uploading to that remote branch.

**GitHub says your fork is ahead of upstream.** This is normal if you have your own commits. Synchronization does not require your fork and upstream to have identical histories.
