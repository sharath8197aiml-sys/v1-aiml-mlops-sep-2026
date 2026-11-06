# Git Upstream Synchronization and Merge Conflict Resolution

## Practical case study: `v1-aiml-mlops-sep-2026`

This guide documents the two stages of the October 31 synchronization discussion: (1) fetching and inspecting the instructor's latest changes, and (2) encountering and safely resolving a CI/CD YAML merge conflict. It includes text-only diagrams, PowerShell commands, explanations, and recovery options.

> **Current state in the provided terminal output:** A merge is in progress and **has not yet been completed**. The conflicting file is `.github/workflows/continuous_integration_and_deployment.yaml`. Do not push until the conflict is resolved and the merge commit is created.

## 1. Understand the three repository references

```text
Instructor's GitHub repository
shivam13juna/v1-aiml-mlops-sep-2026
       |
       | git fetch upstream
       v
Local remote-tracking reference: upstream/master
       |
       | git merge upstream/master
       v
Your local branch: master
       |
       | git push origin master
       v
Your GitHub fork: origin/master
sharath8197aiml-sys/v1-aiml-mlops-sep-2026
```

- **`upstream`**: the instructor's original repository.
- **`origin`**: your GitHub fork.
- **`master`**: your checked-out local branch.
- **`upstream/master`** and **`origin/master`**: local references representing the most recently fetched states of those remote branches.

## 2. First terminal session: what happened and why

### 2.1 `git switch master`

```powershell
git switch master
```

Output: `Already on 'master'` and `Your branch is ahead of 'origin/master' by 1 commit.`

You were already on the correct branch. **One local commit had not yet been published to your fork**. This does not prevent fetching upstream or merging, but you should preserve it.

### 2.2 Incorrect remote name: `origin-master`

```powershell
git pull --ff-only origin-master
```

Git responded that `origin-master` did not appear to be a repository. Git interpreted it as one remote name. The correct syntax separates remote and branch:

```powershell
git pull --ff-only origin master
```

The corrected command returned `Already up to date.` Your local branch contained the commits in the fetched fork branch. **This does not mean your one unpublished local commit had been pushed.**

### 2.3 Fetch the instructor's changes

```powershell
git fetch upstream
```

Output included:

```text
60b53a0..07d1c7a  master -> upstream/master
```

Git downloaded upstream commits and updated its local `upstream/master` reference from `60b53a0` to `07d1c7a`. It did **not** change your checked-out files.

### 2.4 Inspect missing commits

```powershell
git log --oneline master..upstream/master
```

The output included commits about SageMaker lectures, folder renaming, environment setup, and a rollback workflow. The later GitHub screenshot reported 15 commits behind; the log displayed 15 upstream-only entries in this session.

Git opened a pager, showing `~` and `(END)`. This is **not an error**. Press **`q`** to exit.

To avoid the pager:

```powershell
git --no-pager log --oneline master..upstream/master
```

### 2.5 Understanding `A..B`

```text
                 A (local master) ---- local-only commit
                /
Shared history --
                \
                 B (upstream/master) -- upstream-only commits
```

- `git log master..upstream/master`: commits reachable from upstream but not from local master.
- `git log upstream/master..master`: commits reachable from local master but not from upstream.

These commands **inspect history only**; they do not merge or modify files.

## 3. Second terminal session: the merge conflict

### 3.1 Create a safety branch

```powershell
git branch backup-before-oct31-sync
```

This creates a branch pointer at your current local commit. It does **not** switch branches or duplicate all files.

```text
Before merge:

backup-before-oct31-sync ----+
                             |
master ----------------------o  (your current local commit)
```

### 3.2 Attempt the upstream merge

```powershell
git merge upstream/master
```

Git reported:

```text
Auto-merging .github/workflows/continuous_integration_and_deployment.yaml
CONFLICT (content): Merge conflict in .github/workflows/continuous_integration_and_deployment.yaml
Automatic merge failed; fix conflicts and then commit the result.
```

This is a **content conflict**: both sides edited overlapping portions of the same workflow file, so Git cannot safely choose the intended final text.

```text
               Your local master
               modifies workflow YAML
              /
Common base --X  <-- conflicting edits
              \
               Instructor upstream/master
               modifies workflow YAML

Resolution: review BOTH versions -> edit final YAML -> stage -> commit
```

**Do not assume that the local or upstream version is automatically correct.** A CI/CD workflow can include important triggers, build steps, secrets, deployment jobs, or rollback logic.

### 3.3 Interpret `git status` during a merge

Your status showed:

```text
You have unmerged paths.
Changes to be committed:
    new file: ../.github/workflows/rollback.yml
    ... SageMaker lecture files ...
    modified: ../lec8 Deploying on AWS ECS/jmeter.log
Unmerged paths:
    both modified: ../.github/workflows/continuous_integration_and_deployment.yaml
```

Meaning:

- **Changes to be committed**: non-conflicting changes already staged as part of the pending merge. These included SageMaker lectures and the rollback workflow.
- **Unmerged paths**: the one YAML file that still requires manual resolution.
- **Ahead of `origin/master` by 1 commit**: your local unpublished commit still exists. The pending merge has not been committed.
- **`git merge --abort` available**: Git can attempt to return to the pre-merge state if you decide not to proceed.

The `../` prefix appears because your PowerShell prompt is in the `lec9_CI` subdirectory while the affected files are relative to the repository root.

## 4. Resolve the conflict safely in VS Code

### Step 1: Go to the repository root

From `...\v1-aiml-mlops-sep-2026\lec9_CI`:

```powershell
cd ..
git status
```

Do **not** start a second merge. The existing merge is already in progress.

### Step 2: Inspect the conflict

```powershell
git diff -- .github/workflows/continuous_integration_and_deployment.yaml
```

For the complete versions stored in Git's merge index:

```powershell
# Stage 1: common ancestor (if available)
git show :1:.github/workflows/continuous_integration_and_deployment.yaml

# Stage 2: ours = your local master
git show :2:.github/workflows/continuous_integration_and_deployment.yaml

# Stage 3: theirs = instructor's upstream/master
git show :3:.github/workflows/continuous_integration_and_deployment.yaml
```

`git show` prints the versions for comparison. These commands do not resolve the conflict automatically.

### Step 3: Open the file

```powershell
code .github/workflows/continuous_integration_and_deployment.yaml
```

You may see conflict markers like these (illustrative example, **not** your actual YAML):

```yaml
<<<<<<< HEAD
name: Local CI CD Pipeline
=======
name: Instructor CI CD Pipeline
>>>>>>> upstream/master
```

```text
<<<<<<< HEAD             Local version starts
  your changes
=======                  Separator
  instructor's changes
>>>>>>> upstream/master  Instructor version ends
```

VS Code may offer **Accept Current**, **Accept Incoming**, **Accept Both**, or a Merge Editor. These are editing aids, not guarantees of correct CI/CD behavior.

### Step 4: Make an intentional resolution

Review each conflict and produce one valid YAML document. For example, if the two sides change different desired jobs, you may need to integrate both jobs with correct dependencies rather than blindly selecting one entire side.

Check specifically:

- Workflow triggers (`on:`), branch filters, and permissions.
- Job IDs and `needs:` dependencies.
- Python/test steps, Docker build/push, and deployment/rollback steps.
- GitHub Actions secret references and environment names.
- Indentation and duplicate YAML keys.
- Whether the new `rollback.yml` interacts with this workflow.

Do not paste secrets into the YAML. Keep `${{ secrets.NAME }}` references where appropriate.

Remove all `<<<<<<<`, `=======`, and `>>>>>>>` conflict markers and save the file. **The actual conflicting sections were not supplied in the terminal output**, so no exact merged YAML can be prescribed without seeing the file.

### Step 5: Check for leftover conflict markers

```powershell
git diff --check
Select-String -Path .github/workflows/continuous_integration_and_deployment.yaml -Pattern '^(<<<<<<<|=======|>>>>>>>)'
```

`git diff --check` can catch whitespace issues, but it is **not a complete YAML or GitHub Actions validator**. `Select-String` should return no conflict-marker lines. Review the file and run relevant tests or workflow validation before deploying.

### Step 6: Stage only the resolved file

```powershell
git add .github/workflows/continuous_integration_and_deployment.yaml
git status
```

The `Unmerged paths` section should disappear. Other upstream changes should remain staged. Prefer this targeted `git add` over `git add .` so unrelated edits are not accidentally included.

### Step 7: Complete the merge commit

```powershell
git commit -m "Merge upstream master and resolve CI CD workflow conflict"
```

This records the merge result with both histories as parents. It preserves your earlier local commit.

### Step 8: Review and test before publishing

```powershell
git status
git --no-pager log --oneline -6
```

Run tests appropriate to the repository. Git's successful merge does **not** guarantee that the YAML workflow or deployment works correctly.

### Step 9: Push to your fork

```powershell
git push origin master
```

If the push is rejected because the remote advanced, **do not force-push**. Fetch `origin`, inspect differences, and integrate them safely before trying again.

### Step 10: Verify synchronization

```powershell
git status
git --no-pager log --oneline master..upstream/master
```

No output from the second command means local `master` includes all commits from the **last fetched** `upstream/master`. If upstream changed since the fetch, fetch again before making a claim about its current state.

## 5. End-to-end flowchart (text only)

```text
START
  |
  v
Check git status
  |
  +-- Uncommitted changes? -- YES --> Commit or stash safely
  |                                     |
  +------------------- NO <-------------+
  |
  v
Switch to master
  |
  v
Fetch origin and sync fork (if fast-forward possible)
  |
  v
Create backup branch
  |
  v
Fetch upstream
  |
  v
Inspect upstream-only commits
  |
  v
Merge upstream/master
  |
  +-- Merge succeeds? -- YES -------------------------+
  |                                                   |
  NO                                                  |
  |                                                   |
  v                                                   |
Inspect conflict in VS Code                          |
  |                                                   |
  v                                                   |
Compare ours / theirs / common ancestor               |
  |                                                   |
  v                                                   |
Edit final YAML and remove conflict markers           |
  |                                                   |
  v                                                   |
Stage resolved file                                   |
  |                                                   |
  v                                                   |
Commit merge -----------------------------------------+
  |
  v
Review/test code and CI configuration
  |
  v
Push origin master
  |
  v
Verify clean status and upstream history
  |
  v
DONE
```

## 6. Recovery and mistakes to avoid

### Abort an unfinished merge

```powershell
git merge --abort
```

Use this **only while the merge is in progress** if you want to cancel it. It attempts to restore the pre-merge state; if you made unrelated changes during the merge, inspect and protect them first.

### Return to the safety branch for inspection

```powershell
git --no-pager log --oneline backup-before-oct31-sync -5
```

Your backup branch remains at the pre-merge commit. Do not reset or overwrite `master` casually.

### Do not use these as shortcuts

```text
git push --force     # can overwrite remote history
git reset --hard     # can discard uncommitted work
Discard commits      # can remove your fork's unique commits
```

### Common mistakes

| Mistake | Safer approach |
|---|---|
| Typing `origin-master` as a remote | Use `origin master` or `origin/master` in the appropriate command |
| Mistaking `(END)` for an error | Press `q` to exit Git's pager |
| Assuming `fetch` merges code | Fetch downloads; merge integrates |
| Pushing during an unresolved merge | Resolve, stage, commit, test, then push |
| Choosing 'Accept Both' blindly | Inspect YAML structure and workflow semantics |
| Using `git add .` without review | Stage the specific resolved file |
| Assuming successful merge means successful deployment | Validate workflow and run tests |

## 7. Command cheat sheet

```powershell
# Before any new synchronization
git status
git switch master
git fetch origin
git pull --ff-only origin master

# Protect your local state (use a unique branch name)
git branch backup-before-sync

# Inspect instructor changes
git fetch upstream
git --no-pager log --oneline master..upstream/master

# Integrate instructor changes
git merge upstream/master

# If conflict occurs, inspect and resolve in editor
git status
git diff -- .github/workflows/continuous_integration_and_deployment.yaml
code .github/workflows/continuous_integration_and_deployment.yaml

# After manual resolution
git add .github/workflows/continuous_integration_and_deployment.yaml
git status
git commit -m "Merge upstream master and resolve CI CD workflow conflict"

# After testing
git push origin master
git status
```

**Important:** The commands in the cheat sheet describe a *conditional workflow*. Do not run `git commit` unless a merge is awaiting completion, and do not run `git push` until the merge is complete and validated. If `git pull --ff-only` fails because the fork and local branch diverged, inspect the histories and resolve that divergence before proceeding.

## 8. Interview-ready concepts

**What is a merge conflict?** A case where Git cannot automatically determine a correct combined result from changes made on different branches.

**What is the difference between `fetch` and `merge`?** Fetch updates remote-tracking references and downloads objects; merge integrates another commit history into your checked-out branch.

**What does `HEAD` mean in a merge conflict?** In this merge, it refers to your checked-out local `master` version (`ours`).

**What is a merge commit?** A commit with two or more parent commits recording the integration of histories.

**Why create a backup branch?** It provides a stable reference to your pre-merge commit, making inspection and recovery easier.

**What is a clean working tree?** No unstaged or staged uncommitted changes in the checked-out worktree.

**Why is a CI/CD YAML conflict risky?** A syntactically valid resolution can still change triggers, permissions, test jobs, deployment behavior, or rollback safety.

---

**Case-study conclusion:** The transcript confirms that the upstream fetch succeeded, a backup branch was created, and the merge encountered exactly one reported content conflict in the CI/CD workflow YAML. It does **not** yet show a resolved file, merge commit, or successful push for this October 31 synchronization.
