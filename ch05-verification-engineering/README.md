# Chapter 5 - Verification Engineering

## Chapter Summary

Verification Engineering is the discipline of proving - not just eyeballing - that agent-generated code is correct. This chapter introduces **AI-TDD (AI-driven Test-Driven Development)**, a workflow where tests are written *before* the implementation is generated. The test suite becomes a constraint specification that steers the agent, rather than a post-hoc validation step.

The walkthrough builds a Python `UserRepository` module using a `pytest` test suite as the specification, combined with `mypy` for static type enforcement.

### Key Concepts

- **AI-TDD (Inverted TDD):** Write the test suite first. Feed both the tests and the specification to the agent. The tests become the verification gate. The agent's output is only accepted when all tests pass, not when it looks correct.
- **Tests as Specifications:** A well-written test suite is a more precise specification than any English paragraph. A test like `assert repo.find_by_email("x@example.com") is None` is unambiguous; "handle missing users gracefully" is not.
- **Static Type Systems as Verification:** Type annotations (`mypy`, TypeScript strict mode) are a free, always-on verification layer. A `mypy` pass proves structural correctness before a single test runs.
- **The Verification Gate:** A technical checkpoint in the Project Harness (pre-commit hook, CI step) that blocks agent-generated code from advancing until it passes the full verification suite.
- **Red-Green-Refactor for Agents:** Red: write a failing test. Green: prompt the agent to make it pass. Refactor: prompt the agent to clean up while keeping tests green.

---

## Code Snippets

### Setup: Python Environment

```bash
mkdir verification-exercise && cd verification-exercise
python3 -m venv .venv
source .venv/bin/activate
pip install pytest mypy
```

### The Test Suite (Write This First - Before Any Implementation)

Save as `tests/test_user_repository.py`:

```python
import pytest
from src.user_repository import UserRepository, User

# -- Fixtures ------------------------------------------------------------------

@pytest.fixture
def repo() -> UserRepository:
    """Returns a fresh, empty UserRepository for each test."""
    return UserRepository()

@pytest.fixture
def sample_user() -> dict:
    return {
        "name": "Jane Doe",
        "email": "jane@example.com",
        "role": "admin"
    }

# -- Creation Tests ------------------------------------------------------------

def test_create_user_returns_user_with_id(repo, sample_user):
    user = repo.create(sample_user)
    assert user.id is not None
    assert isinstance(user.id, str)

def test_create_user_stores_correct_fields(repo, sample_user):
    user = repo.create(sample_user)
    assert user.name == "Jane Doe"
    assert user.email == "jane@example.com"
    assert user.role == "admin"

def test_create_user_rejects_duplicate_email(repo, sample_user):
    repo.create(sample_user)
    with pytest.raises(ValueError, match="already exists"):
        repo.create(sample_user)

def test_create_user_requires_name(repo):
    with pytest.raises(ValueError):
        repo.create({"email": "x@example.com", "role": "user"})

def test_create_user_requires_email(repo):
    with pytest.raises(ValueError):
        repo.create({"name": "Bob", "role": "user"})

# -- Retrieval Tests -----------------------------------------------------------

def test_find_by_id_returns_correct_user(repo, sample_user):
    created = repo.create(sample_user)
    found = repo.find_by_id(created.id)
    assert found is not None
    assert found.id == created.id

def test_find_by_id_returns_none_for_missing_user(repo):
    result = repo.find_by_id("nonexistent-id")
    assert result is None

def test_find_by_email_returns_correct_user(repo, sample_user):
    created = repo.create(sample_user)
    found = repo.find_by_email("jane@example.com")
    assert found is not None
    assert found.id == created.id

def test_find_by_email_returns_none_for_missing_email(repo):
    result = repo.find_by_email("nobody@example.com")
    assert result is None

# -- Deletion Tests ------------------------------------------------------------

def test_delete_removes_user(repo, sample_user):
    created = repo.create(sample_user)
    repo.delete(created.id)
    assert repo.find_by_id(created.id) is None

def test_delete_raises_for_missing_user(repo):
    with pytest.raises(KeyError):
        repo.delete("nonexistent-id")

# -- Listing Tests -------------------------------------------------------------

def test_list_all_returns_all_users(repo):
    repo.create({"name": "Alice", "email": "alice@example.com", "role": "user"})
    repo.create({"name": "Bob", "email": "bob@example.com", "role": "user"})
    users = repo.list_all()
    assert len(users) == 2

def test_list_all_returns_empty_list_when_no_users(repo):
    assert repo.list_all() == []
```

### The mypy Configuration

Save as `mypy.ini`:

```ini
[mypy]
python_version = 3.10
strict = true
ignore_missing_imports = true
```

### The Implementation Prompt

Once the tests are written, feed this to the agent:

```
Read the test suite in tests/test_user_repository.py completely before writing any code.

Implement the UserRepository class in src/user_repository.py so that every test passes.

Constraints:
- Define a `User` dataclass with fields: id (str), name (str), email (str), role (str).
- `UserRepository` stores users in memory using a dict keyed by id.
- `id` must be a UUID4 string, generated automatically on creation.
- All public methods must have full type annotations compatible with `mypy --strict`.
- Raise ValueError with message containing "already exists" when a duplicate email is submitted.
- Raise ValueError when required fields (name, email) are missing from the input dict.
- Raise KeyError when delete() is called with a non-existent id.

Run `pytest tests/` and `mypy src/` to verify. Do not stop until both pass with zero errors.
```

### Reference: Expected Implementation Structure

The agent should produce something equivalent to this:

```python
import uuid
from dataclasses import dataclass
from typing import Optional

@dataclass
class User:
    id: str
    name: str
    email: str
    role: str

class UserRepository:
    def __init__(self) -> None:
        self._store: dict[str, User] = {}
        self._email_index: dict[str, str] = {}  # email -> id

    def create(self, data: dict[str, str]) -> User:
        if "name" not in data or not data["name"]:
            raise ValueError("name is required")
        if "email" not in data or not data["email"]:
            raise ValueError("email is required")
        if data["email"] in self._email_index:
            raise ValueError(f"User with email {data['email']} already exists")

        user = User(
            id=str(uuid.uuid4()),
            name=data["name"],
            email=data["email"],
            role=data.get("role", "user"),
        )
        self._store[user.id] = user
        self._email_index[user.email] = user.id
        return user

    def find_by_id(self, user_id: str) -> Optional[User]:
        return self._store.get(user_id)

    def find_by_email(self, email: str) -> Optional[User]:
        user_id = self._email_index.get(email)
        return self._store.get(user_id) if user_id else None

    def delete(self, user_id: str) -> None:
        if user_id not in self._store:
            raise KeyError(f"User {user_id} not found")
        user = self._store.pop(user_id)
        del self._email_index[user.email]

    def list_all(self) -> list[User]:
        return list(self._store.values())
```

### Running the Verification Gates

```bash
# Run all tests
pytest tests/ -v

# Run type checker
mypy src/

# Run both together (useful as a pre-commit hook)
pytest tests/ && mypy src/
```

---

## Supplemental Resources

### Testing Frameworks
- **[pytest Documentation](https://docs.pytest.org/)** - The official pytest documentation. The most capable Python testing tool referenced throughout this chapter.
- **[pytest-cov](https://pytest-cov.readthedocs.io/)** - Coverage plugin for pytest. Use it to enforce minimum coverage thresholds in your CI gate.

### Static Type Checking
- **[mypy Documentation](https://mypy.readthedocs.io/)** - Official documentation for mypy, Python's standard static type checker.
- **[TypeScript Strict Mode](https://www.typescriptlang.org/tsconfig#strict)** - The TypeScript equivalent of `mypy --strict`. Enabling all strict checks is the equivalent of a free verification layer for JavaScript projects.

### Test-Driven Development
- **[Test-Driven Development: By Example - Kent Beck](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530)** - The foundational text for TDD. The AI-TDD pattern in this chapter is a direct evolution of Beck's Red-Green-Refactor cycle.
- **[Growing Object-Oriented Software, Guided by Tests - Freeman & Pryce](http://www.growing-object-oriented-software.com/)** - The canonical text on letting tests drive software design. Directly relevant to the AI-TDD workflow.

### Verification at Scale
- **[pre-commit Framework](https://pre-commit.com/)** - The tool for managing and running pre-commit hooks. Use it to automate `pytest` and `mypy` runs on every commit.
- **[GitHub Actions - Quickstart](https://docs.github.com/en/actions/writing-workflows/quickstart)** - How to run your verification suite in CI. The verification gate is only complete when it runs automatically in your pipeline.
