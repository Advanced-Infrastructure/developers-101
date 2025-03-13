# Enforcing Coding Standards with Ruff

## Introduction

[Ruff](https://github.com/astral-sh/ruff) is the fastest Python linter and formatter that consolidates various tools like `pyflakes`, `pyupgrade`, `isort`, `black`, `mccabe`, `bandit`, and more. It helps maintain consistent coding standards, ensures security, and optimizes imports automatically.

## Steps to Integrate Ruff in Your Project

### 1. Install Ruff
You can install Ruff using `pip` or by adding it to your `requirements.txt` file:

```sh
pip install ruff
```

Or, add the following line to `requirements.txt`:

```
ruff==0.9.2
```

### 2. Add `.ruff_cache` to `.gitignore`
The `.ruff_cache` directory stores cache files related to Ruff. To avoid committing unnecessary files, add the following line to your `.gitignore`:

```
.ruff_cache
```

### 3. Configure Ruff
Create a `ruff.toml` file in the root of your project and add the following configuration:

```toml
# Target Python version
[tool.ruff]
target-version = "py313"

# Enable additional linting rules
[lint]
extend-select = [
    "UP",  # pyupgrade
    "I",   # isort
    "C90", # complexity
    "N",   # pep8-naming
    "ASYNC", # flake8-async
    "S",   # flake8-bandit
    "B",   # flake8-bugbear
    "A",   # flake8-builtins
    "C4",  # flake8-comprehensions
]

# Ignore specific rules for test files
[lint.per-file-ignores]
"tests/*.py" = ["S101"]  # Ignore assert statements in tests
```

Refer to the following links for more details on Ruff rules and settings:
- [Ruff Rules](https://docs.astral.sh/ruff/rules)
- [Ruff Settings](https://docs.astral.sh/ruff/settings/#lint_unfixable)
- [Flake8 Rules](https://www.flake8rules.com/)

### 4. Run Ruff Checks
Once Ruff is installed and configured, run the following commands to check and fix linting issues:

```sh
# Check for linting errors
ruff check

# Automatically fix fixable errors
ruff check --fix

# Format code according to standards
ruff format
```

> **ℹ️ Info:** The pre-commit step is WIP. Please don't follow it yet.
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
- repo: https://github.com/astral-sh/ruff-pre-commit
  rev: v0.1.4
  hooks:
    - id: ruff
      args: [ --fix ]
    - id: ruff-format
```

#### Install Pre-Commit Hooks

Run the following command to install the newly added pre-commit hooks:

```sh
pre-commit install
```

### 6. Handling Pre-Commit Failures
If Ruff detects issues during a commit, it will prevent the commit until the issues are fixed. Follow these steps to resolve the errors:

1. Review the errors printed by Ruff.
2. Run `ruff check --fix` to auto-fix issues.
3. Run `ruff format` to format the code.
4. Add the modified files to staging:

   ```sh
   git add .
   ```

5. Retry committing:

   ```sh
   git commit -m "Fix linting issues"
   ```

### 7. Integrate Ruff in IDEs
To ensure code is formatted properly while writing, install the Ruff extension in your IDE:

- **PyCharm Plugin**: [Install from JetBrains Marketplace](https://plugins.jetbrains.com/plugin/20574-ruff)
- **VSCode Extension**: [Install from Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff)

## Conclusion
By integrating Ruff into your workflow, you ensure consistent, optimized, and secure code. Implementing Ruff with `pre-commit` enforces coding standards and prevents unformatted code from being committed. Start using Ruff today to maintain high-quality Python code!

