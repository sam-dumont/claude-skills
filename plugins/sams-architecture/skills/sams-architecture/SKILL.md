---
name: sams-architecture
description: >
  Automatically triggered when designing system architecture, planning new projects,
  creating implementation plans, conducting architecture reviews, or auditing existing
  codebases for consistency. Applies when discussing: system design, technology choices,
  architectural patterns, infrastructure decisions, API design, testing strategies,
  CI/CD pipelines, security architecture, or code organization. Works for Python web APIs,
  infrastructure/DevOps, Garmin/embedded systems, and frontend projects. Distinguishes
  between personal projects (strict standards) and client work (adaptive approach).
---

# Sam's Architecture Standards

This skill codifies Sam's mature architectural patterns developed across 31+ personal projects and several client projects. It automatically applies during project planning, architecture reviews, and codebase audits.

---

## CRITICAL: Zero-Tolerance Anti-Patterns

These are **NEVER acceptable** in Sam's personal projects. Flag immediately if detected:

### Over-Engineering Anti-Patterns

❌ **Premature Microservices**: Breaking into microservices before you understand the domain boundaries
  - Sam's rule: Start monolithic with clear module boundaries, split only when you have operational pain
  - Example: Don't create separate services for "user-service", "auth-service", "notification-service" in a new project

❌ **YAGNI Violations**: Building features "because we might need them later"
  - Sam's rule: Build what you need now, refactor when you need more
  - Example: Don't add multi-tenancy support to a single-user API

❌ **Abstraction Astronauts**: Creating generic frameworks when specific solutions work
  - Sam's rule: Three uses before abstracting. Copy-paste twice, abstract on third use
  - Example: Don't create a "BaseAPIClient" with complex inheritance when you have one API client

❌ **Configuration Complexity**: Making everything configurable "for flexibility"
  - Sam's rule: Hard-code sensible defaults, make configurable only when you need to change it
  - Example: Don't create config files for things that never change per environment

### Under-Engineering Anti-Patterns

❌ **No Tests**: "We'll add tests later" (narrator: they never did)
  - Sam's rule: 80% coverage minimum on personal projects, tests written with code

❌ **Missing CI/CD**: Manual deployments, no automation
  - Sam's rule: CI/CD setup is part of project initialization, not a "nice to have"

❌ **Security as Afterthought**: No authentication, secrets in code, no input validation
  - Sam's rule: Security is non-negotiable from day one

❌ **No Documentation**: "The code is self-documenting"
  - Sam's rule: CLAUDE.md + README.md are mandatory for all personal projects

### Common Bad Practices

❌ **Secrets in Code**: API keys, passwords, tokens committed to git
  - Sam's rule: Environment variables + .env.example template, use Snyk scanning

❌ **Missing Error Handling**: No try/catch, no logging, silent failures
  - Sam's rule: Explicit error handling, structured logging, monitoring

❌ **No Health Checks**: Can't monitor if service is running
  - Sam's rule: /health endpoint on all APIs, liveness/readiness probes in K8s

❌ **Root in Docker**: Running containers as root user
  - Sam's rule: Always create and use non-root user in Dockerfiles

❌ **Manual Database Changes**: Running SQL directly in production
  - Sam's rule: Migrations tracked in code (Alembic for Python), reviewable, replayable

❌ **No Input Validation**: Trusting user input
  - Sam's rule: Pydantic models for Python APIs, validate at boundaries

### Technology Cargo-Culting

❌ **Using Tech Because It's Trendy**: "Let's use GraphQL/gRPC/MongoDB because everyone else does"
  - Sam's rule: Choose tech based on problem requirements, not hype
  - Default stack works for 95% of projects: FastAPI + PostgreSQL + REST

❌ **NoSQL When You Need SQL**: Using MongoDB for relational data
  - Sam's rule: Use PostgreSQL by default, only use NoSQL when you have specific needs (e.g., document storage, time-series)

❌ **Microservices Before Monolith**: Starting with distributed systems
  - Sam's rule: Monolith first, split when you have evidence it's needed

---

## Context Detection: What Type of Project Is This?

Before applying standards, determine the context:

### 1. Personal vs. Client Project

**Personal Projects** (Strict Standards):
- Full architectural control
- 80% test coverage mandatory
- Comprehensive CI/CD required
- CLAUDE.md + README.md mandatory
- Security zero-tolerance
- Examples: ev-consumption-api, id7-car-history, musicgen-api

**Client Projects** (Adaptive Standards):
- Understand their constraints and existing patterns
- Adapt recommendations to their context
- Document deviations in CLAUDE.md
- Three ownership levels (see below)

### 2. Client Project Ownership Levels

**Full Architectural Control** (e.g., Darefore/Datafield, Darefore/movesense-firmware):
- Apply Sam's patterns fully
- Establish best practices from start
- Comprehensive documentation
- Full testing and CI/CD

**Collaborative** (e.g., Stryd/Stryd-Zones):
- Align with existing team patterns
- Suggest improvements diplomatically
- Document shared decisions
- Respect team conventions

**Support Role** (e.g., microoled/activelook-garmin-app):
- Minimal architectural influence
- Focus on bug fixes and requested features
- Follow existing patterns strictly
- Don't introduce major changes

### 3. Project Maturity Level

**New Projects** (< 10 commits):
- Can start lean but with clear upgrade path
- Set up CI/CD infrastructure early
- Document architectural decisions from start
- Tests written alongside code

**Mature Projects** (50+ commits, deployed):
- Full standards apply
- Audit for compliance
- Refactor toward patterns if needed
- Comprehensive documentation expected

---

## Python Web API Architecture

This is Sam's primary stack. These standards are **MANDATORY** for all personal projects.

### Framework Choice

**Default: FastAPI for all new projects**

Rationale:
- Modern async/await patterns for better performance
- Automatic OpenAPI documentation (no Swagger setup)
- Pydantic integration for validation and serialization
- Type hints everywhere improve maintainability
- Faster than Flask for I/O-bound operations

**Flask Migration Policy**:
- Existing Flask projects stay Flask unless:
  - Major refactor is happening anyway
  - Performance is a documented problem
  - Adding async operations is needed
- Don't rewrite working Flask apps just to use FastAPI

**Example**: ev-consumption-api, id7-car-history, musicgen-api all use FastAPI

### Architectural Pattern: Service→Repository→Database

**MANDATORY** three-layer architecture:

```
HTTP Request
    ↓
Routes/Controllers (FastAPI routers)
  - HTTP handling
  - Request/response serialization
  - Authentication/authorization checks
  - Input validation (Pydantic models)
    ↓
Service Layer
  - Business logic
  - Orchestration of multiple repositories
  - Transaction boundaries
  - Error handling and logging
    ↓
Repository Layer
  - Data access abstraction
  - Query building
  - Database-agnostic interface
  - CRUD operations
    ↓
Database (SQLAlchemy models)
  - ORM models
  - Database schema
  - Migrations (Alembic)
```

**Why This Pattern?**
- **Testability**: Service layer can be tested without database (mock repositories)
- **Maintainability**: Business logic separated from data access
- **Flexibility**: Can swap data sources without changing business logic
- **Clarity**: Each layer has a single responsibility

**Anti-Pattern**: Fat controllers that do database queries directly
```python
# ❌ BAD: Controller doing database access
@app.get("/users/{user_id}")
async def get_user(user_id: int, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404)
    return user

# ✅ GOOD: Controller → Service → Repository
@app.get("/users/{user_id}")
async def get_user(user_id: int, service: UserService = Depends()):
    return await service.get_user(user_id)
```

**Directory Structure**:
```
src/
  api/
    routes/          # FastAPI routers
      users.py
      auth.py
  services/          # Business logic
    user_service.py
    auth_service.py
  repositories/      # Data access
    user_repository.py
  models/            # SQLAlchemy models
    user.py
  schemas/           # Pydantic schemas
    user.py
  core/              # Configuration, dependencies
    config.py
    database.py
```

**Reference**: See `/Users/sam/Code/perso/ev-consumption-api/ARCHITECTURE_REFACTORING_STATUS.md` for detailed Service→Repository→Database documentation

### Testing Requirements

**Personal Projects: 80% minimum coverage**

Testing pyramid:
1. **Unit Tests** (60% of tests): Fast, isolated, mock dependencies
   - Service layer tests (mock repositories)
   - Utility function tests
   - Model validation tests

2. **Integration Tests** (30% of tests): With real database
   - Repository layer tests with test database
   - Service layer with real repositories
   - Database migrations testing

3. **E2E Tests** (10% of tests): Full API testing
   - Happy path scenarios
   - Error handling
   - Authentication flows

**Standards**:
- pytest + pytest-cov
- Fixtures in `conftest.py`
- Test database with automatic cleanup
- Async test support with pytest-asyncio
- Mock external dependencies (APIs, services)

**Coverage Gate**:
```yaml
# .github/workflows/ci-cd.yml
- name: Run tests with coverage
  run: |
    pytest --cov=src --cov-report=term --cov-report=xml --cov-fail-under=80
```

**Example Test Structure**:
```python
# tests/unit/services/test_user_service.py
@pytest.fixture
def mock_user_repo():
    return Mock(spec=UserRepository)

async def test_get_user_success(mock_user_repo):
    mock_user_repo.get_by_id.return_value = User(id=1, name="Test")
    service = UserService(mock_user_repo)

    user = await service.get_user(1)

    assert user.id == 1
    assert user.name == "Test"
    mock_user_repo.get_by_id.assert_called_once_with(1)
```

### CI/CD Pipeline

**MANDATORY** for all personal projects. No exceptions.

**Complete GitHub Actions Workflow**:
```yaml
name: CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install -r requirements-dev.txt

      - name: Run tests with coverage
        run: pytest --cov=src --cov-report=term --cov-report=xml --cov-fail-under=80

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/python@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  build:
    needs: [test, security]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:latest
            ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache
          cache-to: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache,mode=max

  version:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Bump version and push tag
        uses: anothrNick/github-tag-action@1.64.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          WITH_V: true
          DEFAULT_BUMP: patch
```

**Reference**: See `/Users/sam/Code/perso/id7-car-history/.github/workflows/ci-cd.yml` for production example

**Pipeline Stages**:
1. **Test**: Multi-Python version testing (3.10, 3.11, 3.12), 80% coverage gate
2. **Security**: Snyk scanning for vulnerabilities
3. **Build**: Docker multi-stage build, push to ghcr.io
4. **Version**: Semantic version bump on main merge
5. **Deploy** (optional): K8s deployment or cloud platform

### Security Standards

**ZERO TOLERANCE** - these are non-negotiable:

#### 1. Authentication

**Every API must have authentication**. No exceptions.

Options:
- **Bearer Token**: For service-to-service or internal APIs
- **OAuth2**: For user-facing APIs
- **API Keys**: Only for rate-limited public APIs

```python
# FastAPI with Bearer token
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

async def verify_token(credentials: HTTPAuthorizationCredentials = Depends(security)):
    token = credentials.credentials
    if not is_valid_token(token):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials"
        )
    return token

@app.get("/protected", dependencies=[Depends(verify_token)])
async def protected_route():
    return {"message": "You have access"}
```

**No Exceptions**: Don't accept "it's just internal" or "we'll add auth later"

#### 2. Environment Variable Validation

**Validate on startup**, fail fast:

```python
# src/core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    api_key: str
    secret_key: str
    environment: str = "development"

    class Config:
        env_file = ".env"
        case_sensitive = False

# Will raise ValidationError if required env vars are missing
settings = Settings()
```

**Always provide .env.example**:
```bash
# .env.example
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
API_KEY=your_api_key_here
SECRET_KEY=your-secret-key-here
ENVIRONMENT=development
```

#### 3. Input Validation

**Validate at boundaries** using Pydantic:

```python
from pydantic import BaseModel, Field, validator

class UserCreate(BaseModel):
    email: str = Field(..., regex=r'^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$')
    password: str = Field(..., min_length=8, max_length=100)
    age: int = Field(..., ge=0, le=150)

    @validator('password')
    def password_strength(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain uppercase')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain digit')
        return v
```

#### 4. Snyk Security Scanning

**MANDATORY** in CI pipeline:

```yaml
- name: Run Snyk to check for vulnerabilities
  uses: snyk/actions/python@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

Fix all HIGH and CRITICAL vulnerabilities before merging.

#### 5. Secrets Management

**Rules**:
- Never commit secrets to git
- Use environment variables
- Use .env.example templates (without real secrets)
- Add .env to .gitignore
- Use secrets management for production (AWS Secrets Manager, K8s Secrets)

**Check on every project**:
```bash
# Audit for potential secrets in git history
git log -p | grep -i "password\|api_key\|secret"
```

### Containerization

**Multi-stage Dockerfile** for optimal size and security:

```dockerfile
# Build stage
FROM python:3.12-slim as builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Runtime stage
FROM python:3.12-slim

WORKDIR /app

# Copy Python dependencies from builder
COPY --from=builder /root/.local /root/.local

# Create non-root user
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# Copy application code
COPY --chown=appuser:appuser . .

# Make sure scripts are executable
ENV PATH=/root/.local/bin:$PATH

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD python -c "import requests; requests.get('http://localhost:8000/health')"

EXPOSE 8000

CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Reference**: See `/Users/sam/Code/perso/musicgen-api/Dockerfile` for production example

**docker-compose.yml** for local development:

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/myapp
    depends_on:
      - db
    volumes:
      - ./src:/app/src  # Hot reload for development

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

**Key Points**:
- Multi-stage build (smaller final image)
- Python slim base images (not alpine - has issues with some packages)
- Non-root user (security)
- Health check endpoint
- .dockerignore to exclude unnecessary files

### Database & Migrations

**Default: PostgreSQL**

Use PostgreSQL unless you have a specific reason not to:
- Mature, reliable, battle-tested
- Excellent JSON support (if you need document features)
- Full-text search built-in
- Great performance for 99% of use cases

**Migrations with Alembic**:

```python
# Alembic initialization
alembic init alembic

# Create migration
alembic revision --autogenerate -m "Create users table"

# Apply migration
alembic upgrade head

# Rollback
alembic downgrade -1
```

**Migration Best Practices**:
- Review auto-generated migrations (don't trust blindly)
- Test migrations on copy of production data
- Make migrations reversible (implement downgrade)
- Never modify applied migrations
- Track migrations in git
- Run migrations in CI to catch issues early

**Connection Pooling**:
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine(
    settings.database_url,
    pool_size=20,
    max_overflow=0,
    pool_pre_ping=True,  # Verify connections before using
    echo=False
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
```

---

## Infrastructure & DevOps Architecture

### Infrastructure as Code: Terraform

**Default choice** for IaC. Modular structure:

```
terraform/
  modules/
    base_vpc/
      main.tf
      variables.tf
      outputs.tf
    kubernetes_base/
      main.tf
      variables.tf
      outputs.tf
    kubernetes_resources/
      main.tf
      variables.tf
      outputs.tf
  environments/
    dev/
      main.tf
      terraform.tfvars
    staging/
      main.tf
      terraform.tfvars
    prod/
      main.tf
      terraform.tfvars
```

**Standards**:
- Separate modules for different concerns
- Remote state with locking (S3 + DynamoDB for AWS)
- Environment separation (dev/staging/prod)
- Variables for all configurable values
- Outputs for values needed by other systems

**Reference**: See `/Users/sam/Code/perso/aws-vpc/` for modular Terraform structure

**Terraform Workflow**:
```yaml
# .github/workflows/terraform.yml
on:
  pull_request:
    paths:
      - 'terraform/**'
  push:
    branches: [main]
    paths:
      - 'terraform/**'

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Terraform Init
        run: terraform init
        working-directory: terraform/environments/prod

      - name: Terraform Plan
        run: terraform plan -out=tfplan
        working-directory: terraform/environments/prod

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: terraform apply -auto-approve tfplan
        working-directory: terraform/environments/prod
```

### Container Orchestration: Kubernetes

**For production deployments**, use Kubernetes:

**Namespace Isolation**:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp-prod
```

**Deployment with Health Checks**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp-prod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: api
        image: ghcr.io/sam/myapp:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: database-url
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

**ConfigMaps for Configuration**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: myapp-prod
data:
  environment: "production"
  log_level: "INFO"
```

**Secrets for Sensitive Data**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
  namespace: myapp-prod
type: Opaque
data:
  database-url: <base64-encoded-value>
  api-key: <base64-encoded-value>
```

**PersistentVolumeClaims for Stateful Services**:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: myapp-prod
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

**Reference**: See `/Users/sam/Code/perso/rancher-cluster/` for Kubernetes patterns

**For Local Development**: Use Docker Compose instead of Kubernetes (simpler, faster)

### CI/CD for Infrastructure

**Automate everything**:
- Terraform plan on PR
- Terraform apply on merge to main
- Kubernetes manifests validated before apply
- No manual changes to infrastructure

---

## Garmin Connect IQ / Embedded Architecture

This section is critical for client work (Darefore, microoled, Stryd).

### Memory Management (CRITICAL for MonkeyC)

**MonkeyC has severe memory constraints**. Memory optimization is not optional.

#### Profile Early and Often

```monkey
// Add memory profiling during development
System.println("Memory: " + System.getSystemStats().usedMemory + "/" + System.getSystemStats().totalMemory);
```

**Profile on target devices**:
- Start with most memory-constrained device
- Check memory after each major feature
- Monitor for leaks during long runs

#### Object Allocation Patterns

**❌ BAD: Allocate in loops**
```monkey
function onUpdate(dc) {
    for (var i = 0; i < data.size(); i++) {
        var point = new [data[i].x, data[i].y];  // Allocates every frame!
        dc.drawCircle(point[0], point[1], 5);
    }
}
```

**✅ GOOD: Reuse buffers**
```monkey
var pointBuffer = new [2];  // Allocate once

function onUpdate(dc) {
    for (var i = 0; i < data.size(); i++) {
        pointBuffer[0] = data[i].x;
        pointBuffer[1] = data[i].y;
        dc.drawCircle(pointBuffer[0], pointBuffer[1], 5);
    }
}
```

#### String Handling

**❌ BAD: String concatenation in loops**
```monkey
var result = "";
for (var i = 0; i < items.size(); i++) {
    result += items[i] + ", ";  // Creates new string each iteration
}
```

**✅ GOOD: Use format or build once**
```monkey
var parts = new [items.size()];
for (var i = 0; i < items.size(); i++) {
    parts[i] = items[i];
}
var result = parts.toString();  // Format once
```

#### Watch for Memory Leaks

**Common leak sources**:
- Timer callbacks that never get cleaned up
- Listener registrations without unregister
- Circular references in data structures
- Large cached data that's never freed

**Reference**: See `/Users/sam/Code/clients/Darefore/movesense-firmware/` for memory optimization patterns

### BLE Protocol Design

**For Garmin↔Device BLE communication**:

#### Service/Characteristic Hierarchy

```
Service: Custom Device Service (UUID: xxx)
├── Characteristic: Command (Write)
│   └── Send commands to device
├── Characteristic: Response (Read/Notify)
│   └── Receive responses from device
├── Characteristic: Data Stream (Notify)
│   └── Continuous data from device
└── Characteristic: Battery (Read)
    └── Battery level
```

**Design Principles**:
- One characteristic per logical function
- Use notifications for continuous data (not polling)
- Keep packet sizes small (20 bytes for BLE 4.0)
- Handle fragmentation for larger messages

#### Connection Reliability

**Handle connection loss gracefully**:
```monkey
function onConnectionLost() {
    // Clear any pending operations
    clearPendingCommands();

    // Update UI to show disconnected
    updateConnectionStatus(false);

    // Don't crash - keep app running
}

function onConnectionRestored() {
    // Re-sync state with device
    resyncDeviceState();

    // Update UI
    updateConnectionStatus(true);
}
```

**Minimize data transfer frequency**:
- Batch updates when possible
- Use delta encoding (send only changes)
- Compress data if protocol allows
- Respect BLE power budget

#### Battery Impact Considerations

**Connection interval**: Longer intervals = better battery
- Default: 30-50ms for responsive UI
- Background: 100-200ms to save battery

**Notification frequency**: Don't spam
- Sensor data: 1-10 Hz depending on use case
- Status updates: Only on change
- Battery level: Every 5 minutes max

**Reference**: Darefore projects use BLE extensively for Movesense↔Garmin communication

### Multi-Device Support

**Challenge**: Garmin devices have vastly different capabilities

| Device Type | Memory | Screen | Notes |
|-------------|--------|--------|-------|
| Forerunner 945 | High | 240x240 | Full features |
| Fenix 6 | High | 260x260 | Full features |
| Vivoactive 3 | Medium | 240x240 | Limited memory |
| Forerunner 245 | Low | 240x240 | Very constrained |

**Strategy**:
1. **Test on most constrained device first**
   - If it works on FR245, it works everywhere
   - If you test on Fenix 6 first, you'll be surprised by crashes on FR245

2. **Device-specific asset optimization**
   ```xml
   <!-- resources/drawables/drawables.xml -->
   <drawables>
     <bitmap id="Logo" filename="logo_240.png">
       <device>fenix6</device>
       <device>fr945</device>
     </bitmap>
     <bitmap id="Logo" filename="logo_small.png">
       <device>fr245</device>  <!-- Smaller image for constrained device -->
     </bitmap>
   </drawables>
   ```

3. **Graceful feature degradation**
   ```monkey
   if (System.getSystemStats().totalMemory > 100000) {
       // Enable advanced features on high-memory devices
       enableAdvancedGraphs();
   } else {
       // Basic features only on constrained devices
       enableBasicDisplay();
   }
   ```

4. **Memory budgets per device class**
   - High-end (Fenix, FR945): Can use 80% of available memory
   - Mid-range (Vivoactive): Use max 60% of available memory
   - Low-end (FR245): Use max 40% of available memory (leave headroom for system)

**Reference**: See `/Users/sam/Code/clients/Darefore/Datafield/` for multi-device MonkeyC patterns

### Testing for Embedded

**Simulator testing is insufficient**. You MUST test on real hardware.

**Testing Checklist**:
- [ ] Test on lowest-memory target device
- [ ] Profile memory usage during typical session
- [ ] Test BLE connection/disconnection scenarios
- [ ] Battery life testing (run for hours, measure drain)
- [ ] Test with actual sensor hardware (if BLE device)
- [ ] Test all user interactions on physical buttons
- [ ] Test in sunlight (screen visibility)
- [ ] Test during actual activity (running/cycling)

**Memory Profiling**:
```monkey
// Add debug overlay during development
function onUpdate(dc) {
    // ... normal rendering ...

    if (DEBUG) {
        var stats = System.getSystemStats();
        var memText = "Mem: " + stats.usedMemory + "/" + stats.totalMemory;
        dc.drawText(10, 10, Graphics.FONT_TINY, memText, Graphics.TEXT_JUSTIFY_LEFT);
    }
}
```

**Battery Testing Protocol**:
1. Full charge device
2. Run app for 2 hours during activity
3. Measure battery drain
4. Compare to baseline (device without app)
5. Target: < 10% additional drain over 2 hours

---

## Frontend / Static Site Architecture

### Framework Choice

**Static Sites**: Astro (default)
- Rationale: Fast, excellent DX, brings your own framework
- Use for: Documentation, blogs, marketing sites
- Example: Personal website, project documentation

**Mobile Apps**: React Native with Expo
- Rationale: Cross-platform, large ecosystem, fast development
- Use for: iOS/Android apps
- Example: Companion apps for Garmin devices

**Web Apps**: React or Next.js
- React: For SPAs with separate backend
- Next.js: For SSR or SSG with API routes
- Use for: Complex interactive web apps

### State Management

**Simple apps** (< 5 components with shared state):
- React Context API
- No external library needed

**Complex apps** (> 5 components, complex state):
- Zustand (lightweight) or Redux Toolkit (if you need middleware)
- Don't use Redux for simple apps (overkill)

**Example**:
```typescript
// Using Zustand for simple global state
import create from 'zustand';

interface AppState {
  user: User | null;
  setUser: (user: User) => void;
  isLoading: boolean;
  setLoading: (loading: boolean) => void;
}

export const useAppStore = create<AppState>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  isLoading: false,
  setLoading: (loading) => set({ isLoading: loading }),
}));
```

### BLE Integration (React Native)

**For mobile apps that connect to BLE devices**:

**Library**: @react-native-ble-plx (standard choice)

```typescript
import { BleManager } from 'react-native-ble-plx';

const bleManager = new BleManager();

// Scan for devices
const scanForDevices = () => {
  bleManager.startDeviceScan(null, null, (error, device) => {
    if (error) {
      console.error('Scan error:', error);
      return;
    }

    if (device && device.name === 'MyDevice') {
      bleManager.stopDeviceScan();
      connectToDevice(device);
    }
  });
};

// Connect to device
const connectToDevice = async (device) => {
  const connected = await device.connect();
  await connected.discoverAllServicesAndCharacteristics();

  // Subscribe to notifications
  connected.monitorCharacteristicForService(
    SERVICE_UUID,
    CHARACTERISTIC_UUID,
    (error, characteristic) => {
      if (characteristic) {
        const data = base64.decode(characteristic.value);
        handleData(data);
      }
    }
  );
};
```

**BLE Best Practices**:
- Handle permissions properly (location for BLE scanning)
- Request only while scanning (don't keep permissions always)
- Clean up subscriptions on unmount
- Handle connection loss gracefully
- Show clear UI feedback for connection state

### Deployment

**Static Sites**:
- GitHub Pages (for public projects)
- Vercel or Netlify (for more complex needs)
- S3 + CloudFront (if already on AWS)

**Mobile Apps**:
- Expo EAS Build & Submit for app stores
- TestFlight for iOS beta testing
- Google Play Internal Testing for Android

**GitHub Actions for deployment**:
```yaml
name: Deploy Static Site

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

---

## Documentation Standards

**MANDATORY** for all personal projects. No exceptions.

### 1. CLAUDE.md (REQUIRED)

Every personal project MUST have a CLAUDE.md file at the root. This is the primary documentation for AI assistants and future maintainers.

**Template**:
```markdown
# [Project Name]

## Purpose
[1-2 paragraphs: What problem does this solve? Why does it exist?]

## Architecture Overview

### Technology Stack
- Language: Python 3.12
- Framework: FastAPI
- Database: PostgreSQL
- ORM: SQLAlchemy
- Testing: pytest
- CI/CD: GitHub Actions

### Architectural Pattern
This project follows the Service→Repository→Database pattern:
- **Routes** (`src/api/routes/`): HTTP handling, validation
- **Services** (`src/services/`): Business logic, orchestration
- **Repositories** (`src/repositories/`): Data access abstraction
- **Models** (`src/models/`): SQLAlchemy ORM models

[Include architecture diagram if complex]

## Key Architectural Decisions

### Why FastAPI instead of Flask?
[Rationale for framework choice]

### Why PostgreSQL?
[Rationale for database choice]

### Why Service→Repository→Database pattern?
[Explain testability, maintainability benefits]

## Development Workflow

### Initial Setup
```bash
# Clone and install
git clone [repo-url]
cd [project-name]
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Set up environment
cp .env.example .env
# Edit .env with your values

# Set up database
alembic upgrade head

# Run tests
pytest
```

### Running Locally
```bash
# Start database
docker-compose up -d db

# Run API
uvicorn src.main:app --reload

# API available at http://localhost:8000
# Docs at http://localhost:8000/docs
```

### Running Tests
```bash
# All tests with coverage
pytest --cov=src --cov-report=term

# Specific test file
pytest tests/unit/services/test_user_service.py

# With verbose output
pytest -v
```

## Testing Approach

### Coverage Requirements
- Minimum: 80%
- Target: 90%+

### Test Organization
- `tests/unit/`: Fast, isolated tests (mock dependencies)
- `tests/integration/`: Tests with real database
- `tests/e2e/`: Full API tests

### Test Database
Tests use a separate test database that's automatically created and torn down.

## Deployment

### CI/CD Pipeline
1. **Test**: Multi-Python version (3.10-3.12), 80% coverage gate
2. **Security**: Snyk scanning
3. **Build**: Docker image to ghcr.io
4. **Version**: Semantic version bump
5. **Deploy**: [Kubernetes/Cloud/etc.]

### Manual Deployment
[If applicable]

## Common Pitfalls and Gotchas

### Database Connection Pooling
[Explain any specific configuration needed]

### Memory Usage
[If applicable, especially for long-running processes]

### Rate Limiting
[If API has rate limits]

## Environment Variables

See `.env.example` for all required variables.

Critical variables:
- `DATABASE_URL`: PostgreSQL connection string
- `API_KEY`: For external service authentication
- `SECRET_KEY`: For JWT signing

## Project-Specific Notes

[Any quirks, workarounds, or special considerations for this specific project]
```

**Example**: Reference `/Users/sam/Code/perso/ev-consumption-api/ARCHITECTURE_REFACTORING_STATUS.md` for comprehensive documentation

### 2. README.md (REQUIRED)

**Purpose**: User-facing documentation (GitHub landing page)

**Template**:
```markdown
# [Project Name]

[Brief description - 1-2 sentences]

[![CI/CD](https://github.com/[user]/[repo]/actions/workflows/ci-cd.yml/badge.svg)](https://github.com/[user]/[repo]/actions/workflows/ci-cd.yml)
[![codecov](https://codecov.io/gh/[user]/[repo]/branch/main/graph/badge.svg)](https://codecov.io/gh/[user]/[repo])

## Features

- Feature 1
- Feature 2
- Feature 3

## Architecture

[High-level architecture diagram or explanation]

This project uses:
- FastAPI for the web framework
- PostgreSQL for data storage
- Service→Repository→Database pattern for code organization

## Quick Start

```bash
# Install dependencies
pip install -r requirements.txt

# Set up environment
cp .env.example .env

# Run migrations
alembic upgrade head

# Start server
uvicorn src.main:app --reload
```

## API Documentation

Once running, visit:
- OpenAPI docs: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

## Development

See [CLAUDE.md](CLAUDE.md) for detailed development documentation.

## Testing

```bash
pytest --cov=src
```

## Deployment

[Brief deployment instructions or link to deployment docs]

## License

[License information]
```

### 3. Architecture Decision Records (ADRs)

**REQUIRED** for significant architectural decisions.

**When to create an ADR**:
- Framework or language changes
- Major refactoring decisions
- Security architecture decisions
- Infrastructure changes
- Breaking changes to APIs

**Format**: `docs/adr/YYYYMMDD-decision-title.md`

**Template**:
```markdown
# [Number]. [Title]

Date: YYYY-MM-DD

## Status

[Proposed | Accepted | Deprecated | Superseded by ADR-XXX]

## Context

[What is the issue that we're seeing that is motivating this decision or change?]

## Decision

[What is the change that we're proposing and/or doing?]

## Consequences

### Positive
- [What becomes easier or better?]

### Negative
- [What becomes harder or worse?]

### Neutral
- [What changes but is neither better nor worse?]

## Alternatives Considered

### Alternative 1: [Name]
[Why we didn't choose this]

### Alternative 2: [Name]
[Why we didn't choose this]
```

**Example ADR**:
```markdown
# 1. Use FastAPI instead of Flask for new API projects

Date: 2024-11-15

## Status

Accepted

## Context

We need to choose a web framework for new Python API projects. Historically, we've used Flask, but FastAPI has gained significant traction and offers several advantages for modern API development.

## Decision

We will use FastAPI as the default framework for all new Python API projects. Existing Flask projects will remain Flask unless a major refactor makes migration worthwhile.

## Consequences

### Positive
- Automatic OpenAPI documentation generation (no manual Swagger setup)
- Native async/await support for better performance on I/O-bound operations
- Built-in request validation with Pydantic (type safety)
- Better IDE support due to type hints
- Modern Python 3.10+ features encouraged

### Negative
- Team needs to learn new framework
- Some Flask extensions don't have FastAPI equivalents
- Existing Flask projects create a two-framework codebase

### Neutral
- Migration path exists for Flask apps if needed
- Both frameworks use similar routing patterns

## Alternatives Considered

### Alternative 1: Continue using Flask
- Pros: Team familiarity, large ecosystem
- Cons: No automatic docs, no native async, manual validation setup
- Why rejected: FastAPI advantages outweigh learning cost

### Alternative 2: Use Django REST Framework
- Pros: Batteries included, Django ORM
- Cons: Heavy for simple APIs, opinionated structure
- Why rejected: Too much overhead for typical API projects
```

### 4. .env.example (REQUIRED)

**ALWAYS** provide environment variable template:

```bash
# Database Configuration
DATABASE_URL=postgresql://user:password@localhost:5432/dbname

# API Keys
API_KEY=your_api_key_here
EXTERNAL_SERVICE_KEY=your_service_key_here

# Security
SECRET_KEY=your-secret-key-here
JWT_ALGORITHM=HS256
JWT_EXPIRATION_MINUTES=30

# Application Settings
ENVIRONMENT=development
DEBUG=true
LOG_LEVEL=INFO

# Optional Features
ENABLE_FEATURE_X=false
```

**Rules**:
- Comment every variable
- Provide example values (non-sensitive)
- Group related variables
- Document optional vs required variables

---

## Format-Specific Guidance

### When Planning a New Project

**Step-by-step approach**:

1. **Detect Project Type**
   - Python Web API?
   - Infrastructure/DevOps?
   - Garmin/Embedded?
   - Frontend/Static site?

2. **Apply Technology Stack Defaults**
   - Python API → FastAPI + PostgreSQL + Service→Repository→Database
   - Infrastructure → Terraform + K8s
   - Garmin → MonkeyC with memory optimization
   - Frontend → Astro (static) or React Native (mobile)

3. **Set Up Mandatory Documentation**
   - Create CLAUDE.md with architecture overview
   - Create README.md with quick start
   - Create .env.example template
   - Set up ADR directory: `docs/adr/`

4. **Configure CI/CD Pipeline**
   - GitHub Actions workflow
   - Multi-version testing (if applicable)
   - 80% coverage gate
   - Snyk security scanning
   - Docker build and push
   - Semantic versioning

5. **Create Initial Project Structure**
   - Python API: `src/api/routes/`, `src/services/`, `src/repositories/`, `src/models/`
   - Terraform: `modules/`, `environments/`
   - MonkeyC: Follow Connect IQ project structure
   - React: Component structure with clear separation

6. **Security Setup**
   - Authentication mechanism (Bearer/OAuth2)
   - Environment variable validation
   - Input validation (Pydantic for Python)
   - .gitignore with .env

7. **Testing Setup**
   - pytest + pytest-cov (Python)
   - Test directory structure
   - Initial fixtures in conftest.py
   - CI integration

**Example Dialogue**:
```
User: "I want to build a new API for tracking weather data"

Assistant Response:
- Project Type: Python Web API
- Stack: FastAPI + PostgreSQL + Service→Repository→Database
- Authentication: Bearer token (since it's tracking data, needs auth)
- Testing: pytest with 80% coverage
- CI/CD: GitHub Actions with multi-Python testing, Snyk, Docker

Let me help you set up the project structure:

1. First, I'll create the initial directory structure following the Service→Repository→Database pattern
2. Set up FastAPI with a /health endpoint
3. Configure PostgreSQL with SQLAlchemy
4. Set up pytest with coverage
5. Create GitHub Actions workflow
6. Add CLAUDE.md and README.md documentation
7. Create .env.example template

Let's start with the project structure...
```

### When Auditing Existing Projects

**Audit Checklist**:

#### 1. Architectural Pattern Compliance
- [ ] **Python APIs**: Using Service→Repository→Database pattern?
  - Check for fat controllers (anti-pattern)
  - Verify service layer exists
  - Verify repository abstraction exists

- [ ] **Infrastructure**: Using Terraform modules?
  - Check for modular structure
  - Verify remote state configuration
  - Check environment separation

#### 2. Testing & Coverage
- [ ] Tests exist?
- [ ] Coverage meets 80% minimum (personal projects)?
- [ ] Test structure (unit/integration/e2e)?
- [ ] Tests run in CI?

#### 3. CI/CD Completeness
- [ ] GitHub Actions workflow exists?
- [ ] Multi-version testing (if applicable)?
- [ ] Coverage gate in pipeline?
- [ ] Snyk security scanning?
- [ ] Docker build automated?
- [ ] Versioning automated?

#### 4. Security Audit
- [ ] Authentication present (no unauthenticated APIs)?
- [ ] Environment variable validation on startup?
- [ ] Input validation (Pydantic models)?
- [ ] No secrets committed to git?
- [ ] .env in .gitignore?
- [ ] Snyk scanning enabled?

#### 5. Documentation Completeness
- [ ] CLAUDE.md exists and is comprehensive?
- [ ] README.md exists with quick start?
- [ ] .env.example template exists?
- [ ] ADRs for major decisions?

#### 6. Code Organization
- [ ] Clear directory structure?
- [ ] Separation of concerns?
- [ ] No God objects (classes doing too much)?
- [ ] DRY principle followed (but not over-abstracted)?

#### 7. Containerization (if applicable)
- [ ] Dockerfile exists?
- [ ] Multi-stage build?
- [ ] Non-root user?
- [ ] Health check?
- [ ] docker-compose.yml for local dev?

**Output Format**:
```markdown
# Architecture Audit: [Project Name]

## Summary
[Overall assessment - compliant/needs work/major issues]

## Compliance Score: X/7
- Architectural Pattern: ✅/❌
- Testing & Coverage: ✅/❌
- CI/CD: ✅/❌
- Security: ✅/❌
- Documentation: ✅/❌
- Code Organization: ✅/❌
- Containerization: ✅/❌

## Critical Issues (Fix Immediately)
1. [Issue 1 - e.g., "No authentication on API endpoints"]
2. [Issue 2]

## Important Issues (Fix Soon)
1. [Issue 1 - e.g., "Test coverage at 45%, needs to reach 80%"]
2. [Issue 2]

## Improvements (Nice to Have)
1. [Improvement 1]
2. [Improvement 2]

## Detailed Findings

### Architectural Pattern
[Findings]

### Testing & Coverage
[Findings]

### CI/CD
[Findings]

### Security
[Findings]

### Documentation
[Findings]

### Code Organization
[Findings]

### Containerization
[Findings]

## Recommended Action Plan
1. [Action 1]
2. [Action 2]
3. [Action 3]
```

### When Conducting Architecture Reviews

**Review Process**:

1. **Evaluate Alignment with Patterns**
   - Service→Repository→Database pattern followed?
   - Clear separation of concerns?
   - Appropriate abstraction levels?

2. **Check for Over-Engineering**
   - Unnecessary abstractions?
   - Premature optimization?
   - Over-configured systems?
   - Too many layers?

3. **Check for Under-Engineering**
   - Missing tests?
   - No error handling?
   - No logging/monitoring?
   - Security gaps?

4. **Security Review**
   - Authentication/authorization proper?
   - Input validation comprehensive?
   - Secrets management correct?
   - Vulnerability scanning in place?

5. **Performance Considerations**
   - N+1 query problems?
   - Missing indexes?
   - Inefficient algorithms?
   - Memory leaks (especially for embedded)?

6. **Maintainability Assessment**
   - Code readability?
   - Documentation quality?
   - Test coverage?
   - Clear structure?

**Output Format**:
```markdown
# Architecture Review: [Feature/Module Name]

## Overview
[What was reviewed]

## Strengths
- [Strength 1]
- [Strength 2]

## Concerns

### Critical
- [Concern 1 - e.g., "No input validation on user-provided data"]

### Important
- [Concern 2 - e.g., "Fat controller with business logic"]

### Minor
- [Concern 3]

## Recommendations
1. [Recommendation 1]
2. [Recommendation 2]

## Code Examples

### Issue: [Description]
```python
# Current (problematic)
[code]

# Recommended
[code]
```

## Next Steps
1. [Step 1]
2. [Step 2]
```

### When Integrating with Client Projects

**Assessment Process**:

1. **Determine Ownership Level**
   - Full architectural control? (e.g., Darefore/Datafield)
   - Collaborative? (e.g., Stryd/Stryd-Zones)
   - Support role? (e.g., microoled/activelook-garmin-app)

2. **Understand Existing Patterns**
   - What's their current architecture?
   - What patterns do they use?
   - What constraints do they have?
   - What's their tech stack?

3. **Adapt Recommendations**
   - **Full Control**: Apply Sam's patterns fully
   - **Collaborative**: Suggest improvements, align with team
   - **Support**: Follow their patterns strictly

4. **Document Deviations**
   - If patterns differ from Sam's standards, document why in CLAUDE.md
   - Explain constraints that led to different choices
   - Note what you'd do differently if you had full control

**Example: Full Control (Darefore)**:
```
Context: New Garmin data field for Darefore's Movesense device

Approach:
- Apply full Sam architecture standards
- Memory optimization from day one
- Comprehensive BLE protocol design
- Multi-device support strategy
- Complete documentation (CLAUDE.md)
- Testing on real hardware throughout
- ADRs for major decisions
```

**Example: Collaborative (Stryd)**:
```
Context: Contributing to Stryd's existing Garmin app

Approach:
- Understand their existing code patterns
- Follow their naming conventions
- Use their testing approach
- Suggest improvements diplomatically ("Have you considered X?")
- Document shared decisions in PR descriptions
- Don't force Sam's patterns if they conflict with team norms
```

**Example: Support (microoled)**:
```
Context: Bug fixes and minor features for microoled's ActiveLook app

Approach:
- Follow existing patterns strictly
- Don't introduce major architectural changes
- Focus on minimal, targeted fixes
- Match their code style
- Don't add "improvements" beyond what's requested
```

---

## What Sam Does NOT Do

Comprehensive exclusion list - flag these immediately:

### APIs Without Authentication
❌ "It's just internal" - **NO**. Every API gets authentication.
❌ "We'll add auth later" - **NO**. Auth from day one.

### Testing Shortcuts
❌ "We'll add tests later" - **NO**. Tests written with code.
❌ Coverage below 80% on personal projects - **UNACCEPTABLE**.
❌ "Manual testing is enough" - **NO**. Automated tests required.

### CI/CD Shortcuts
❌ "Too much setup overhead" - **NO**. CI/CD is part of project init.
❌ Manual deployments without automation - **NO**. Automate everything.
❌ Skipping security scanning - **NO**. Snyk in pipeline.

### Architecture Mistakes
❌ Fat controllers with business logic - **NO**. Service→Repository→Database.
❌ Database queries in routes - **NO**. Repository abstraction required.
❌ Monolithic single file - **NO**. Clear module separation.
❌ Premature microservices - **NO**. Start monolithic.

### Framework Choices
❌ Using Flask for new projects - **NO**. FastAPI is the standard.
❌ Using MongoDB for relational data - **NO**. PostgreSQL default.
❌ Rolling custom auth instead of OAuth2/JWT - **NO**. Use standards.

### Documentation Avoidance
❌ Missing CLAUDE.md - **UNACCEPTABLE** for personal projects.
❌ No README - **UNACCEPTABLE**.
❌ No .env.example - **UNACCEPTABLE**.
❌ "Code is self-documenting" - **NO**. Document decisions.

### Security Negligence
❌ Secrets committed to git - **ZERO TOLERANCE**.
❌ No input validation - **UNACCEPTABLE**.
❌ No environment variable validation - **UNACCEPTABLE**.
❌ Docker containers running as root - **NO**. Create non-root user.
❌ Missing health check endpoints - **NO**. /health required.

### Code Quality Issues
❌ No error handling - **NO**. Explicit error handling required.
❌ No logging - **NO**. Structured logging required.
❌ Silent failures - **NO**. Fail loudly or handle explicitly.
❌ No type hints (Python) - **NO**. Type hints everywhere.

### Embedded/Garmin Mistakes
❌ Not testing on real hardware - **NO**. Simulator insufficient.
❌ Ignoring memory constraints - **CRITICAL**. Profile memory always.
❌ Allocating in loops - **NO**. Reuse buffers.
❌ Not testing on constrained devices - **NO**. Test on FR245 first.

### Infrastructure Shortcuts
❌ Manual infrastructure changes - **NO**. Use Terraform.
❌ No remote state for Terraform - **NO**. S3 + DynamoDB.
❌ No environment separation - **NO**. dev/staging/prod distinct.

---

## Key Success Patterns

These are patterns Sam consistently applies across mature projects:

### 1. Service→Repository→Database Pattern (Python APIs)
**Always** separate concerns:
- Routes handle HTTP
- Services handle business logic
- Repositories handle data access

### 2. 80% Test Coverage (Personal Projects)
**Always** maintain high coverage with comprehensive test pyramid.

### 3. Comprehensive CI/CD
**Always** automate: testing, security scanning, building, versioning, deployment.

### 4. Security First
**Always** include: authentication, input validation, secrets management, vulnerability scanning.

### 5. Memory Optimization (Embedded)
**Always** profile early, minimize allocations, test on constrained devices.

### 6. Complete Documentation
**Always** provide: CLAUDE.md, README.md, .env.example, ADRs for major decisions.

### 7. Containerization Standards
**Always** use: multi-stage builds, non-root users, health checks.

### 8. Pragmatic Technology Choices
**Always** choose: FastAPI (not Flask for new projects), PostgreSQL (not MongoDB), Terraform (not manual), K8s (for production).

---

## Activation Context

This skill automatically activates when you:
- Plan a new project
- Design system architecture
- Conduct architecture reviews
- Audit existing codebases
- Make technology choices
- Set up CI/CD pipelines
- Design security architecture
- Organize code structure
- Create implementation plans
- Discuss testing strategies

Use this skill to ensure consistency with Sam's mature architectural patterns across all project types.
