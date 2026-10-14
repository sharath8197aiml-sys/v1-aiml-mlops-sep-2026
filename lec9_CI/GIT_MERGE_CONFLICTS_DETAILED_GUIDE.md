# Git Fork Synchronization and Merge Conflicts — Detailed Notes

> **Project:** `v1-aiml-mlops-sep-2026`  
> **Your fork (`origin`):** `https://github.com/sharath8197aiml-sys/v1-aiml-mlops-sep-2026`  
> **Original repository (`upstream`):** `https://github.com/shivam13juna/v1-aiml-mlops-sep-2026.git`  
> **Working branch:** `master`  
> **Environment:** Windows PowerShell and VS Code

## 1. The GitHub message you saw

GitHub displayed:

> This branch is **8 commits ahead of and 3 commits behind** `shivam13juna/v1-aiml-mlops-sep-2026:master`.

It also warned that **this branch has conflicts that must be resolved** and offered options including **Discard 8 commits** and **Open pull request**.

### Meaning

| Message | Interpretation |
|---|---|
| **8 commits ahead** | Your fork's `master` contains 8 commits not in the original repository's `master`. |
| **3 commits behind** | The original repository has 3 commits your fork's `master` does not contain. |
| **Conflicts** | GitHub cannot complete its proposed synchronization automatically; overlapping edits or other merge conflicts require a decision. |
| **Discard 8 commits** | A destructive option that would remove your fork's 8 unique commits from the branch to match upstream. Avoid this if you want your work. |
| **Open pull request** | Starts a pull-request-based process; conflicts may still require manual resolution. It is not an automatic fix. |

**Commits are not files.** One commit can change multiple files; multiple commits can modify the same file.

### Example history (illustrative)

```text
                  A---B---C   upstream/master (3 new commits)
                 /
common commits ---
                 \
                  D---E---F---G---H---I---J---K   origin/master (8 new commits)
```

The exact commit names and graph above are illustrative, not your repository's actual history.

## 2. Your actual local Git state

You ran:

```powershell
git status
git branch --show-current
git remote -v
```

And got:

```text
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

```text
master
```

```text
origin  https://github.com/sharath8197aiml-sys/v1-aiml-mlops-sep-2026 (fetch)
origin  https://github.com/sharath8197aiml-sys/v1-aiml-mlops-sep-2026 (push)
```

**What this confirms:**

- You were on local branch `master`.
- No uncommitted local changes were present.
- Your local branch was up to date with its locally recorded `origin/master` tracking branch. (To verify against GitHub's current state, fetch first.)
- `origin` correctly pointed to **your fork**.
- No `upstream` remote was shown yet.

**What it does not confirm:** That your fork is synchronized with the original repository. `origin` and `upstream` are different remotes.

## 3. Origin vs upstream vs local branch

```text
Original repository (upstream)
shivam13juna/v1-aiml-mlops-sep-2026
             |
             | git fetch upstream
             v
Your laptop: local master
             |
             | git push origin master
             v
Your GitHub fork (origin)
sharath8197aiml-sys/v1-aiml-mlops-sep-2026
```

| Term | Purpose |
|---|---|
| `origin` | Remote pointing to your fork, usually where you push. |
| `upstream` | Remote pointing to the original instructor/source repository. |
| `master` | Your current local branch. |
| `origin/master` | Locally stored remote-tracking reference for your fork's master branch, updated by fetch. |
| `upstream/master` | Locally stored remote-tracking reference for the original repository's master branch, updated by fetch. |

`git fetch` downloads commits and updates remote-tracking references **without merging them into your working branch**.

`git merge upstream/master` attempts to incorporate those fetched commits into your local `master`.

`git push origin master` publishes your local `master` to your fork.

## 4. What is a merge conflict?

A merge conflict happens when Git cannot automatically reconcile competing changes. Common cases include:

- Both branches modify the same lines in a file.
- One branch deletes a file that the other modifies.
- Both branches add a file at the same path with incompatible contents.
- Renames and edits overlap in ways Git cannot safely reconcile.

**Illustrative example:** The original repository sets `pytest==9.1.1` in `requirements.txt`, while your fork sets `pytest==8.4.2` on the same line.

Git may mark the conflict like this:

```text
<<<<<<< HEAD
pytest==8.4.2
=======
pytest==9.1.1
>>>>>>> upstream/master
```

- `<<<<<<< HEAD` begins the version from your currently checked-out branch during this merge.
- `=======` separates the two versions.
- `>>>>>>> upstream/master` ends the incoming version.

You must choose the correct version, or manually write a compatible combined result, and remove the conflict markers. **Do not automatically choose one side without understanding the code.**

The screenshot did **not** identify the actual conflicting files; the `requirements.txt` example is hypothetical.

## 5. Safe synchronization: step-by-step

### Step 0 — Work in the repository root

```powershell
cd 'D:\MLOPS_Shivam_prasad\MLOPS_Github_Code\v1-aiml-mlops-sep-2026'
```

### Step 1 — Verify the branch and clean state

```powershell
git status
git branch --show-current
git remote -v
```

If you have uncommitted changes, **commit them or stash them first**. Do not begin a complex merge with unprotected work.

### Step 2 — Add the original repository as upstream

```powershell
git remote add upstream https://github.com/shivam13juna/v1-aiml-mlops-sep-2026.git
```

Then:

```powershell
git remote -v
```

Expected remote names: `origin` (your fork) and `upstream` (original).

If Git reports `remote upstream already exists`, do **not** add it again. Inspect `git remote -v`; if the URL is wrong, correct it deliberately with `git remote set-url upstream <correct-url>`.

### Step 3 — Refresh your fork and make a backup

```powershell
git fetch origin
git switch master
git status
git branch backup-master-before-sync
```

The backup branch points to your current local commits. Keep it until you have verified the merge.

Because the earlier `git status` only reflects locally known remote-tracking state, confirm your local branch is up to date with your fork:

```powershell
git pull --ff-only origin master
```

If fast-forward fails, **stop** and inspect the differences instead of forcing the pull. You can update your backup branch after the fast-forward if you want the backup to include those newly retrieved commits.

### Step 4 — Fetch original-repository updates

```powershell
git fetch upstream
```

See commits in upstream that local `master` does not have:

```powershell
git log --oneline master..upstream/master
```

See commits in local `master` that upstream does not have:

```powershell
git log --oneline upstream/master..master
```

Optional graphical overview:

```powershell
git log --oneline --graph --decorate --all -20
```

These commands **inspect** changes; they do not merge or delete files.

### Step 5 — Merge upstream into your local master

```powershell
git merge upstream/master
```

**Outcome A: Automatic merge succeeds.** Continue to verification and push.

**Outcome B: Git reports conflicts.** Do not push yet. Run:

```powershell
git status
```

Git lists files that need resolution.

### Step 6 — Resolve conflicts in VS Code

1. Open the repository in VS Code: `code .`
2. Open the Source Control view and inspect files marked as conflicted.
3. Open each conflict in the Merge Editor (when offered), or edit the conflict markers manually.
4. Decide whether to keep your change, the incoming upstream change, or a compatible combination.
5. Save each resolved file.
6. Run relevant tests, especially if Python files or dependencies changed.

For your basic pytest demo, one useful targeted test is:

```powershell
python -m pytest lec9_CI/basic_pytest_demo/test_base_file.py -v
```

If your repository's tests have other dependencies or paths, adjust the command accordingly.

Then stage the resolved files:

```powershell
git add .
git status
```

Once all conflicts are resolved, finish the merge:

```powershell
git commit
```

Git may open an editor with a default merge commit message. Save and close it to complete the commit.

If Git already completed an automatic merge commit, **do not run an extra `git commit` just for the merge**.

### Step 7 — Verify before publishing

```powershell
git status
git log --oneline --graph --decorate -12
```

Optionally rerun project tests. Check that your own documentation, code, and `.github/workflows` files are still correct.

### Step 8 — Push to your fork

```powershell
git push origin master
```

This pushes to **your fork**, not to the instructor's repository.

Return to GitHub and refresh the fork page. If no new upstream commits arrived, the “behind” count should be cleared. Your fork can still show commits “ahead” because your personal work remains in it.

## 6. Recovery and safety commands

| Situation | Safe response |
|---|---|
| You have not started a merge yet | Inspect `git status` and make a backup branch. |
| You started a merge and want to cancel it | `git merge --abort` (only while a merge is in progress). |
| You want to see unresolved paths | `git status` or `git diff --name-only --diff-filter=U` |
| You need to inspect the original commit state | `git log backup-master-before-sync --oneline -10` |
| `git push` is rejected | Fetch and inspect divergence; do not immediately force-push. |
| GitHub offers “Discard 8 commits” | Avoid it when preserving your own commits is the goal. |

**Do not use `git reset --hard upstream/master` or `git push --force` as a quick fix.** Those commands can discard or overwrite your fork's work if used incorrectly.

## 7. Your routine for future updates

After the initial configuration and successful merge, a common routine is:

```powershell
git status
git switch master
git fetch origin
git pull --ff-only origin master
git fetch upstream
git merge upstream/master
# Resolve conflicts if needed; verify tests.
git push origin master
```

The process is **not real-time automatic synchronization**. You run it when you want the latest upstream changes, or configure a separate automation later.

## 8. Frequently asked questions

### If local `master` is “up to date with origin/master”, why does GitHub say 3 behind?

Because `origin/master` refers to **your fork**, while GitHub's “3 behind” compares **your fork with the original repository**. Both statements can be true at the same time.

### Does `git fetch upstream` overwrite my code?

No. It downloads remote history and updates `upstream/*` references without changing your checked-out files.

### Does `git merge upstream/master` erase my 8 commits?

A normal merge is designed to preserve both histories. Conflicts must be resolved correctly; a destructive reset is different from a merge.

### Why does `job2` in GitHub Actions not matter to this merge?

GitHub Actions CI/CD and Git branch synchronization are separate concerns. After merging, a workflow might run tests on the pushed commits, but the workflow does not automatically resolve Git conflicts.

### Can I keep my fork ahead of the original?

Yes. That's normal when you add your own notes, code, and workflows to a fork. The goal is usually to **incorporate upstream changes while preserving your own work**, not to make both repositories identical.

## 9. Command cheat sheet

| Command | Meaning |
|---|---|
| `git status` | Show branch, staged/unstaged files, and conflicts. |
| `git branch --show-current` | Show current local branch. |
| `git remote -v` | Show fetch/push remote URLs. |
| `git remote add upstream URL` | Add the original repository as a remote. |
| `git branch backup-master-before-sync` | Create a local backup branch. |
| `git fetch upstream` | Download upstream history without merging. |
| `git log --oneline master..upstream/master` | Show upstream-only commits relative to local master. |
| `git merge upstream/master` | Merge upstream changes into local master. |
| `git diff --name-only --diff-filter=U` | List unresolved conflict files. |
| `git merge --abort` | Cancel an in-progress merge. |
| `git add .` | Stage resolved files and other changes. |
| `git commit` | Complete a manually resolved merge. |
| `git push origin master` | Publish your local master to your fork. |

## 10. Interview-ready summary

**Q: What does “8 commits ahead and 3 commits behind” mean on a GitHub fork?**  
A: The fork has eight commits missing from upstream, and upstream has three commits missing from the fork. The histories have diverged. Fetching and merging upstream can incorporate the missing changes while preserving the fork's own commits.

**Q: What is a Git merge conflict?**  
A: A conflict occurs when Git cannot safely reconcile changes from two histories. A developer must inspect the conflicting edits, choose or combine the appropriate content, test the result, and complete the merge.

**Q: What is the difference between `origin` and `upstream`?**  
A: `origin` usually points to your fork, and `upstream` points to the original repository from which you forked.

**Q: Why create a backup branch before a merge?**  
A: It retains an easy-to-find reference to your pre-merge commit history in case you need to inspect or recover earlier work.

---

**Core lesson:** Keep your work, fetch upstream updates, resolve conflicts carefully, test the merged code, and push the result to your own fork. **Never discard unique commits merely to remove the “behind” warning.**
