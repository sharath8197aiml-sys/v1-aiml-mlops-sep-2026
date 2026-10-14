# Git Fork Synchronization — Detailed Future Workflow

**Project:** `v1-aiml-mlops-sep-2026`  
**Your fork (`origin`):** `https://github.com/sharath8197aiml-sys/v1-aiml-mlops-sep-2026`  
**Instructor's repository (`upstream`):** `https://github.com/shivam13juna/v1-aiml-mlops-sep-2026`  
**Working branch:** `master`  
**Environment:** Windows PowerShell / VS Code

## 1. Goal

Keep three locations synchronized **without discarding your own commits**:

```text
Instructor GitHub repository (upstream/master)
                  |
             git fetch upstream
                  v
Your laptop (local master) -- git push origin master --> Your GitHub fork (origin/master)
                  ^
                  |
          git pull --ff-only origin master
```

- `origin` points to **your fork**. You can normally push your work here.
- `upstream` points to the **instructor's original repository**. You fetch new lessons and code here.
- `master` is the local branch on your laptop.
- `upstream/master` and `origin/master` are local *remote-tracking references* updated by fetching. They are not separate working folders.

**Important:** `git fetch` does not change your working files. `git merge` does. `git push` updates the remote repository. Git does not continuously sync automatically.

## 2. First-time configuration (do only once)

Open VS Code terminal at the repository root:

```powershell
cd 'D:\MLOPS_Shivam_prasad\MLOPS_Github_Code\v1-aiml-mlops-sep-2026'
git remote -v
git branch --show-current
```

Expected remotes:

```text
origin    https://github.com/sharath8197aiml-sys/v1-aiml-mlops-sep-2026 (fetch)
origin    https://github.com/sharath8197aiml-sys/v1-aiml-mlops-sep-2026 (push)
upstream  https://github.com/shivam13juna/v1-aiml-mlops-sep-2026 (fetch)
upstream  https://github.com/shivam13juna/v1-aiml-mlops-sep-2026 (push)
```

If `upstream` is **missing**, add it:

```powershell
git remote add upstream https://github.com/shivam13juna/v1-aiml-mlops-sep-2026.git
```

If you see `error: remote upstream already exists`, **do not add it again**. Verify with `git remote -v`. If its URL is incorrect, update it:

```powershell
git remote set-url upstream https://github.com/shivam13juna/v1-aiml-mlops-sep-2026.git
```

## 3. Safe routine whenever the instructor updates the repository

### Step 1 — Inspect local work

```powershell
git status
git branch --show-current
```

Proceed when `master` is checked out and `nothing to commit, working tree clean` appears. If files are modified, either commit intentional changes first or safely stash them. Do not casually discard files.

**Option A — commit your work:**

```powershell
git add <specific-file-or-folder>
git commit -m "Save local work before upstream sync"
```

Replace `<specific-file-or-folder>` with the actual path; review `git status` before committing. Avoid accidentally committing `.env`, `__pycache__`, or other generated files.

**Option B — temporarily stash work, including untracked files:**

```powershell
git stash push -u -m "Before upstream sync"
```

After synchronization, restore it with `git stash pop`; this may itself create conflicts. Check `git stash list` before applying.

### Step 2 — Switch to master

```powershell
git switch master
```

### Step 3 — Update local master from your fork

```powershell
git fetch origin
git pull --ff-only origin master
```

`--ff-only` updates your local branch only if Git can fast-forward it. It refuses to create an unexpected merge. If it fails due to divergence, stop and inspect the histories (`git log --oneline --graph --decorate --all -20`) instead of force-pushing.

### Step 4 — Download upstream updates

```powershell
git fetch upstream
```

This refreshes `upstream/master` and downloads objects but **does not merge them** into local `master`.

### Step 5 — Inspect what changed

```powershell
git log --oneline master..upstream/master
git log --oneline upstream/master..master
git diff --stat master upstream/master
```

- First command: commits upstream has that local master does not.
- Second command: commits local master has that upstream does not.
- Third command: summary of differences in file contents.

If the first command returns no commits, your branch already contains all commits from the fetched upstream branch; no upstream merge is necessary.

### Step 6 — Optional backup before merging

```powershell
git branch backup-master-2026-10-14
```

Use a new descriptive backup name each time (the date above is only an example). A backup branch points to the current commit; it is not a second physical copy of your project. Check existing names with `git branch`.

### Step 7 — Merge upstream into your local branch

```powershell
git merge upstream/master
```

**Outcome A: Automatic merge succeeds.** Git may create a merge commit or fast-forward, depending on history. Continue to validation.

**Outcome B: Conflict.** Git pauses the merge and prints affected paths. **Do not push yet.** Follow Section 4 below.

### Step 8 — Validate merged files and tests

```powershell
git status
git log -1 --oneline
python -m pytest lec9_CI/basic_pytest_demo/test_base_file.py -v
```

Run other tests appropriate to the changed code. Review `.github/workflows/` if the instructor changed CI files: two workflows with overlapping push triggers can run twice. A clean `git status` after a completed merge is expected unless you intentionally have other changes.

### Step 9 — Push the merged branch to your fork

```powershell
git push origin master
```

This updates **your fork**, not the instructor's repository. You should not need `--force` for the normal workflow.

### Step 10 — Verify synchronization

```powershell
git status
git fetch origin
git fetch upstream
git log --oneline master..upstream/master
git log --oneline origin/master..master
```

If the last two log commands produce no output, your local `master` contains the fetched upstream commits and has no unpushed commits relative to your fork. Your fork can still show **ahead of upstream**, because it contains your own unique commits; that is expected.

## 4. Resolving merge conflicts safely

### 4.1 Identify conflicted files

```powershell
git status
git diff --name-only --diff-filter=U
```

Open conflicted files in VS Code and review the changes. Git may show conflict markers for competing text edits:

```text
<<<<<<< HEAD
Your version
=======
Instructor's version
>>>>>>> upstream/master
```

Edit the file to its intended final contents and remove the markers. Then stage **only the resolved file**:

```powershell
git add path/to/conflicted_file.py
```

Repeat for each conflict; verify with `git status`, then complete the merge:

```powershell
git commit
```

If Git automatically completed the merge, do **not** create an unnecessary extra commit.

### 4.2 Modify/delete conflict — your actual example

You previously saw:

```text
CONFLICT (modify/delete): .github/workflows/continuous_integration.yaml
 deleted in upstream/master and modified in HEAD.
```

This means the instructor deleted the old workflow while you had modified it. You must choose deliberately:

**Keep your local workflow file:**

```powershell
git add .github/workflows/continuous_integration.yaml
```

**Accept upstream's deletion:**

```powershell
git rm .github/workflows/continuous_integration.yaml
```

Then inspect `git status` and commit the merge. In your earlier merge, `git add .` kept the old workflow, while upstream added `continuous_integration_and_deployment.yaml`. Review whether both workflows should remain enabled.

### 4.3 Abort a merge that is still in progress

```powershell
git merge --abort
```

This attempts to restore the pre-merge state. Use it **before** the merge commit, not after. It may be unable to reconstruct unrelated uncommitted modifications perfectly; start merges with a clean tree.

### 4.4 Recovery using a backup branch

Inspect a backup:

```powershell
git log --oneline backup-master-before-sync -5
git diff backup-master-before-sync master --stat
```

You can restore individual files from the backup if necessary:

```powershell
git restore --source=backup-master-before-sync -- path/to/file
```

Review changes before committing. Avoid `git reset --hard` or `git push --force` unless you understand their destructive consequences.

## 5. Common errors and what they mean

| Message | Meaning | Recommended response |
|---|---|---|
| `remote upstream already exists` | The remote is configured | Use `git remote -v`; don't add it again |
| `nothing to commit, working tree clean` | No pending local file changes | Safe starting point for fetch/merge |
| `Already up to date.` | No new commits to integrate for that operation | Continue with other checks |
| `Automatic merge failed` | Conflicts need resolution | Inspect `git status`, resolve, stage, commit |
| `deleted by them` | Incoming branch deleted a file you changed | Decide whether to keep or delete it |
| `fatal: Not possible to fast-forward` | Local and remote histories diverged | Inspect commits; don't force push |
| `rejected (non-fast-forward)` | Remote has commits your local branch lacks | Fetch and integrate the remote changes |
| `Permission denied` / `403` | Wrong credentials or no write permission | Verify `origin` and GitHub authentication |
| `LF will be replaced by CRLF` | Line-ending conversion warning | Usually harmless; consider `.gitattributes` |

## 6. Prevent generated files from cluttering commits

Create or update `.gitignore` at the repository root:

```gitignore
__pycache__/
*.py[cod]
.pytest_cache/
.venv/
.env
```

`.gitignore` only affects **untracked** files. Already tracked caches must be explicitly removed from Git's index (without deleting your local copies), then committed. Review each path before removing it. Keep or exclude model files such as `classifier.pkl` according to your project's artifact policy.

## 7. Copy-and-paste daily checklist

**Run only after confirming the working tree is clean and your branch is `master`:**

```powershell
git status
git switch master
git fetch origin
git pull --ff-only origin master
git fetch upstream
git log --oneline master..upstream/master
git diff --stat master upstream/master
git merge upstream/master
# STOP HERE if conflicts occur; resolve them and finish the merge.
python -m pytest lec9_CI/basic_pytest_demo/test_base_file.py -v
git status
git push origin master
```

Do not paste the whole block blindly if a command fails. Inspect each result, particularly after `git pull`, `git merge`, and tests.

## 8. Interview-ready explanations

**Q: What is the difference between `origin` and `upstream`?**  
`origin` usually names your fork; `upstream` usually names the source repository you forked from. These are conventions, not reserved Git keywords.

**Q: What is the difference between `fetch`, `pull`, and `merge`?**  
`fetch` downloads remote commits and updates remote-tracking refs. `merge` integrates another branch into the current branch. `pull` normally performs a fetch followed by integration (merge or rebase, depending on configuration); `--ff-only` permits only a fast-forward.

**Q: Why can my fork be ahead of upstream after synchronization?**  
Your fork can contain your own commits that the instructor's repository does not have. Being ahead is not inherently a problem.

**Q: Why use a backup branch?**  
It preserves an easy-to-find pointer to the pre-merge commit, helping inspect or recover earlier content.

**Q: Why shouldn't I use `git push --force` for normal synchronization?**  
Force pushing can overwrite remote branch history and potentially discard others' work. Normal fetch, merge, and push preserve history.

---

**Remember:** **Inspect → Commit/stash → Fetch origin → Fast-forward local → Fetch upstream → Compare → Merge → Resolve → Test → Push origin → Verify.**
