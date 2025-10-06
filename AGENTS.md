# AGENTS.md - Comprehensive Guidance for AI Coding Agents

This document provides comprehensive guidance specifically for AI coding agents (Claude, GitHub Copilot, Cursor AI, etc.) working with the Aignostics Python SDK repository. It complements the detailed CLAUDE.md files throughout the codebase with agent-specific workflows, best practices, and gotchas.

## Quick Start for AI Agents

### First Steps

1. **Read this file first** - It provides the high-level context you need
2. **Review `/CLAUDE.md`** - Root documentation with architecture overview
3. **Review `/src/aignostics/CLAUDE.md`** - Module index and dependency graph
4. **Review `/CODE_STYLE.md`** - Mandatory style requirements
5. **Review `/CONTRIBUTING.md`** - Development workflows
6. **Review module-specific CLAUDE.md** - Detailed implementation guidance

### Repository Structure at a Glance

```
aignostics-python-sdk/
├── AGENTS.md            # This file - AI agent guidance
├── CLAUDE.md            # Root documentation with architecture
├── CODE_STYLE.md        # Mandatory style requirements
├── CONTRIBUTING.md      # Development workflows
├── Makefile            # Build commands (make help)
├── pyproject.toml      # Package configuration
├── src/aignostics/     # Source code
│   ├── CLAUDE.md       # Module index and architecture
│   ├── platform/       # Authentication & API client
│   │   └── CLAUDE.md   # Platform module documentation
│   ├── application/    # ML application orchestration
│   │   └── CLAUDE.md   # Application module documentation
│   ├── wsi/           # Whole slide image processing
│   │   └── CLAUDE.md   # WSI module documentation
│   ├── dataset/       # Dataset operations
│   │   └── CLAUDE.md   # Dataset module documentation
│   ├── bucket/        # Cloud storage
│   │   └── CLAUDE.md   # Bucket module documentation
│   ├── utils/         # Core infrastructure
│   │   └── CLAUDE.md   # Utils module documentation
│   ├── gui/           # Desktop interface
│   │   └── CLAUDE.md   # GUI module documentation
│   ├── notebook/      # Marimo notebook integration
│   │   └── CLAUDE.md   # Notebook module documentation
│   ├── qupath/        # QuPath bioimage analysis
│   │   └── CLAUDE.md   # QuPath module documentation
│   └── system/        # System diagnostics
│       └── CLAUDE.md   # System module documentation
└── tests/             # Test suite
    └── CLAUDE.md       # Test documentation
```

## Critical Context for AI Agents

### Domain: Computational Pathology

This SDK operates in the **medical imaging domain**, specifically **computational pathology**:

- **Whole slide images (WSI)**: Gigapixel-scale medical images of tissue samples
- **DICOM**: Medical imaging standard for storing and transmitting medical images
- **IDC (Imaging Data Commons)**: National Cancer Institute repository for imaging data
- **QuPath**: Open-source bioimage analysis platform
- **AI/ML inference**: Running machine learning models on medical imaging data
- **HIPAA compliance**: Healthcare data privacy requirements

**Key Implication**: When working with this codebase, understand that:
- Performance matters (files can be gigabytes)
- Security is critical (medical data)
- Reliability is paramount (healthcare context)
- Regulatory compliance is required

### Architecture: Modulith Pattern

This SDK uses a **modulith architecture** (NOT microservices, NOT monolith):

```
┌─────────────────────────────────────────┐
│          Single Deployable Unit         │
├─────────────────────────────────────────┤
│  Well-Defined Module Boundaries         │
│  ├── platform (foundation)              │
│  ├── application (orchestration)        │
│  ├── wsi (image processing)             │
│  ├── dataset (data operations)          │
│  ├── bucket (cloud storage)             │
│  ├── utils (infrastructure)             │
│  ├── gui (desktop interface)            │
│  ├── notebook (interactive analysis)    │
│  ├── qupath (bioimage analysis)         │
│  └── system (diagnostics)               │
└─────────────────────────────────────────┘
```

**Key Principles:**
1. **High cohesion within modules** - Related functionality stays together
2. **Loose coupling between modules** - Clear interfaces, minimal dependencies
3. **No circular dependencies** - Strict dependency hierarchy
4. **Service discovery** - Dynamic module loading via dependency injection
5. **Dual presentation layers** - CLI and GUI for most modules

### Module Pattern: Service + CLI + GUI

**Every module follows this pattern:**

```python
module/
├── _service.py      # Business logic (BaseService subclass)
├── _cli.py          # CLI commands (Typer)
├── _gui.py          # GUI interface (NiceGUI)
├── _settings.py     # Configuration (Pydantic)
└── CLAUDE.md        # Detailed documentation
```

**Dependency flow:**
```
CLI Layer   →   Service Layer   ←   GUI Layer
   ↓                ↓                    ↓
           Platform/Utils/Other Services
```

**Never:**
- CLI directly accessing GUI
- GUI directly accessing CLI
- Circular dependencies between modules

## Development Workflows for AI Agents

### Environment Setup

```bash
# Check Python version (requires 3.11+)
python --version

# Install dependencies
make install
# OR
uv sync
uv run pre-commit install

# Verify installation
uv run aignostics --help
```

### Running the Build Pipeline

```bash
# Full CI pipeline (lint, test, docs, audit)
make all

# Individual steps
make lint      # Ruff + MyPy
make test      # Pytest with coverage
make docs      # Sphinx documentation
make audit     # Security & license checks

# Test specific Python version
make test 3.12

# Help
make help
```

### Code Quality Gates

**Before submitting ANY code, ensure:**

1. **Linting passes**: `make lint`
   - Ruff formatting (Black-compatible, 120 char lines)
   - Ruff linting (ALL rules enabled except explicit exclusions)
   - MyPy strict mode (100% type coverage)

2. **Tests pass**: `make test`
   - Minimum 85% code coverage (fail on <85%)
   - All tests passing across Python 3.11, 3.12, 3.13

3. **Documentation updated**:
   - Docstrings for all public APIs (Google style)
   - CLAUDE.md updated for architectural changes
   - Type hints for all functions

4. **Security checks pass**: `make audit`
   - No secrets in code (detect-secrets)
   - No vulnerabilities (pip-audit)
   - License compliance (pip-licenses)

### Common Agent Tasks

#### Task 1: Adding a New CLI Command

```python
# In module/_cli.py
import typer
from ._service import Service

cli = typer.Typer(name="module", help="Module description")

@cli.command("new-command")
def new_command(
    param: str = typer.Option(..., help="Parameter description")
):
    """Command description shown in help."""
    service = Service()
    result = service.perform_action(param)
    console.print(result)
```

**Steps:**
1. Add command to `_cli.py`
2. Implement business logic in `_service.py`
3. Add tests in `tests/module/cli_test.py`
4. Update `module/CLAUDE.md` with command documentation
5. Run `make lint test`

#### Task 2: Adding a New Service Method

```python
# In module/_service.py
from aignostics.utils import BaseService, Health, get_logger

logger = get_logger(__name__)

class Service(BaseService):
    def health(self) -> Health:
        """Health check implementation."""
        return Health(status=Health.Code.UP)

    def new_method(self, param: str) -> dict:
        """Method description.

        Args:
            param: Parameter description

        Returns:
            Result dictionary

        Raises:
            ValueError: If param is invalid
        """
        logger.info("Executing new_method", extra={"param": param})

        if not param:
            raise ValueError("param cannot be empty")

        # Implementation
        result = {"status": "success", "param": param}

        logger.debug("Method completed", extra={"result": result})
        return result
```

**Steps:**
1. Add method to `_service.py`
2. Add type hints for all parameters and returns
3. Add Google-style docstring
4. Add structured logging
5. Add error handling with specific exceptions
6. Add tests in `tests/module/service_test.py`
7. Update `module/CLAUDE.md`
8. Run `make lint test`

#### Task 3: Adding a New Module

**Complete checklist:**

1. Create module directory: `src/aignostics/new_module/`
2. Create `_service.py` inheriting from `BaseService`
3. Create `_cli.py` with Typer commands
4. Create `_gui.py` with NiceGUI interface (optional)
5. Create `_settings.py` with Pydantic models
6. Create `__init__.py` with exports
7. Create `CLAUDE.md` with comprehensive documentation
8. Update `src/aignostics/CLAUDE.md` module index
9. Create test directory: `tests/aignostics/new_module/`
10. Create test files: `service_test.py`, `cli_test.py`, `gui_test.py`
11. Update root `CLAUDE.md` if architectural changes
12. Run `make lint test docs audit`

#### Task 4: Fixing a Bug

**Standard process:**

1. **Reproduce the bug**:
   ```bash
   # Run specific test
   pytest tests/module/test_file.py::test_function -vvv

   # Or run CLI command
   uv run aignostics command --param value
   ```

2. **Write a failing test** (if doesn't exist):
   ```python
   def test_bug_reproduction():
       """Test case that reproduces the bug."""
       service = Service()
       with pytest.raises(ExpectedError):
           service.buggy_method(problematic_input)
   ```

3. **Fix the bug**:
   - Update implementation
   - Ensure fix is minimal and focused
   - Add comments explaining the fix if non-obvious

4. **Verify the fix**:
   ```bash
   make lint test
   ```

5. **Update documentation** if behavior changed

6. **Commit with conventional commit**:
   ```bash
   git commit -m "fix(module): brief description of bug fix"
   ```

#### Task 5: Refactoring Code

**Principles:**

1. **Maintain backward compatibility** whenever possible
2. **Update tests FIRST** if changing behavior
3. **Run tests continuously** during refactoring
4. **Small, focused commits** - one logical change per commit
5. **Document breaking changes** in module CLAUDE.md

**Example refactoring workflow:**

```bash
# 1. Ensure tests pass before refactoring
make test

# 2. Make small change
# ... edit code ...

# 3. Run tests
make test

# 4. If tests fail, fix or update tests
# ... fix tests ...

# 5. Repeat until refactoring complete
# 6. Final verification
make all
```

## Critical Gotchas for AI Agents

### 1. Package Manager: uv (NOT pip)

❌ **WRONG:**
```bash
pip install package
pip install -e .
```

✅ **CORRECT:**
```bash
uv add package
uv sync
uv run aignostics
```

### 2. Import Patterns

❌ **WRONG:**
```python
# Circular imports
from aignostics.application import Service
from aignostics.platform import Client  # in application/_service.py

# Absolute imports within module
from aignostics.platform._service import Service  # in platform/_cli.py
```

✅ **CORRECT:**
```python
# Import from other modules
from aignostics.platform import Client  # OK if platform is a dependency

# Relative imports within module
from ._service import Service  # in platform/_cli.py
from ._settings import Settings  # in platform/_service.py
```

### 3. Service Discovery Pattern

❌ **WRONG:**
```python
# Manual registration
SERVICES = [PlatformService, ApplicationService, ...]
```

✅ **CORRECT:**
```python
# Automatic discovery via inheritance
from aignostics.utils import locate_implementations, BaseService

services = locate_implementations(BaseService)
# Returns all BaseService subclasses automatically
```

### 4. Error Handling

❌ **WRONG:**
```python
try:
    result = risky_operation()
except Exception as e:
    print(f"Error: {e}")  # Too generic, no context
```

✅ **CORRECT:**
```python
from aignostics.utils import get_logger

logger = get_logger(__name__)

try:
    result = risky_operation()
except SpecificException as e:
    logger.error(
        "Operation failed",
        extra={
            "operation": "risky_operation",
            "error": str(e),
            "context": additional_context
        }
    )
    raise  # Or handle appropriately
```

### 5. Configuration Management

❌ **WRONG:**
```python
# Hardcoded configuration
API_URL = "https://api.aignostics.com"
TOKEN_FILE = "~/.aignostics/token.json"
```

✅ **CORRECT:**
```python
# Pydantic settings with environment variables
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    api_url: str = "https://api.aignostics.com"
    token_file: Path = Path.home() / ".aignostics" / "token.json"

    class Config:
        env_prefix = "AIGNOSTICS_"
```

### 6. File Processing

❌ **WRONG:**
```python
# Load entire file into memory
with open(large_file, 'rb') as f:
    data = f.read()  # OOM for large WSI files!
```

✅ **CORRECT:**
```python
# Stream processing
CHUNK_SIZE = 1024 * 1024  # 1MB
with open(large_file, 'rb') as f:
    while chunk := f.read(CHUNK_SIZE):
        process_chunk(chunk)
```

### 7. Testing with External Dependencies

❌ **WRONG:**
```python
# Real API calls in tests
def test_api_call():
    client = Client()  # Makes real OAuth request!
    result = client.applications.list()
```

✅ **CORRECT:**
```python
# Mock external dependencies
@pytest.fixture
def mock_client():
    with patch('aignostics.platform.Client') as mock:
        mock_instance = MagicMock()
        mock.return_value = mock_instance
        yield mock_instance

def test_api_call(mock_client):
    mock_client.applications.list.return_value = [...]
    # Test logic
```

### 8. Version Format

❌ **WRONG:**
```python
# Missing 'v' prefix
version_id = "heta:1.2.3"  # FAILS validation!
```

✅ **CORRECT:**
```python
# Must have 'v' prefix for semver
version_id = "heta:v1.2.3"  # Validated by semver library
```

### 9. Authentication Token Format

❌ **WRONG:**
```python
# Store just the token
token_file.write_text(token)
```

✅ **CORRECT:**
```python
# Store token with expiry timestamp
token_file.write_text(f"{token}:{expiry_timestamp}")
```

### 10. GUI Testing

❌ **WRONG:**
```python
# Direct GUI testing without plugin
def test_gui():
    from aignostics.gui import create_page
    create_page()  # No testing infrastructure!
```

✅ **CORRECT:**
```python
# Use NiceGUI testing plugin
from nicegui.testing import Screen

def test_gui(screen: Screen):
    # screen fixture provided by nicegui.testing.plugin
    screen.open('/gui')
    assert screen.find('Expected Text')
```

## Module-Specific Quick References

### Platform Module

**Purpose**: Authentication & API client

**Key Files**:
- `_authentication.py` - OAuth 2.0 device flow
- `_client.py` - Main client with resources
- `resources/` - Resource wrappers (Applications, Runs)

**Common Tasks**:
```python
from aignostics.platform import Client

# Initialize client (handles auth automatically)
client = Client(cache_token=True)

# Get current user
me = client.me()

# List applications
for app in client.applications.list():
    print(app.application_id)

# List runs (max page_size=100)
for run in client.runs.list(page_size=50):
    print(run.application_run_id)
```

**Gotchas**:
- Token cached as `token:expiry_timestamp` format
- 5-minute refresh buffer before expiry
- `client.application(id)` iterates ALL applications (O(n))
- Pagination max page_size is 100 for runs

### Application Module

**Purpose**: ML application orchestration

**Key Files**:
- `_service.py` - Run lifecycle management
- `_gui/` - GUI components for run monitoring

**Common Tasks**:
```python
from aignostics.application import Service

service = Service()

# Validate version (MUST have 'v' prefix!)
version = service.application_version("heta:v2.1.0")

# Submit run
run = service.run_application(
    application_id="heta",
    files=["slide1.svs", "slide2.tiff"]
)
```

**Gotchas**:
- Version format MUST be `app-id:vX.Y.Z` (note the 'v'!)
- Uses semver.Version.is_valid() for validation
- QuPath integration requires `ijson` package
- Upload chunk size is 1MB (not configurable)

### WSI Module

**Purpose**: Whole slide image processing

**Key Files**:
- `_service.py` - Format detection and processing
- `_openslide_handler.py` - Handler for .svs, .tiff, .ndpi
- `_pydicom_handler.py` - Handler for DICOM files

**Common Tasks**:
```python
from aignostics.wsi import Service

service = Service()

# Get metadata
metadata = service.get_metadata(Path("slide.svs"))

# Generate thumbnail
thumbnail = service.get_thumbnail(
    Path("slide.svs"),
    size=(512, 512)
)
thumbnail.save("preview.jpg")
```

**Gotchas**:
- OpenSlide must be installed system-wide
- DICOM files don't support tile extraction
- Large files MUST be processed in tiles (memory!)
- Multi-frame DICOM uses middle frame for thumbnails

### Utils Module

**Purpose**: Core infrastructure

**Key Components**:
- `_di.py` - Dependency injection and service discovery
- `_log.py` - Structured logging (Logfire, Sentry)
- `_settings.py` - Pydantic-based configuration
- `_health.py` - Health check framework

**Common Tasks**:
```python
from aignostics.utils import (
    BaseService,
    Health,
    get_logger,
    locate_implementations
)

# Logging
logger = get_logger(__name__)
logger.info("Message", extra={"context": "value"})

# Service discovery
services = locate_implementations(BaseService)

# Health checks
class MyService(BaseService):
    def health(self) -> Health:
        return Health(status=Health.Code.UP)
```

**Gotchas**:
- Service discovery happens automatically (no registration needed)
- Logging uses structured format (don't use print!)
- Settings are Pydantic models (type-safe)

## Testing Strategies for AI Agents

### Test Organization

```
tests/
├── conftest.py                    # Global fixtures
├── aignostics/
│   ├── platform/
│   │   ├── authentication_test.py # Auth flow tests
│   │   ├── resources/
│   │   │   ├── runs_test.py      # Resource tests
│   │   │   └── applications_test.py
│   │   └── scheduled_test.py     # Periodic validation
│   ├── application/
│   │   ├── service_test.py       # Business logic tests
│   │   ├── cli_test.py           # CLI command tests
│   │   └── gui_test.py           # GUI component tests
│   └── ...
└── fixtures/                      # Test data
```

### Test Markers

```python
@pytest.mark.docker        # Requires Docker
@pytest.mark.scheduled     # Periodic validation
@pytest.mark.long_running  # Extended execution
@pytest.mark.sequential    # No parallel execution
@pytest.mark.skip_with_act # Skip in GitHub Act
```

### Writing Tests

**Unit Test Pattern:**

```python
from aignostics.module import Service

def test_method_success():
    """Test successful execution."""
    service = Service()
    result = service.method("valid_input")

    assert result["status"] == "success"
    assert result["data"] is not None

def test_method_validation_error():
    """Test input validation."""
    service = Service()

    with pytest.raises(ValueError, match="Invalid input"):
        service.method("")
```

**Mock External Dependencies:**

```python
from unittest.mock import patch, MagicMock

@pytest.fixture
def mock_api():
    with patch('aignostics.platform._client.PublicApi') as mock:
        api = MagicMock()
        api.list_applications.return_value = []
        mock.return_value = api
        yield api

def test_with_mock_api(mock_api):
    # Test logic using mocked API
    pass
```

### Running Tests

```bash
# All tests
make test

# Specific test file
pytest tests/aignostics/platform/authentication_test.py

# Specific test function
pytest tests/aignostics/platform/authentication_test.py::test_token_refresh

# With verbose output
pytest -vvv

# With coverage
pytest --cov=aignostics --cov-report=html

# Stop on first failure
pytest -x

# Run in parallel
pytest -n auto
```

## Debugging Strategies

### Enable Debug Logging

```bash
export AIGNOSTICS_LOG_LEVEL=DEBUG
uv run aignostics command
```

### Use Breakpoints

```python
def problematic_function():
    # ... code ...
    import pdb; pdb.set_trace()  # Debugger here
    # ... more code ...
```

### Check Service Health

```bash
uv run aignostics system health
```

### Inspect WSI Files

```bash
uv run aignostics wsi inspect slide.svs
```

### Check Authentication

```bash
uv run aignostics user whoami
```

## Documentation Standards

### Code Documentation

**Every public function/method MUST have:**

```python
def function_name(param1: str, param2: int) -> dict:
    """Brief description of what the function does.

    More detailed description if needed. Explain the purpose,
    behavior, and any important details.

    Args:
        param1: Description of param1
        param2: Description of param2

    Returns:
        Description of return value

    Raises:
        ValueError: When param1 is empty
        NotFoundException: When resource not found

    Example:
        >>> result = function_name("test", 42)
        >>> print(result["status"])
        success
    """
    pass
```

### Module Documentation

**Every CLAUDE.md MUST include:**

1. Module Overview
2. Core Responsibilities
3. User Interfaces (CLI/GUI/Service)
4. Architecture & Design Patterns
5. Critical Implementation Details
6. Usage Patterns & Best Practices
7. Testing Strategies
8. Common Pitfalls & Solutions
9. Module Dependencies
10. Development Guidelines

### Commit Messages

**Format: Conventional Commits**

```bash
# Format
<type>(<scope>): <description>

# Types
feat     # New feature
fix      # Bug fix
docs     # Documentation only
style    # Formatting, no code change
refactor # Code change, no feature/fix
perf     # Performance improvement
test     # Adding/updating tests
chore    # Maintenance tasks

# Examples
feat(platform): add pagination support for applications
fix(wsi): correct thumbnail generation for DICOM files
docs(application): update semver validation documentation
test(platform): add tests for token expiry handling
```

## Security Considerations

### Never Commit Secrets

```python
# ❌ WRONG
API_KEY = "secret_key_123"
TOKEN = "eyJhbGc..."

# ✅ CORRECT
from pydantic import SecretStr
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    api_key: SecretStr
    token: SecretStr

    class Config:
        env_prefix = "AIGNOSTICS_"

# Use environment variables
export AIGNOSTICS_API_KEY=secret_key_123
```

### Pre-commit Hook

The `detect-secrets` pre-commit hook will catch accidental secret commits.

### Authentication

- OAuth 2.0 device flow (NOT basic auth)
- JWT tokens (NOT API keys)
- Token caching with expiry
- 5-minute refresh buffer

### Data Handling

- Medical data requires HIPAA compliance
- Use signed URLs for file transfers
- Sanitize file paths (no path traversal)
- Validate all inputs (Pydantic)

## Performance Optimization

### File Processing

```python
# ❌ SLOW: Load entire file
data = file.read()

# ✅ FAST: Stream in chunks
CHUNK_SIZE = 1024 * 1024  # 1MB
while chunk := file.read(CHUNK_SIZE):
    process(chunk)
```

### Pagination

```python
# ❌ SLOW: Materialize full list
runs = list(client.runs.list())

# ✅ FAST: Use generator
for run in client.runs.list():
    if meets_criteria(run):
        break  # Early exit
```

### Caching

```python
# ❌ SLOW: Repeated lookups
for id in app_ids:
    app = client.application(id)  # O(n) each time!

# ✅ FAST: Cache applications
apps = {app.application_id: app
        for app in client.applications.list()}
for id in app_ids:
    app = apps.get(id)  # O(1)
```

## Integration with Other Tools

### GitHub Copilot

This repository includes `.github/copilot-instructions.md` with GitHub Copilot-specific guidance.

### Claude Code

You are Claude Code! Use this document + CLAUDE.md files for comprehensive guidance.

### Cursor AI

Cursor AI can use this document as a knowledge source for codebase understanding.

## Quick Command Reference

### Build Commands

```bash
make help              # Show all available commands
make install          # Install dev dependencies
make all              # Run full CI pipeline
make lint             # Ruff + MyPy
make test             # Run tests
make test 3.12        # Test specific Python version
make test_sequential  # Run sequential tests
make test_long_running # Run long-running tests
make docs             # Build Sphinx documentation
make audit            # Security & license audit
make clean            # Clean build artifacts
```

### Development Commands

```bash
# Run CLI
uv run aignostics --help
uv run aignostics command --param value

# Install package in development mode
uv sync
uv run pre-commit install

# Run specific test
pytest tests/path/to/test.py::test_function -vvv

# Run tests with coverage
pytest --cov=aignostics --cov-report=html

# Format code
uv run ruff format .

# Lint code
uv run ruff check .

# Type check
uv run mypy src

# Generate documentation
make docs
```

### Git Workflows

```bash
# Create feature branch
git checkout -b feature/description

# Make changes, commit with conventional commits
git add .
git commit -m "feat(module): description"

# Push and create PR
git push origin feature/description
```

## Getting Help

### Documentation Hierarchy

1. **This file (AGENTS.md)** - AI agent workflows and quick reference
2. **Root CLAUDE.md** - Architecture overview and module index
3. **Module CLAUDE.md** - Detailed module documentation
4. **CODE_STYLE.md** - Code style requirements
5. **CONTRIBUTING.md** - Development workflows
6. **Test CLAUDE.md** - Testing patterns and strategies

### When Stuck

1. **Check module's CLAUDE.md** for implementation details
2. **Check tests** for usage examples
3. **Run `make help`** for available commands
4. **Check logs** with debug level enabled
5. **Run health checks** to verify system state

### Common Questions

**Q: How do I add a new module?**
A: See "Task 3: Adding a New Module" section

**Q: Why is my test failing?**
A: Check test markers, ensure mocks are set up, review module CLAUDE.md for gotchas

**Q: How do I handle large files?**
A: Always stream, never load entire file. See "File Processing" gotcha

**Q: What's the dependency order?**
A: See `src/aignostics/CLAUDE.md` dependency graph

**Q: Can I use async/await?**
A: Yes, but most of the codebase is synchronous. Add async carefully

**Q: How do I test GUI components?**
A: Use `nicegui.testing.plugin` - see test CLAUDE.md

---

## Summary for AI Agents

**Remember:**

1. **Read CLAUDE.md files** - They contain critical implementation details
2. **Follow modulith architecture** - Respect module boundaries
3. **Use conventional commits** - Type(scope): description
4. **Mock external dependencies** - Never make real API calls in tests
5. **Stream large files** - Never load entirely into memory
6. **Version format is critical** - Must be `app-id:vX.Y.Z`
7. **Run `make lint test`** - Before every commit
8. **Update documentation** - When changing behavior
9. **Respect medical data sensitivity** - This is healthcare software
10. **Ask questions via code comments** - When implementation is unclear

**Core Values:**
- Readability over cleverness
- Maintainability over brevity
- Security over convenience
- Reliability over speed

This SDK powers critical medical imaging workflows. Your contributions directly impact healthcare outcomes. Code accordingly.

---

*Document version: 1.0*
*Last updated: 2025-10-06*
*Maintained by: Aignostics Development Team*
