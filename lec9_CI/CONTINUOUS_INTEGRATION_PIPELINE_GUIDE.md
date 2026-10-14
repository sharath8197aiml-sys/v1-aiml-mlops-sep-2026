# GitHub Actions CI/CD Pipeline — `continuous_integration.yaml`

A detailed reference for the `v1-aiml-mlops-sep-2026` learning project, based on the uploaded workflow and our discussion.

## 1. What is YAML?

**YAML** means **YAML Ain't Markup Language** (originally *Yet Another Markup Language*). It is a human-readable format for structured configuration. GitHub Actions, Docker Compose, Kubernetes, and Ansible commonly use YAML.

```yaml
name: Sharath
role: AI Engineer
experience: 5
skills:
  - Python
  - Docker
  - Pytest
```

Equivalent Python data:

```python
{"name": "Sharath", "role": "AI Engineer", "experience": 5,
 "skills": ["Python", "Docker", "Pytest"]}
```

| Syntax | Meaning | Example |
|---|---|---|
| `key: value` | Mapping entry | `name: CI` |
| Indentation | Nesting | `jobs:` → `job1:` |
| `-` | List item | `- main` |
| `#` | Comment | `# Run tests` |
| `\|` | Multiline literal | `run: \|` |
| `${{ ... }}` | GitHub Actions expression | `${{ secrets.DOCKER_USERNAME }}` |

Use **spaces, not tabs**, for indentation.

## 2. Your uploaded workflow

The workflow is named `CI`. It responds to pushes on `main`, `master`, and the **literal** branch name `any-feature-branch`. It defines two jobs:

1. `job1`: Ubuntu runner → checkout → Python 3.12 → install dependencies → pytest.
2. `job2`: waits for `job1` → Ubuntu runner → checkout → Docker Hub login → build and push an image.

```text
Push to main / master / any-feature-branch
                 |
                 v
         JOB 1: TESTING
     Checkout repository
       Set up Python 3.12
     Install requirements
           Run pytest
                 |
          success only
                 v
       JOB 2: PUBLISH
      Checkout repository
       Docker Hub login
       Build Docker image
      Push image to Hub
                 |
                 v
shivam13juna/oct7_2026_live_ci:latest
```

**This workflow publishes a Docker image; it does not deploy it to AWS ECS or another runtime.**

## 3. Workflow name: `name`

```yaml
name: CI
```

This label appears in the GitHub repository's **Actions** tab. You could use `name: Python Tests and Docker Build` instead. The label does not determine execution.

## 4. Trigger: `on`, `push`, and `branches`

```yaml
on:
  push:
    branches:
      - main
      - master
      - any-feature-branch
```

- `on`: event that starts a workflow.
- `push`: a qualifying push to GitHub triggers it.
- `branches`: restricts the matching branches.
- `any-feature-branch` matches **only that exact name**; it is not a wildcard.

| Pushed branch | Runs? |
|---|---|
| `main` | Yes |
| `master` | Yes |
| `any-feature-branch` | Yes |
| `feature/login` | No |
| `develop` | No |

To match feature branches:

```yaml
on:
  push:
    branches:
      - main
      - master
      - 'feature/**'
```

To run on all branches:

```yaml
on:
  push:
```

To test pull requests as well:

```yaml
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]
```

## 5. Jobs and runners

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      # ...
  job2:
    needs: job1
    runs-on: ubuntu-latest
    steps:
      # ...
```

A **job** is a sequence of steps on a runner. A **runner** is the machine executing the job. `ubuntu-latest`, `windows-latest`, and `macos-latest` are possible runner labels.

Your laptop uses Windows, but this workflow runs on Ubuntu. Avoid hardcoded Windows paths such as `D:\...` in commands intended for Ubuntu.

Each job normally gets its **own runner**. Files produced in `job1` do not automatically appear in `job2`; use uploaded/downloaded artifacts if they must be transferred.

## 6. Checkout step: `uses`

```yaml
- name: Checkout code
  uses: actions/checkout@v7
```

- `-` marks a step in the steps list.
- `name` is its display label.
- `uses` invokes a reusable GitHub Action.
- `@v7` denotes an action release reference, **not a branch name**.

Checkout places the workflow repository's code in the runner workspace so later steps can read the files. Each of your two jobs checks out the repository separately.

**Version caution:** These are the version references in your uploaded file; check their availability against the official action releases before using them.

## 7. Set up Python: `with`

```yaml
- name: Set up Python
  uses: actions/setup-python@v7
  with:
    python-version: '3.12'
```

`with` passes input parameters to an action. Here the runner uses Python 3.12, regardless of the Python 3.14 environment shown on your Windows laptop. Choose a version compatible with the project dependencies.

## 8. Install dependencies: `run: |`

```yaml
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
```

`run` executes shell commands. The `|` keeps the following indented lines as a multiline script. First pip is upgraded; then dependencies are installed from `requirements.txt`.

The uploaded workflow assumes `requirements.txt` is in the **repository root**. If it is instead inside `lec9_CI`, adjust the command:

```yaml
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r lec9_CI/requirements.txt
```

This alternative works only if that path actually exists. A sample requirements file might list `pytest`, `flask`, `scikit-learn`, and `joblib`; this is illustrative, not a verified inventory of your repository.

## 9. Run tests with pytest

```yaml
- name: Run tests
  run: |
    pytest
```

Pytest discovers matching test files (including `test_*.py`) according to its discovery configuration. If tests pass, pytest normally returns exit code `0`; a test failure or collection error normally returns a nonzero code, failing the step and job.

For your `basic_pytest_demo` exercise, the test module has four test functions and 13 assertions. You can target it specifically:

```yaml
- name: Run tests
  run: |
    python -m pytest lec9_CI/basic_pytest_demo/test_base_file.py -v
```

This avoids collecting the unrelated `test_flask.py` that previously failed due to missing `classifier.pkl`. The path assumes the same repository layout discussed earlier.

## 10. Job dependency: `needs: job1`

```yaml
job2:
  needs: job1
  runs-on: ubuntu-latest
```

Jobs can otherwise run in parallel. `needs: job1` makes `job2` wait for a **successful** `job1` by default.

| Job 1 result | Job 2 |
|---|---|
| Tests pass | Starts |
| Test fails | Skipped |
| Dependency install fails | Skipped |
| Job 1 cancelled | Normally skipped |

This prevents publishing an image when tests fail.

## 11. Docker Hub login and GitHub Secrets

```yaml
- name: Login to DockerHub
  uses: docker/login-action@v4
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

`${{ ... }}` is GitHub Actions expression syntax. The `secrets` context supplies protected values at runtime.

Set up secrets in **Repository → Settings → Secrets and variables → Actions**:

| Secret | Value |
|---|---|
| `DOCKER_USERNAME` | Docker Hub account name |
| `DOCKER_PASSWORD` | Appropriately scoped Docker Hub access token |

Do **not** put actual credentials in the committed YAML. Secret availability also depends on event type and repository settings.

## 12. Build and push the Docker image

```yaml
- name: Build Docker image
  uses: docker/build-push-action@v7
  with:
    push: true
    tags: shivam13juna/oct7_2026_live_ci:latest
```

- `docker/build-push-action` builds the image.
- `push: true` publishes it to the configured registry.
- `tags` names the target image.

```text
shivam13juna / oct7_2026_live_ci : latest
   account       repository         tag
```

The uploaded workflow targets **`shivam13juna`**, not automatically your Docker Hub account. The authenticated account must have push permission for that repository.

A usable Dockerfile must be present in the expected build context. If it is under `lec9_CI`, configure `context` and `file` appropriately. The default setup does not deploy the image to a running service.

## 13. What happens after `git push`?

From a local clone, for example:

```bash
git add .
git commit -m "Update pytest examples"
git push origin master
```

If the workflow is committed at the correct location and the push matches its trigger:

1. GitHub detects the push.
2. `job1` starts on Ubuntu.
3. The code is checked out.
4. Python 3.12 and dependencies are prepared.
5. Pytest runs.
6. If successful, `job2` starts on another Ubuntu runner.
7. `job2` checks out code, logs in to Docker Hub, builds and pushes the image.

If pytest fails, `job2` is skipped by default.

## 14. Required workflow location

GitHub Actions workflows belong under `.github/workflows/` at the **repository root**:

```text
v1-aiml-mlops-sep-2026/
└── .github/
    └── workflows/
        └── continuous_integration.yaml
```

Putting the YAML only in `lec9_CI` does **not** register it as a GitHub Actions workflow.

## 15. Suggested improvements

### Test feature branches, publish only from a protected branch

Your original workflow can publish `latest` from all three configured branches. To restrict publishing to `master`:

```yaml
job2:
  if: github.ref == 'refs/heads/master'
  needs: job1
  runs-on: ubuntu-latest
```

If your publishing branch is `main`, change the condition accordingly. Add pull-request testing as needed, and keep publishing restricted to trusted pushes.

### Explicit test scope

Use `python -m pytest lec9_CI/basic_pytest_demo/test_base_file.py -v` if you want only the basic four tests; run the full suite only after all other test dependencies and artifacts are ready.

### Check external action versions

Your source uses `actions/checkout@v7`, `actions/setup-python@v7`, `docker/login-action@v4`, and `docker/build-push-action@v7`. These are preserved from your uploaded YAML, **not independently validated here**. Verify supported tags on their official GitHub repositories.

### Improve release traceability

Consider publishing a unique version or commit-SHA tag alongside `latest`, rather than overwriting only a mutable `latest` tag.

## 16. Where to inspect workflow results

1. Open your GitHub repository.
2. Select **Actions**.
3. Open the **CI** workflow.
4. Select a workflow run.
5. Open `job1` or `job2` to view step logs.

Use these logs to diagnose failed package installations, pytest errors, missing files, action-resolution problems, Docker authentication, and build failures.

## 17. Interview-ready glossary

| Term | Meaning |
|---|---|
| Workflow | Entire automation described in YAML |
| Trigger/event | The GitHub event that starts it |
| Job | Collection of steps on one runner |
| Runner | Machine executing a job |
| Step | Individual action or command |
| `uses` | Calls a reusable action |
| `run` | Runs shell commands |
| `with` | Passes action inputs |
| `needs` | Declares a job dependency |
| Secrets | Protected runtime configuration values |
| Artifact | Output retained or transferred from a workflow |
| Docker registry | Service that stores and distributes container images |

## 18. Key takeaways

- **YAML is configuration**, not Python code.
- **`on`** controls when the workflow starts.
- **`jobs` and `steps`** define what runs.
- **`needs`** gates Docker publishing on successful tests.
- **GitHub Secrets** avoid hardcoding Docker Hub credentials.
- **Docker publishing is not deployment** to AWS ECS or another server.
- **Paths matter:** commands run in the GitHub runner workspace, not on your Windows `D:` drive.
- Start with a passing pytest-only workflow, then enable Docker publishing and later deployment.

---

*Reference prepared from the uploaded `continuous_integration.yaml` and the step-by-step explanations in this conversation. Suggested improvements and example paths are identified as such.*
