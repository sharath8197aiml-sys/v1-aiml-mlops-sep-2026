# Pytest Quick Reference — `basic_pytest_demo`

## Project structure

```text
lec9_CI/
└── basic_pytest_demo/
    ├── base_file.py
    ├── use_base_file.py
    ├── test_base_file.py
    └── PYTEST_GUIDE_FIXED.md
```

- `base_file.py`: defines `add`, `subtract`, `multiply`, and `divide`.
- `use_base_file.py`: imports and prints `a` and `b`.
- `test_base_file.py`: contains four pytest test functions and thirteen assertions.

## 1. Activate the Conda environment

Open **VS Code Terminal** or **Command Prompt**:

```cmd
conda activate agents
```

## 2. Navigate to the folder

In Windows Command Prompt:

```cmd
cd /d D:\MLOPS_Shivam_prasad\MLOPS_Github_Code\v1-aiml-mlops-sep-2026\lec9_CI\basic_pytest_demo
```

In PowerShell, use `cd` with the same path (without `/d`).

## 3. Run the tests

```cmd
python -m pytest test_base_file.py -v
```

Expected test results (assuming the uploaded code is unchanged):

```text
test_base_file.py::test_add PASSED
test_base_file.py::test_subtract PASSED
test_base_file.py::test_multiply PASSED
test_base_file.py::test_divide PASSED

================ 4 passed ================
```

Pytest counts **four test functions**, even though those functions contain **thirteen assertions**.

## 4. Useful commands

| Goal | Command |
|---|---|
| Run all tests in current directory | `python -m pytest -v` |
| Run only this file | `python -m pytest test_base_file.py -v` |
| Run addition test | `python -m pytest test_base_file.py::test_add -v` |
| Run division test | `python -m pytest test_base_file.py::test_divide -v` |
| Show `print()` output | `python -m pytest test_base_file.py -v -s` |
| Stop on first failure | `python -m pytest test_base_file.py -x` |
| Show pytest version | `python -m pytest --version` |

## 5. Run from the parent directory

From `lec9_CI`:

```cmd
python -m pytest basic_pytest_demo/test_base_file.py -v
```

This targets the basic demo without collecting unrelated tests such as `test_flask.py`.

## 6. How pytest works

1. Pytest discovers `test_base_file.py` because its name begins with `test_`.
2. It imports the functions from `base_file.py`.
3. It discovers `test_add`, `test_subtract`, `test_multiply`, and `test_divide`.
4. It evaluates each `assert` statement and reports pass/fail per test function.

### Example

```python
from base_file import add

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
```

This is **one test with three assertions**.

## 7. Import behavior in `base_file.py`

Your `base_file.py` includes a `print()` statement at module level. That statement runs when pytest imports the module. Code inside this guard runs only when the file is executed directly:

```python
if __name__ == "__main__":
    print("Running directly")
```

Use `-s` if you want to see output that pytest would normally capture.

## 8. Troubleshooting

### `pytest` is not recognized

```cmd
conda activate agents
python -m pip install pytest
python -m pytest --version
```

Prefer `python -m pytest` to use pytest from the active Python interpreter.

### `ModuleNotFoundError: No module named 'base_file'`

Run pytest from the `basic_pytest_demo` directory, or verify that the project imports and Python paths are configured correctly.

### Unrelated `test_flask.py` fails during collection

Run only the demo test file:

```cmd
python -m pytest basic_pytest_demo/test_base_file.py -v
```

The Flask test's `classifier.pkl` error is a separate issue.

### `collected 0 items`

Check that your test filename starts with `test_` and your test functions start with `test_`.

## 9. Connection to CI/CD

The same command can be used in a GitHub Actions job after installing dependencies:

```bash
python -m pytest basic_pytest_demo/test_base_file.py -v
```

This helps automatically check your code after pushes or pull requests.

## 10. View this Markdown file in VS Code

1. Open the `.md` file in the **Text Editor**.
2. Confirm you see the `# Pytest Quick Reference` heading.
3. Press **Ctrl + Shift + V** to open Markdown Preview.
4. In Windows Command Prompt, verify file contents with `type PYTEST_GUIDE_FIXED.md`.
