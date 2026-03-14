---
name: advanced-code-quality
description: >
  Use when going beyond basic linting to enforce advanced Python code quality gates.
  Triggers on: cognitive complexity, code duplication detection, architectural enforcement,
  import boundaries, dependency hygiene, test quality beyond coverage, mutation testing,
  docstring coverage, type coverage, AI-generated code detection, AI slop detection,
  complexipy, jscpd, import-linter, deptry, diff-cover, mutmut, interrogate, sloppylint,
  refurb, pylint design checks, wily, radon maintainability, pytestarch, griffe,
  "bulletproof Python", "tighten quality gates", "advanced quality checks",
  "beyond ruff and mypy", "catch AI-generated code issues", "code duplication",
  "architectural boundaries", "dependency drift", "test effectiveness".
  Complements the code-quality skill (ruff, mypy, xenon, vulture, pre-commit) with 20+ additional tools.
---

# Advanced Python Code Quality

Goes beyond the basic stack (ruff, mypy, xenon, vulture, pre-commit) to cover every remaining quality dimension. Each section maps a gap to specific tools with exact configuration and thresholds.

**Core insight**: AI-generated code doesn't need different tools — it needs **tighter thresholds on existing tools**.

---

## Quick Reference: Tool Inventory

| Gap | Tool | Install | Threshold |
|-----|------|---------|-----------|
| Cognitive complexity | **complexipy** | `pip install complexipy` | ≤10 per function |
| Complexity trends | **wily** | `pip install wily` | No regression vs base branch |
| Maintainability Index | **radon mi** | Already installed (via xenon) | MI ≥ 20 (rank A) |
| Code duplication | **jscpd** | `npm install -g jscpd` | ≤5% duplication |
| Pylint design checks | **pylint** | `pip install pylint` | max-args=6, max-branches=10 |
| Modernization | **refurb** | `pip install refurb` | Zero warnings |
| Custom lint rules | **fixit** | `pip install fixit` | Project-specific |
| Architectural boundaries | **import-linter** | `pip install import-linter` | Zero contract violations |
| Architecture tests | **pytestarch** | `pip install pytestarch` | All tests pass |
| Dependency hygiene | **deptry** | `pip install deptry` | Zero DEP001–DEP005 |
| Dependency conflicts | **pipdeptree** | `pip install pipdeptree` | `--warn fail` |
| Diff coverage | **diff-cover** | `pip install diff-cover` | ≥95% on changed lines |
| Mutation testing | **mutmut** | `pip install mutmut` | ≥75% mutation score |
| Performance regression | **pytest-benchmark** | `pip install pytest-benchmark` | ≤10% regression |
| Docstring coverage | **interrogate** | `pip install interrogate` | ≥95% coverage |
| API breaking changes | **griffe** | `pip install griffe` | Zero breaking changes |
| Type completeness | **pyright** | `pip install pyright` | `--verifytypes` |
| Type bootstrap | **MonkeyType** | `pip install MonkeyType` | N/A (dev tool) |
| AI hallucinated imports | **sloppylint** | `pip install sloppylint` | Zero violations |
| Platform tracking | **SonarCloud** | Free for OSS | Sonar way quality gate |

---

## 1. Cognitive Complexity

Cyclomatic complexity (xenon) counts execution paths. **Cognitive complexity** measures how hard code is for a human to understand — deeply nested loops score much higher than flat switch-style logic even at identical cyclomatic complexity.

### complexipy (Rust, fast, pre-commit ready)

```toml
# pyproject.toml
[tool.complexipy]
paths = ["src"]
max-complexity-allowed = 10
exclude = ["src/generated/**"]
```

```yaml
# .pre-commit-config.yaml
- repo: https://github.com/rohaquinlop/complexipy-pre-commit
  rev: v4.2.0
  hooks:
    - id: complexipy
```

### radon Maintainability Index (already installed via xenon)

Gate at rank A (MI ≥ 20) for new code. Halstead metrics flag functions where estimated bugs exceed 0.5.

### wily — complexity trend tracking

```yaml
# GitHub Actions — fail PR if complexity regresses
- name: Check complexity trend
  run: |
    pip install wily
    wily build src/ -n 20
    wily diff src/ -r origin/${{ github.event.pull_request.base.ref }}
```

---

## 2. Code Duplication

Neither Ruff nor mypy detect copy-pasted code blocks.

### jscpd (recommended for CI)

Token-based (Rabin-Karp), gates on percentage threshold. Exits non-zero when exceeded.

```json
{
  "threshold": 5,
  "minLines": 5,
  "minTokens": 50,
  "mode": "strict",
  "format": ["python"],
  "ignore": ["**/tests/**", "**/migrations/**"],
  "reporters": ["console", "json"],
  "gitignore": true
}
```

### Alternatives

| Tool | Runtime | Algorithm | Threshold | Best for |
|------|---------|-----------|-----------|----------|
| jscpd | Node.js | Rabin-Karp | Percentage (5%) | CI gating with clear thresholds |
| pylint R0801 | Python | Line-based | Min lines (6) | Teams already using pylint |
| CPD (PMD) | Java | Rabin-Karp | Token count (75) | Java-centric CI environments |

Pylint R0801: `min-similarity-lines = 6`, `ignore-imports = true`. Cannot be suppressed inline — whole-project check.

---

## 3. Architectural Enforcement

### import-linter (declarative contracts)

Three contract types: **layers** (one-way deps), **independence** (zero cross-imports), **forbidden** (block specific imports).

```toml
# pyproject.toml
[tool.importlinter]
root_package = "myapp"
include_external_packages = true
exclude_type_checking_imports = true

[[tool.importlinter.contracts]]
id = "clean-arch"
name = "Clean Architecture layers"
type = "layers"
layers = [
    "myapp.infrastructure",
    "myapp.interface",
    "myapp.application",
    "myapp.domain",
]

[[tool.importlinter.contracts]]
id = "domain-purity"
name = "Domain has no framework dependencies"
type = "forbidden"
source_modules = ["myapp.domain"]
forbidden_modules = ["sqlalchemy", "fastapi", "requests", "redis"]

[[tool.importlinter.contracts]]
id = "bounded-contexts"
name = "Bounded contexts are independent"
type = "independence"
modules = ["myapp.domain.orders", "myapp.domain.users", "myapp.domain.payments"]
```

```yaml
# .pre-commit-config.yaml
- repo: https://github.com/seddonym/import-linter
  rev: v2.7
  hooks:
    - id: import-linter
```

### pytestarch — architecture rules as tests

```python
from pytestarch import Rule, EvaluableArchitecture

def test_domain_isolation(evaluable: EvaluableArchitecture):
    rule = (
        Rule()
        .modules_that()
        .are_sub_modules_of("src.myapp.domain")
        .should_not()
        .import_modules_that()
        .are_sub_modules_of("src.myapp.infrastructure")
    )
    rule.assert_applies(evaluable)
```

**pytest-archon** alternative: `archrule("domain").match("myapp.domain*").should_not_import("myapp.infrastructure*").check("myapp")`

---

## 4. Dependency Hygiene

### deptry (Rust-powered)

Cross-references imports against declared dependencies:
- **DEP001**: Import of undeclared package
- **DEP002**: Declared but never imported (bloat)
- **DEP003**: Transitive dependency used without declaring
- **DEP004**: Dev dependency used in production code

```toml
# pyproject.toml
[tool.deptry]
exclude = ["tests", "docs", "migrations"]

[tool.deptry.per_rule_ignores]
DEP002 = ["celery", "gunicorn"]  # Runtime-only, not imported
DEP004 = ["pytest"]
```

```yaml
# .pre-commit-config.yaml
- repo: https://github.com/fpgmaas/deptry.git
  rev: "0.24.0"
  hooks:
    - id: deptry
```

### pipdeptree — dependency conflicts

```bash
pipdeptree --warn fail  # Catch conflicting versions
```

---

## 5. Test Quality Gates

### diff-cover — PR-level coverage gating

New code should have near-complete coverage even if the overall codebase doesn't.

```yaml
# GitHub Actions
- run: pytest --cov=src --cov-branch --cov-report=xml --cov-fail-under=80
- run: diff-cover coverage.xml --compare-branch=origin/main --fail-under=95
```

### mutmut — mutation testing

Mutates source code (flips operators, changes constants, modifies returns) and reruns tests. Surviving mutants = weak tests.

```bash
mutmut run --paths-to-mutate src/core/
# Gate at ≥75% mutation score for critical modules
```

mutmut 3+ uses `fork()` for speed. Set `mutate_only_covered_lines = true` to skip untested code.

### Additional pytest plugins

- **pytest-benchmark**: `--benchmark-compare-fail=mean:10%` fails CI on performance regression
- **pytest-randomly**: Randomizes test order to catch hidden inter-test dependencies
- **pytest-deadfixtures**: Detects unused fixtures
- **hypothesis**: Property-based testing — systematically generates edge cases

---

## 6. Pylint Design Checks & Refurb

Ruff implements ~209 of pylint's ~409 rules. Missing: type-inference checks, cross-module analysis, design checker suite.

### Pylint design thresholds (tighter than defaults)

```toml
[tool.pylint.design]
max-args = 6              # Default: 5
max-positional-arguments = 4
max-locals = 12           # Default: 15
max-returns = 5           # Default: 6
max-branches = 10         # Default: 12
max-statements = 40       # Default: 50
max-bool-expr = 4         # Default: 5
max-parents = 5           # Default: 7
```

### refurb — modernization beyond Ruff

Catches idioms Ruff hasn't ported: `Path(x).read_text()` over `open(x).read()`, tuple membership over chained `or`, `list.clear()` over `del x[:]`.

```yaml
# .pre-commit-config.yaml
- repo: https://github.com/dosisod/refurb
  rev: v2.0.0
  hooks:
    - id: refurb
```

### fixit (Meta) — custom lint rules with auto-fixes

Built on LibCST. Use for project-specific codemods: enforcing API patterns, migrating deprecated internal calls.

### SonarQube/SonarCloud

Comprehensive platform: cognitive complexity + duplication + tech debt + framework-aware rules (Django, FastAPI, Flask, NumPy, Pandas). Imports reports from Ruff, pylint, Bandit, mypy. Free for OSS via SonarCloud.

---

## 7. Documentation & Type Coverage

### interrogate — docstring coverage

```toml
# pyproject.toml
[tool.interrogate]
ignore-init-method = true
ignore-magic = true
ignore-module = true
fail-under = 95
style = "google"
exclude = ["tests", "docs"]
```

```yaml
# .pre-commit-config.yaml
- repo: https://github.com/econchick/interrogate
  rev: 1.7.0
  hooks:
    - id: interrogate
      args: [--quiet, --fail-under=95]
```

### Ruff docstring rules

- **D rules**: Docstring style enforcement (replaces deprecated pydocstyle)
- **DOC rules** (preview): Docstring-to-signature consistency. Enable `DOC202`, `DOC402`, `DOC403`, `DOC501` with `preview = true`

### griffe — API breaking change detection

```bash
griffe check mypackage --against pypi  # Block releases with accidental API breaks
```

### Type coverage

- **pyright**: Faster than mypy, `--verifytypes` for type completeness
- **mypy `--any-exprs-report`**: Percentage of expressions with known types vs `Any`. Gate at 85%
- **MonkeyType**: Runtime tracing to bootstrap annotations — `monkeytype run -m pytest` then `monkeytype apply module_name`

---

## 8. AI-Generated Code Detection

Research: 5.2% of commercial LLM outputs contain hallucinated package imports (USENIX Security 2025, 576K samples). Code cloning rose from 8.3% to 12.3% in AI-heavy codebases (GitClear).

### sloppylint

Detects hallucinated imports (4-layer resolution: builtins → stdlib → installed → importlib), cross-language leakage (`.push()` in Python), placeholder code, mutable defaults.

```bash
pip install sloppylint
sloppylint src/
```

### Tightened thresholds for AI-heavy codebases

Compose existing tools with stricter settings:

- **complexipy** threshold **8** (half the default) — catches nested-logic tendency
- **jscpd** threshold **3%** — catches copy-paste patterns
- **pylint max-statements=30** — catches bloated functions
- **Ruff B006** — mutable default arguments (top AI mistake)
- **Ruff SIM** — overly complex patterns AI generates
- **Vulture** — dead code AI generates but never calls
- **deptry DEP001** — imports of undeclared packages
- **pip-audit** — slopsquatting attacks via hallucinated packages

---

## 9. Integration Pattern

### Pre-commit: staged by speed

```yaml
# .pre-commit-config.yaml
default_install_hook_types: [pre-commit, commit-msg, pre-push]
fail_fast: true

repos:
  # ~0.1s — Formatting
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.15.6
    hooks:
      - id: ruff-check
        args: [--fix, --exit-non-zero-on-fix]
      - id: ruff-format

  # ~0.2s — Quick validation
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: check-yaml
      - id: check-toml
      - id: check-added-large-files
      - id: debug-statements

  # ~0.3s — Cognitive complexity
  - repo: https://github.com/rohaquinlop/complexipy-pre-commit
    rev: v4.2.0
    hooks:
      - id: complexipy

  # ~0.3s — Docstring coverage
  - repo: https://github.com/econchick/interrogate
    rev: 1.7.0
    hooks:
      - id: interrogate
        args: [--quiet, --fail-under=95]

  # ~0.5s — Dependency hygiene
  - repo: https://github.com/fpgmaas/deptry.git
    rev: "0.24.0"
    hooks:
      - id: deptry

  # ~0.5s — Architectural boundaries
  - repo: https://github.com/seddonym/import-linter
    rev: v2.7
    hooks:
      - id: import-linter

  # ~0.5s — Secrets
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.22.1
    hooks:
      - id: gitleaks

  # ~0.5s — Modernization
  - repo: https://github.com/dosisod/refurb
    rev: v2.0.0
    hooks:
      - id: refurb

  # ~2-10s — Type checking (pre-push only)
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.14.0
    hooks:
      - id: mypy
        stages: [pre-push]
        require_serial: true
        pass_filenames: false
        additional_dependencies: [types-requests, types-PyYAML]

  # Conventional commits
  - repo: https://github.com/compilerla/conventional-pre-commit
    rev: v3.6.0
    hooks:
      - id: conventional-pre-commit
        stages: [commit-msg]
```

### CI: parallel jobs with quality gate

```yaml
jobs:
  pre-commit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
      - uses: pre-commit/action@v3.0.1

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -e ".[dev]"
      - run: mypy --strict src/

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - run: pip install -e ".[dev]"
      - run: pytest --cov=src --cov-branch --cov-report=xml --cov-fail-under=80
      - run: diff-cover coverage.xml --compare-branch=origin/main --fail-under=95

  duplication:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npx jscpd src/ --threshold 5 --format python

  quality-gate:
    needs: [pre-commit, typecheck, test, duplication]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - run: |
          for result in "${{ needs.pre-commit.result }}" "${{ needs.typecheck.result }}" \
                        "${{ needs.test.result }}" "${{ needs.duplication.result }}"; do
            [[ "$result" != "success" ]] && echo "Gate failed" && exit 1
          done
```

**Key principles**: `pyproject.toml` as single source of truth, cache pre-commit and mypy/ruff caches in CI, `--output-format=github` in Ruff for PR annotations, keep pre-commit under 5 seconds total.

---

## Anti-Patterns

- **Skipping cognitive complexity because xenon "already covers it"** — they measure different things
- **Running all tools in pre-commit** — stage slow tools to `pre-push` or CI-only
- **No diff-cover** — overall coverage hides untested new code
- **No architectural enforcement** — dependency violations are the most expensive defects
- **Same thresholds for AI-heavy codebases** — tighten them (complexity 8, duplication 3%, statements 30)
- **Ignoring deptry** — transitive dependency reliance is a ticking time bomb
- **No mutation testing on critical paths** — coverage ≠ test quality
