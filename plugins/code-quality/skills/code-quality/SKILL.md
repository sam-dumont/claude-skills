---
name: code-quality
description: >
  Automatically triggered when setting up Python code quality tooling, configuring linters,
  adding type checking, improving code standards, running code quality checks, setting up
  pre-commit hooks, or auditing Python code for quality issues. Applies when discussing:
  ruff setup, mypy configuration, code formatting, linting rules, cyclomatic complexity,
  dead code detection, file length limits, pre-commit hooks, Makefile targets for quality,
  uv-based workflows, pyproject.toml configuration, or "check my code quality".
  Also triggers on: "set up linting", "add type checking", "configure ruff", "format my code",
  "run quality checks", "set up pre-commit", "code smells", "clean up this code",
  "add code quality", "quality gate", "CI quality pipeline".
---

# Python Code Quality Skill

This skill sets up and enforces comprehensive Python code quality using a battle-tested
toolchain. Based on real production Makefiles using `uv` for fast dependency management.

## Philosophy

- **Fast feedback**: Use `uv run` and `uvx` for instant tool execution — no global installs
- **Layered checks**: Lint → Format → Typecheck → Complexity → Dead Code → File Length
- **CI-ready**: Every check is a Makefile target that returns non-zero on failure
- **Opinionated defaults**: Start strict, relax only with justification

---

## Tool Stack

| Tool | Purpose | Config Location |
|------|---------|-----------------|
| **ruff** | Linting + formatting (replaces flake8, isort, black) | `pyproject.toml` |
| **mypy** | Static type checking | `pyproject.toml` |
| **xenon** | Cyclomatic complexity gating | CLI flags |
| **vulture** | Dead code detection | CLI flags |
| **pre-commit** | Git hook automation | `.pre-commit-config.yaml` |

---

## Setup: pyproject.toml Configuration

When setting up code quality for a Python project, add these sections to `pyproject.toml`:

```toml
# =============================================================================
# Ruff — Linting & Formatting
# =============================================================================
[tool.ruff]
target-version = "py312"           # Adjust to project's minimum Python version
line-length = 120
src = ["src", "tests"]

[tool.ruff.lint]
select = [
    "E",      # pycodestyle errors
    "W",      # pycodestyle warnings
    "F",      # pyflakes
    "I",      # isort (import sorting)
    "N",      # pep8-naming
    "UP",     # pyupgrade
    "B",      # flake8-bugbear
    "SIM",    # flake8-simplify
    "S",      # flake8-bandit (security)
    "A",      # flake8-builtins
    "C4",     # flake8-comprehensions
    "DTZ",    # flake8-datetimez
    "T20",    # flake8-print
    "PT",     # flake8-pytest-style
    "RET",    # flake8-return
    "PTH",    # flake8-use-pathlib
    "ERA",    # eradicate (commented-out code)
    "PL",     # pylint subset
    "RUF",    # ruff-specific rules
]
ignore = [
    "S101",   # assert usage (fine in tests)
    "PLR0913", # too many arguments (relax for data-heavy functions)
]

[tool.ruff.lint.per-file-ignores]
"tests/**/*.py" = ["S101", "PLR2004", "T20"]  # Allow asserts, magic values, prints in tests

[tool.ruff.lint.isort]
known-first-party = ["PROJECT_NAME"]   # Replace with actual package name

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
line-ending = "lf"

# =============================================================================
# Mypy — Type Checking
# =============================================================================
[tool.mypy]
python_version = "3.12"               # Adjust to project's minimum Python version
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
strict_equality = true
warn_redundant_casts = true
warn_unused_ignores = true
no_implicit_reexport = true

[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false          # Relax for test functions

# =============================================================================
# Pytest
# =============================================================================
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --strict-markers"
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
]
```

### Key Decisions

- **Line length 120**: 80 is too restrictive for modern screens; 120 balances readability and density
- **`select` not `extend-select`**: Explicit about exactly which rules are active
- **`S` rules included**: Ruff's built-in bandit subset catches common security issues during linting
- **Strict mypy**: `disallow_untyped_defs` forces type annotations — relax per-module if needed
- **Tests relaxed**: Asserts, magic values, and print statements are fine in test code

---

## Setup: Dev Dependencies

Add to `pyproject.toml` under dev dependencies:

```toml
[project.optional-dependencies]
dev = [
    "ruff>=0.8",
    "mypy>=1.13",
    "pytest>=8.0",
    "pytest-cov>=6.0",
    "pre-commit>=4.0",
]
```

Or with uv groups:

```toml
[dependency-groups]
dev = [
    "ruff>=0.8",
    "mypy>=1.13",
    "pytest>=8.0",
    "pytest-cov>=6.0",
    "pre-commit>=4.0",
]
```

Tools that run via `uvx` (no install needed): `xenon`, `vulture`.

---

## Setup: Makefile Targets

Add these targets to the project Makefile:

```makefile
# =============================================================================
# Code Quality
# =============================================================================

lint:
	uv run ruff check src tests

lint-fix:
	uv run ruff check --fix src tests

format:
	uv run ruff format src tests

format-check:
	uv run ruff format --check src tests

typecheck:
	uv run mypy src/PROJECT_NAME

# File length gate (max 500 lines per .py file)
MAX_LINES := 500
file-length:
	@FAILED=0; \
	for f in $$(find src/ -name '*.py'); do \
		count=$$(wc -l < "$$f"); \
		if [ "$$count" -gt $(MAX_LINES) ]; then \
			echo "ERROR: $$f has $$count lines (max $(MAX_LINES))"; \
			FAILED=1; \
		fi; \
	done; \
	if [ "$$FAILED" -eq 1 ]; then \
		echo ""; \
		echo "Files exceeding $(MAX_LINES) lines must be split into smaller modules."; \
		exit 1; \
	fi; \
	echo "All files under $(MAX_LINES) lines."

# Cyclomatic complexity gate (Xenon grade C)
complexity:
	cd /tmp && uvx xenon --max-absolute C --max-modules D --max-average C $(CURDIR)/src/

# Dead code detection
dead-code:
	uvx vulture src/ --min-confidence 90

# Ensure dev dependencies are installed
ensure-dev:
	@uv sync --all-extras --quiet

# Run all checks (same as CI)
check: ensure-dev lint format-check typecheck file-length complexity test
	@echo "All checks passed!"

# Full CI-equivalent pipeline (locally)
ci: ensure-dev lint format-check typecheck file-length complexity dead-code test
	@echo "Full CI pipeline passed!"
```

---

## Setup: Pre-commit Hooks

Create `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.6    # Pin to a specific version
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.13.0
    hooks:
      - id: mypy
        additional_dependencies: []  # Add stubs your project needs

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
        args: ['--maxkb=500']
      - id: check-merge-conflict
      - id: debug-statements
```

Install hooks:

```bash
uv run pre-commit install
```

---

## Running Checks

### Quick Check (During Development)

```bash
make lint format-check typecheck
```

### Full Quality Gate (Before Committing)

```bash
make check
```

This runs: `lint → format-check → typecheck → file-length → complexity → test`

### Full CI Pipeline (Before Pushing)

```bash
make ci
```

Adds: `dead-code` to the check pipeline.

---

## When Reviewing Existing Code

When asked to review or improve code quality in an existing project, follow this order:

### 1. Assess Current State

```bash
# Check if quality tools are configured
cat pyproject.toml | grep -A5 'tool.ruff'
cat pyproject.toml | grep -A5 'tool.mypy'
ls .pre-commit-config.yaml
ls Makefile
```

### 2. Run Existing Checks (If Available)

```bash
make lint 2>/dev/null || uv run ruff check src/
make typecheck 2>/dev/null || uv run mypy src/
```

### 3. Report Findings

Structure findings as:

- **Critical**: Type errors, undefined names, security issues (ruff S rules)
- **Warning**: Complexity issues, dead code, long files
- **Style**: Formatting, import ordering, naming conventions

### 4. Fix Incrementally

- Fix critical issues first
- Auto-fix what ruff can handle: `make lint-fix`
- Format: `make format`
- Address type errors manually
- Split long files if over 500 lines

---

## Complexity Thresholds

| Metric | Tool | Threshold | Action |
|--------|------|-----------|--------|
| Cyclomatic complexity | xenon | Grade C (max per function) | Refactor functions with complexity > 10 |
| Module complexity | xenon | Grade D (max per module) | Split modules that are too complex |
| Average complexity | xenon | Grade C (project average) | Overall project health indicator |
| File length | wc -l | 500 lines | Split into submodules |
| Dead code confidence | vulture | 90% | Investigate and remove confirmed dead code |

---

## Common Ruff Rule Groups Explained

| Code | Name | What It Catches |
|------|------|-----------------|
| E/W | pycodestyle | Basic style violations |
| F | pyflakes | Unused imports, undefined names |
| I | isort | Import ordering |
| N | pep8-naming | Naming convention violations |
| UP | pyupgrade | Python version upgrade opportunities |
| B | flake8-bugbear | Common bugs and design problems |
| SIM | flake8-simplify | Code that can be simplified |
| S | flake8-bandit | Security issues |
| C4 | flake8-comprehensions | Unnecessary list/dict/set calls |
| PTH | flake8-use-pathlib | os.path → pathlib suggestions |
| ERA | eradicate | Commented-out code |
| PL | pylint | Subset of pylint checks |
| RUF | ruff-specific | Ruff's own rules |

---

## Anti-Patterns This Skill Prevents

- **No linting configured**: Every Python project must have ruff
- **Black + isort + flake8 separately**: Use ruff — it replaces all three, 10-100x faster
- **Mypy with `--ignore-missing-imports` everywhere**: Fix the imports, add stubs
- **No complexity gates**: Functions grow unbounded without xenon checks
- **500+ line files**: Sign of poor module decomposition — split them
- **Dead code accumulation**: Vulture catches unused functions/classes
- **Manual formatting**: Pre-commit hooks automate this on every commit
- **Global tool installs**: Use `uv run` and `uvx` — no system pollution

---

## Adapting to Existing Projects

When a project already has some quality tooling:

1. **Don't replace working configs** — extend them
2. **Migrate incrementally**: If using black+isort+flake8, migrate to ruff one tool at a time
3. **Start mypy in lenient mode** if not yet typed: `disallow_untyped_defs = false`, then tighten
4. **Add `# type: ignore[specific-error]`** for known issues, never blanket ignores
5. **Set `--min-confidence 90`** for vulture to reduce false positives in new projects
