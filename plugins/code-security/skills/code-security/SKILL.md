---
name: code-security
description: >
  Automatically triggered when running security scans, auditing code for vulnerabilities,
  checking dependencies for CVEs, reviewing code for security issues, or hardening Python
  applications. Applies when discussing: security audit, vulnerability scan, bandit,
  pip-audit, dependency vulnerabilities, OWASP, injection, path traversal, secrets detection,
  insecure deserialization, auth bypass, or "is this code secure?".
  Also triggers on: "run security checks", "find vulnerabilities", "security review",
  "check for CVEs", "audit dependencies", "security lint", "pen test this code",
  "harden this application", "check for secrets", "SAST scan", "code security scan",
  "security pipeline", "supply chain security".
---

# Python Code Security Skill

This skill combines **static analysis tools** with **LLM-powered dynamic analysis** to provide
comprehensive security coverage. Static tools catch known patterns; Claude reasons about logic
flaws, business rule bypasses, and context-dependent vulnerabilities that no static tool can detect.

## Two-Layer Security Model

```
┌─────────────────────────────────────────────────────────┐
│  Layer 1: Static Analysis (Automated)                   │
│  Bandit · pip-audit · ruff S-rules · secrets detection  │
│  → Known vulnerability patterns, CVEs, exposed secrets  │
├─────────────────────────────────────────────────────────┤
│  Layer 2: LLM Dynamic Analysis (Claude)                 │
│  Code path reasoning · business logic review            │
│  → Logic flaws, auth bypasses, race conditions,         │
│    context-dependent vulnerabilities                    │
└─────────────────────────────────────────────────────────┘
```

---

## Layer 1: Static Analysis Tools

### Tool Stack

| Tool | Purpose | Invocation |
|------|---------|------------|
| **Bandit** | Python-specific security linter (SAST) | `uvx bandit -r src/` |
| **pip-audit** | Dependency CVE scanner | `uvx pip-audit` |
| **ruff S-rules** | Bandit rules integrated in ruff | `uv run ruff check --select S src/` |
| **detect-secrets** | Secrets/credentials in source code | `uvx detect-secrets scan` |
| **safety** | Alternative dependency vulnerability check | `uvx safety check` |

### Makefile Targets

Add these targets to the project Makefile:

```makefile
# =============================================================================
# Security
# =============================================================================

# Security lint (Bandit) — high severity only for CI gates
security-lint:
	uvx bandit -r src/ --severity-level high -q

# Full security lint (all severities) — for thorough review
security-lint-full:
	uvx bandit -r src/ -f json -o bandit-report.json || true
	@echo "Full report: bandit-report.json"
	uvx bandit -r src/

# Dependency vulnerability audit
pip-audit:
	uvx pip-audit -r <(uv pip freeze | grep -v -e '^-e ' -e '^$(PROJECT_NAME)==') --strict

# Secrets detection
secrets-scan:
	uvx detect-secrets scan --all-files --exclude-files '\.git/.*' | python -c "import sys,json; r=json.load(sys.stdin); findings=r.get('results',{}); exit(1) if findings else print('No secrets found.')"

# Combined security gate for CI
security-check: security-lint pip-audit secrets-scan
	@echo "All security checks passed!"
```

### Bandit Configuration

Add to `pyproject.toml`:

```toml
[tool.bandit]
exclude_dirs = ["tests", "scripts"]
skips = []          # Don't skip anything by default — be explicit about exceptions

[tool.bandit.assert_used]
skips = ["*/test_*.py", "*_test.py"]
```

### Key Static Analysis Rules

Bandit catches these categories:

| ID | Category | Example |
|----|----------|---------|
| B101 | assert usage | `assert` in production code |
| B102 | exec usage | `exec()` / `eval()` calls |
| B103 | set_bad_file_permissions | `os.chmod(path, 0o777)` |
| B104 | hardcoded_bind_all | Binding to `0.0.0.0` |
| B105-B107 | hardcoded_password | Passwords in source code |
| B108 | hardcoded_tmp_directory | Using `/tmp` directly |
| B301-B303 | pickle / md5 / sha1 | Insecure deserialization/hashing |
| B501-B503 | SSL/TLS issues | `verify=False`, no cert validation |
| B601-B602 | shell injection | `subprocess.call(shell=True)` |
| B608 | SQL injection | String-formatted SQL queries |
| B701 | jinja2 autoescape | Templates without autoescaping |

---

## Layer 2: LLM Dynamic Analysis

This is what makes this skill unique. When triggered, Claude performs **reasoning-based security
analysis** that goes beyond pattern matching.

### What Claude Analyzes (Static Tools Cannot)

#### 1. Authentication & Authorization Logic

- Are there endpoints missing auth decorators/middleware?
- Can a user escalate privileges by manipulating request data?
- Are JWT tokens validated correctly (expiry, issuer, audience)?
- Is there IDOR (Insecure Direct Object Reference) — can user A access user B's resources by guessing IDs?
- Are admin-only operations properly gated?

**How to check:**
```
Review all route/endpoint definitions. For each one:
1. Does it have authentication middleware/decorator?
2. Does it verify the authenticated user owns the requested resource?
3. Are role checks applied for privileged operations?
```

#### 2. Input Validation & Injection

- Are user inputs validated/sanitized before use in queries, commands, file paths?
- Is there path traversal risk? (e.g., `../../etc/passwd` in file parameters)
- Are SQL queries parameterized? (string formatting = injection risk)
- Is user input reflected in responses without escaping? (XSS)
- Are regex patterns vulnerable to ReDoS?

**How to check:**
```
Trace every user input from entry point to usage:
1. HTTP parameters, headers, body fields
2. File uploads (name, content, MIME type)
3. Environment variables from untrusted sources
4. Data from external APIs or databases (second-order injection)
```

#### 3. Path Traversal & File Operations

- Can user-controlled input influence file paths?
- Are symlink attacks possible?
- Is `os.path.join` used with absolute user paths? (it ignores the base!)
- Are temporary files created securely? (`tempfile.mkstemp` vs `/tmp/myfile`)

**How to check:**
```python
# VULNERABLE: os.path.join ignores base when second arg is absolute
path = os.path.join("/safe/dir", user_input)  # user_input = "/etc/passwd" → "/etc/passwd"

# SAFE: resolve and check prefix
base = Path("/safe/dir").resolve()
target = (base / user_input).resolve()
if not str(target).startswith(str(base)):
    raise ValueError("Path traversal detected")
```

#### 4. Secrets & Configuration

- Are secrets hardcoded anywhere? (API keys, passwords, tokens)
- Are `.env` files in `.gitignore`?
- Are secrets logged or included in error messages?
- Is `DEBUG = True` possible in production?
- Are default credentials present?

**How to check:**
```
Search for patterns:
- "password", "secret", "api_key", "token", "credential" in source
- Base64-encoded strings that decode to credentials
- URLs with embedded credentials (postgres://user:pass@host)
- Default values in config that look like real secrets
```

#### 5. Race Conditions & Concurrency

- TOCTOU (Time of Check to Time of Use) vulnerabilities
- Database operations that should be atomic but aren't
- File operations without proper locking
- Session/state manipulation between check and action

**How to check:**
```
Look for patterns:
1. if file_exists(x): read(x)  → file could be replaced between check and read
2. if user.balance >= amount: user.balance -= amount  → without DB transaction lock
3. Check permission, then perform action in separate DB queries
```

#### 6. Cryptographic Issues

- Using MD5/SHA1 for security purposes (only acceptable for checksums)
- Hardcoded IVs or salts
- ECB mode encryption
- Custom crypto implementations (never roll your own)
- Insufficient key lengths

#### 7. Dependency & Supply Chain

- Are dependencies pinned to specific versions?
- Are there known-vulnerable transitive dependencies?
- Is `requirements.txt` used without hashes?
- Are lock files (`uv.lock`, `poetry.lock`) committed?

#### 8. Error Handling & Information Disclosure

- Do error messages expose internal paths, stack traces, or config?
- Are database errors returned directly to users?
- Do 500 errors reveal framework versions?
- Are different error messages returned for "user not found" vs "wrong password"? (user enumeration)

---

## Running a Full Security Audit

When asked to perform a security review, follow this protocol:

### Phase 1: Static Analysis (Automated)

```bash
# Run all static tools
make security-lint-full    # Bandit full report
make pip-audit             # Dependency CVEs
make secrets-scan          # Exposed secrets
uv run ruff check --select S src/  # Ruff security rules
```

### Phase 2: LLM Dynamic Analysis (Claude)

Systematically review these areas using parallel agents where possible:

1. **Entry points audit**: Map all HTTP endpoints, CLI commands, message handlers.
   For each: verify auth, input validation, output encoding.

2. **Data flow tracing**: Follow user input from entry to storage/output.
   Flag any point where input is used unsanitized.

3. **Auth & access control review**: Check every protected resource for IDOR,
   privilege escalation, missing auth checks.

4. **File & path operations**: Find all file I/O and verify paths are constrained.

5. **Crypto & secrets review**: Check for hardcoded secrets, weak algorithms,
   improper key management.

6. **Configuration review**: Check for debug modes, permissive CORS, missing
   security headers.

### Phase 3: Report

Structure findings using severity levels:

| Severity | Description | Example |
|----------|-------------|---------|
| **CRITICAL** | Exploitable now, data breach risk | SQL injection, auth bypass, RCE |
| **HIGH** | Exploitable with some effort | IDOR, path traversal, insecure deserialization |
| **MEDIUM** | Requires specific conditions | CSRF without state-changing impact, info disclosure |
| **LOW** | Best practice violation | Missing security headers, verbose errors in dev |
| **INFO** | Informational finding | Dependency update available, deprecated API usage |

For each finding, provide:
1. **Location**: File, line number, function
2. **Description**: What the vulnerability is
3. **Impact**: What an attacker could achieve
4. **Proof of concept**: How to exploit it (for authorized testing)
5. **Fix**: Exact code change to remediate

---

## CI Integration

Add to the CI pipeline (GitHub Actions example):

```yaml
security:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: astral-sh/setup-uv@v4

    - name: Security lint (Bandit)
      run: uvx bandit -r src/ --severity-level high -q

    - name: Dependency audit
      run: uvx pip-audit -r <(uv pip freeze) --strict

    - name: Secrets scan
      run: |
        uvx detect-secrets scan --all-files --exclude-files '\.git/.*' > /tmp/secrets.json
        python -c "import json; r=json.load(open('/tmp/secrets.json')); exit(1) if r.get('results') else print('Clean')"
```

---

## Security Hardening Checklist

When setting up or reviewing a Python project, verify:

### Application
- [ ] All endpoints have authentication (unless explicitly public)
- [ ] Authorization checks verify resource ownership (no IDOR)
- [ ] All user input is validated with strict schemas (Pydantic, etc.)
- [ ] SQL queries use parameterized statements (never f-strings/format)
- [ ] File paths are resolved and constrained to allowed directories
- [ ] Secrets loaded from environment variables, never hardcoded
- [ ] Error responses don't leak internal details in production
- [ ] CORS is restricted to specific origins (not `*`)
- [ ] Security headers set (HSTS, X-Frame-Options, CSP, etc.)
- [ ] Rate limiting on authentication endpoints

### Dependencies
- [ ] All dependencies pinned with lock file committed
- [ ] `pip-audit` passes with no known CVEs
- [ ] No unnecessary dependencies (reduce attack surface)
- [ ] Lock file is up to date

### Infrastructure
- [ ] `.env` and secret files in `.gitignore`
- [ ] No secrets in Docker layers or build args
- [ ] Non-root user in Dockerfile
- [ ] Debug mode disabled in production config
- [ ] Logging does not include secrets or PII

---

## Common Vulnerability Patterns in Python

### Path Traversal
```python
# VULNERABLE
@app.get("/files/{filename}")
def get_file(filename: str):
    return FileResponse(f"/data/{filename}")  # ../../etc/passwd

# SAFE
@app.get("/files/{filename}")
def get_file(filename: str):
    base = Path("/data").resolve()
    target = (base / filename).resolve()
    if not target.is_relative_to(base):
        raise HTTPException(403, "Access denied")
    return FileResponse(target)
```

### SQL Injection
```python
# VULNERABLE
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# SAFE
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

### Command Injection
```python
# VULNERABLE
os.system(f"convert {user_filename} output.png")

# SAFE
subprocess.run(["convert", user_filename, "output.png"], check=True)
```

### Insecure Deserialization
```python
# VULNERABLE
data = pickle.loads(user_data)  # Arbitrary code execution

# SAFE
data = json.loads(user_data)  # Only data, no code execution
```

### SSRF (Server-Side Request Forgery)
```python
# VULNERABLE
response = requests.get(user_provided_url)  # Can hit internal services

# SAFE
parsed = urlparse(user_provided_url)
if parsed.hostname in ALLOWED_HOSTS:
    response = requests.get(user_provided_url)
```

---

## Anti-Patterns This Skill Prevents

- **No security scanning in CI**: Every project needs at minimum Bandit + pip-audit
- **`# nosec` without justification**: Every Bandit suppression needs a comment explaining why
- **`verify=False` in requests**: Never disable SSL verification, even in dev
- **`shell=True` in subprocess**: Almost never needed — use list arguments
- **Pickle for untrusted data**: Use JSON or msgpack instead
- **String-formatted SQL**: Always use parameterized queries
- **Blanket exception handling**: `except Exception: pass` hides security errors
- **Secrets in source code**: Use environment variables or secret managers
- **Running as root in containers**: Always use a non-root user
- **Static tools only**: Pattern matching misses logic flaws — LLM analysis catches what static tools cannot
