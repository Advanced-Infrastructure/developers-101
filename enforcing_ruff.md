# Enforcing Coding Standards with Ruff

## Introduction

[Ruff](https://github.com/astral-sh/ruff) is the fastest Python linter and formatter that consolidates various tools like `pyflakes`, `pyupgrade`, `isort`, `black`, `mccabe`, `bandit`, and more. It helps maintain consistent coding standards, ensures security, and optimizes imports automatically.

## Steps to Integrate Ruff in Your Project

### 1. Install Ruff
You can install Ruff using `pip` or by adding it to your `requirements.txt` file:

```sh
pip install ruff
```

### 2. Add `.ruff_cache` to `.gitignore`
The `.ruff_cache` directory stores cache files related to Ruff. To avoid committing unnecessary files, add the following line to your `.gitignore`:

```
.ruff_cache
```

### 3. Configure Ruff
Create a `ruff.toml` file in the root of your project and add the following configuration:

```toml
# find default setting at: https://docs.astral.sh/ruff/configuration/

# Exclude a variety of commonly ignored directories.
exclude = [
    ".bzr",
    ".direnv",
    ".eggs",
    ".git",
    ".git-rewrite",
    ".hg",
    ".ipynb_checkpoints",
    ".mypy_cache",
    ".nox",
    ".pants.d",
    ".pyenv",
    ".pytest_cache",
    ".pytype",
    ".ruff_cache",
    ".svn",
    ".tox",
    ".venv",
    ".vscode",
    "__pypackages__",
    "_build",
    "buck-out",
    "build",
    "dist",
    "node_modules",
    "site-packages",
    "venv",
    "test",
    "tests",
    "scripts",
    "test_flow",
    "common_utils",
    "common-utils",
    "infra_access",
    "ogr2ogr.py",
]

# Same as Black.
line-length = 88
indent-width = 4

# Assume Python 3.13
target-version = "py313"

[lint]
# Enable Pyflakes (`F`) and a subset of the pycodestyle (`E`) codes by default.
# Unlike Flake8, Ruff doesn't enable pycodestyle warnings (`W`) or
# McCabe complexity (`C901`) by default.
select = ["E4", "E7", "E9", "F"]
extend-select = [
    "UP",  # pyupgrade
    "I", # isort
    "N", # pep8-naming
    "ASYNC", # flake8-async
    "S", # flake8-bandit
    "B", # flake8-bugbear
    "A", # flake8-builtins
    "C4", # flake8-comprehensions
    "PTH", # flake8-use-pathlib
    "DTZ", # flake8-datetimez
]
ignore = []

# Allow fix for all enabled rules (when `--fix`) is provided.
fixable = ["ALL"]
unfixable = []

# Allow unused variables when underscore-prefixed.
dummy-variable-rgx = "^(_+|(_+[a-zA-Z0-9_]*[a-zA-Z0-9]+?))$"

[format]
# Like Black, use double quotes for strings.
quote-style = "double"

# Like Black, indent with spaces, rather than tabs.
indent-style = "space"

# Like Black, respect magic trailing commas.
skip-magic-trailing-comma = false

# Like Black, automatically detect the appropriate line ending.
line-ending = "auto"

# Enable auto-formatting of code examples in docstrings. Markdown,
# reStructuredText code/literal blocks and doctests are all supported.
#
# This is currently disabled by default, but it is planned for this
# to be opt-out in the future.
docstring-code-format = false

# Set the line length limit used when formatting code snippets in
# docstrings.
#
# This only has an effect when the `docstring-code-format` setting is
# enabled.
docstring-code-line-length = "dynamic"
```

Refer to the following links for more details on Ruff rules and settings:
- [Ruff Rules](https://docs.astral.sh/ruff/rules)
- [Ruff Settings](https://docs.astral.sh/ruff/settings/#lint_unfixable)
- [Flake8 Rules](https://www.flake8rules.com/)

### 4. Run Ruff Checks
Once Ruff is installed and configured, run the following commands to check and fix linting issues:

```sh
# Check for linting errors
ruff check .

# Automatically fix fixable errors
ruff check . --fix

# check for a specific rule (say B: https://docs.astral.sh/ruff/rules/#flake8-bugbear-b)
ruff check . --select="B"

# add a no quality assesment check flag to a line (use only if unavoidable)
# use only for a specific file and a specific rule
ruff check insert/file/path/ --select="B002" --add-noqa

# Format code according to standards
ruff format
```

### 5. Enforce Ruff in Pre-Commit Hooks
To prevent committing unformatted or non-linted code, integrate Ruff with `pre-commit`.

#### Install `pre-commit`

If you haven't installed `pre-commit`, install it using:

```sh
pip install pre-commit
```

#### Update `.pre-commit-config.yaml`

Add the following configuration to your `.pre-commit-config.yaml` file:

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    # Ruff version.
    rev: v0.12.7
    hooks:
      # Run the linter.
      - id: ruff-check
      # Run the formatter.
      - id: ruff-format
        args: [--check]
```

#### Install Pre-Commit Hooks

Run the following command to install the newly added pre-commit hooks:

```sh
pre-commit install
```

### 6. Handling Pre-Commit Failures
If pre-commit detects that ruff check and ruff format are not done during a commit, it will prevent the commit until the issues are fixed. Fix the ruff linting and formatting violations and try committing again.

You can check check if all pre-commit hooks are satisfied even before making the commit by running:
```sh
pre-commit run
```

### 7. Integrate Ruff in IDEs
To ensure code is formatted properly while writing, install the Ruff extension in your IDE:

- **PyCharm Plugin**: [Install from JetBrains Marketplace](https://plugins.jetbrains.com/plugin/20574-ruff)
- **VSCode Extension**: [Install from Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff)

## Conclusion
By integrating Ruff into your workflow, you ensure consistent, optimized, and secure code. Implementing Ruff with `pre-commit` enforces coding standards and prevents unformatted code from being committed. Start using Ruff today to maintain high-quality Python code!

