# Understanding the `.github` Folder in GitHub

## 1. What is `.github`?

The `.github` directory stores GitHub-specific repository configuration, including GitHub Actions workflows, issue and pull-request templates, code ownership rules, and Dependabot configuration.

For the MLOps project, the most important subdirectory is **`.github/workflows/`**.

## 2. Example project structure

```text
v1-aiml-mlops-sep-2026/
├── .github/
│   ├── workflows/
│   │   ├── continuous_integration.yaml
│   │   └── deployment.yaml
│   ├── ISSUE_TEMPLATE/
│   │   └── bug_report.md
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── CODEOWNERS
│   └── dependabot.yml
├── lec9_CI/
│   ├── basic_pytest_demo/
│   │   ├── base_file.py
│   │   └── test_base_file.py
│   └── requirements.txt
├── Dockerfile
└── README.md
```

| Path | Purpose |
|---|---|
| `.github/workflows/` | GitHub Actions workflow definitions |
| `continuous_integration.yaml` | Automated testing and possibly image publishing |
| `deployment.yaml` | Optional deployment automation |
| `ISSUE_TEMPLATE/` | Issue templates |
| `PULL_REQUEST_TEMPLATE.md` | Pull-request description template |
| `CODEOWNERS` | Code ownership and review routing |
| `dependabot.yml` | Dependency update configuration |

## 3. Why `.github/workflows/` matters

GitHub Actions recognizes YAML workflow files under **`.github/workflows/`** at the repository root. A file at `lec9_CI/continuous_integration.yaml` is just an ordinary YAML file, not an automatically registered workflow.

Workflow files can use `.yml` or `.yaml` extensions.

## 4. Example Python pytest workflow

Save as `.github/workflows/continuous_integration.yaml`:

```yaml
name: CI

on:
  push:
    branches:
      - main
      - master

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install pytest
        run: python -m pip install pytest

      - name: Run tests
        run: python -m pytest lec9_CI/basic_pytest_demo/test_base_file.py -v
```

The action versions above are illustrative; check supported release tags when maintaining a real workflow.

### What happens on a push?

```text
Developer pushes to main/master
             |
             v
GitHub detects push event
             |
             v
Reads .github/workflows/continuous_integration.yaml
             |
             v
Starts Ubuntu runner
             |
             v
Checks out code and sets up Python
             |
             v
Runs pytest
       /           \
      v             v
 Tests pass      Tests fail
 Job succeeds    Job fails
```

Check the result in **GitHub repository → Actions**.

## 5. Is `.github` mandatory?

No. You can use Git, push code, clone repositories, and run pytest locally without `.github`.

| Task | Need `.github`? |
|---|---|
| Store Python code | No |
| Clone, pull, or push | No |
| Run pytest on your laptop | No |
| Configure GitHub Actions | Yes: `.github/workflows/` |
| Define repository issue templates | Yes, for repository-managed templates |
| Configure Dependabot with YAML | Yes: `.github/dependabot.yml` |

## 6. Why does its name start with a dot?

On Unix-like systems, names beginning with `.` are conventionally hidden. Compare:

| Name | Purpose |
|---|---|
| `.git/` | Local Git metadata and history |
| `.github/` | GitHub-specific configuration, usually committed to Git |
| `.env` | Environment variables; keep secrets out of Git |
| `.gitignore` | Patterns for files Git should ignore |

The `.github` directory is **not** the same as `.git`.

## 7. Create `.github/workflows/` in VS Code

1. Open the repository root in VS Code.
2. Create a folder named `.github`.
3. Inside it, create `workflows`.
4. Create `continuous_integration.yaml` in `workflows`.
5. Paste the workflow YAML, save, and commit it.

From the repository root:

```bash
git add .github/workflows/continuous_integration.yaml
git commit -m "Add GitHub Actions CI pipeline"
git push origin master
```

Replace `master` with `main` if that is your branch. If your `origin` points to a fork, the push updates **your fork**.

## 8. Interview-ready answer

**Question:** What is the purpose of the `.github` folder?

**Answer:** The `.github` directory holds GitHub-specific repository configuration. Its `workflows` subdirectory contains GitHub Actions YAML files that automate tasks such as testing, building, and deployment. The directory can also hold issue templates, pull-request templates, code ownership rules, and Dependabot settings. GitHub Actions workflows must be placed under `.github/workflows/` to be recognized.

## 9. Key takeaway for this MLOps project

Keep the source code and tests under `lec9_CI/`, and put the GitHub Actions workflow under `.github/workflows/`. A push to a configured branch can then trigger a remote Ubuntu runner to execute your pytest tests automatically.
