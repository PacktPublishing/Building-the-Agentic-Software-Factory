# Chapter 6 - The Complete Harness

## Chapter Summary

The Complete Project Harness is the synthesis of all three engineering disciplines: Context Engineering, Specification Engineering, and Verification Engineering. This chapter shows how to wire them together into a self-reinforcing system - a governed, opinionated repository that actively constrains the agent's behavior at every level, from the moment it reads the first file to the moment it attempts to commit code.

The walkthrough builds a `ProductInventory` module using the full harness stack: an `architecture.md` context document, a Pydantic specification, and a `pytest` verification suite wired to a pre-commit gate.

### Key Concepts

- **The Client Harness:** Agent tooling you install in your IDE or terminal (Cursor, Claude Code, Windsurf). This is the execution environment. You configure it but cannot modify its underlying architecture.
- **The Project Harness:** Everything you build inside your repository to constrain the agent - `architecture.md`, ADRs, `.cursorrules`, `AGENTS.md`, type systems, linters, pre-commit hooks, and CI gates. This is the harness you own.
- **`architecture.md`:** A persistent, authoritative document at the repository root that describes the bounded contexts, technology choices, naming conventions, and absolute architectural constraints. The agent reads this file first in any session.
- **Harness Layers:** The harness operates at three levels - the *Information Layer* (ADRs, `architecture.md`), the *Specification Layer* (type annotations, schemas), and the *Verification Layer* (test suites, linters, pre-commit hooks, CI gates).
- **Working on the Loop vs. in the Loop:** The harness transitions the developer from being *in* the agent's execution loop (manually catching every error) to being *on* the loop (the harness catches errors automatically, and the developer handles exceptions).

---

## Code Snippets

### The `architecture.md` Context Document

Place this at the repository root. The agent reads it at the start of every session:

```markdown
# Architecture Reference - ProductInventory Service

## Bounded Context
This service manages product inventory records only. It does NOT handle orders, payments, or shipping.
Cross-context data must flow through the defined API contracts, never through direct imports.

## Technology Stack
- Language: Python 3.10+
- Validation: Pydantic v2 (MUST use, not dataclasses or TypedDict)
- Testing: pytest with pytest-cov (minimum 90% coverage required)
- Type Checking: mypy --strict (MUST pass with zero errors)

## Absolute Constraints
- ALL public functions MUST have complete type annotations.
- ALL data models MUST be Pydantic BaseModel subclasses.
- NEVER use `dict` as a parameter or return type for domain objects. Use typed Pydantic models.
- NEVER import from the `orders/`, `payments/`, or `shipping/` modules.

## Naming Conventions
- Models: PascalCase (e.g., `ProductItem`, `InventoryRecord`)
- Functions: snake_case (e.g., `add_product`, `decrement_stock`)
- Constants: UPPER_SNAKE_CASE (e.g., `MAX_STOCK_LEVEL`)
- Files: snake_case (e.g., `product_repository.py`)

## File Structure
src/
  inventory/
    models.py        - Pydantic models only
    repository.py    - Data access logic only
    service.py       - Business logic only
tests/
  inventory/
    test_models.py
    test_repository.py
    test_service.py
```

### Pydantic Models (The Specification Layer)

Save as `src/inventory/models.py`:

```python
from __future__ import annotations
from typing import Optional
from pydantic import BaseModel, Field, field_validator
import uuid

class ProductItem(BaseModel):
    id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    sku: str = Field(min_length=3, max_length=50)
    name: str = Field(min_length=1, max_length=200)
    quantity: int = Field(ge=0)
    unit_price: float = Field(gt=0)

    @field_validator('sku')
    @classmethod
    def sku_must_be_uppercase(cls, v: str) -> str:
        if v != v.upper():
            raise ValueError('SKU must be uppercase')
        return v

class CreateProductRequest(BaseModel):
    sku: str = Field(min_length=3, max_length=50)
    name: str = Field(min_length=1, max_length=200)
    quantity: int = Field(ge=0)
    unit_price: float = Field(gt=0)

class StockAdjustment(BaseModel):
    product_id: str
    delta: int  # Positive to add stock, negative to remove

class InventoryStats(BaseModel):
    total_products: int
    total_units: int
    total_value: float
```

### The Verification Suite (Write Before Implementation)

Save as `tests/inventory/test_service.py`:

```python
import pytest
from src.inventory.models import CreateProductRequest, StockAdjustment
from src.inventory.service import InventoryService

@pytest.fixture
def service() -> InventoryService:
    return InventoryService()

@pytest.fixture
def sample_request() -> CreateProductRequest:
    return CreateProductRequest(
        sku="WIDGET-001",
        name="Standard Widget",
        quantity=100,
        unit_price=9.99
    )

def test_add_product_returns_product_with_id(service, sample_request):
    product = service.add_product(sample_request)
    assert product.id is not None
    assert product.sku == "WIDGET-001"

def test_add_product_rejects_duplicate_sku(service, sample_request):
    service.add_product(sample_request)
    with pytest.raises(ValueError, match="SKU"):
        service.add_product(sample_request)

def test_adjust_stock_increases_quantity(service, sample_request):
    product = service.add_product(sample_request)
    service.adjust_stock(StockAdjustment(product_id=product.id, delta=50))
    updated = service.get_product(product.id)
    assert updated is not None
    assert updated.quantity == 150

def test_adjust_stock_cannot_go_below_zero(service, sample_request):
    product = service.add_product(sample_request)
    with pytest.raises(ValueError, match="insufficient stock"):
        service.adjust_stock(StockAdjustment(product_id=product.id, delta=-200))

def test_get_stats_returns_correct_totals(service):
    service.add_product(CreateProductRequest(
        sku="A001", name="Product A", quantity=10, unit_price=5.00))
    service.add_product(CreateProductRequest(
        sku="B002", name="Product B", quantity=20, unit_price=10.00))
    stats = service.get_stats()
    assert stats.total_products == 2
    assert stats.total_units == 30
    assert stats.total_value == pytest.approx(250.00)
```

### The Pre-Commit Verification Gate

Create `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: local
    hooks:
      - id: mypy-strict
        name: mypy (strict type check)
        entry: mypy src/ --strict
        language: system
        pass_filenames: false

      - id: pytest-full
        name: pytest (full verification suite)
        entry: pytest tests/ --tb=short -q
        language: system
        pass_filenames: false
```

Install the hooks:

```bash
pip install pre-commit
pre-commit install
```

### The Harness Prompt Template

Feed this to the agent once all harness artifacts are in place:

```
Before writing any code, read the following in order:
1. architecture.md - for architectural constraints and naming conventions
2. src/inventory/models.py - for the Pydantic model contracts
3. tests/inventory/test_service.py - for the behavioral verification criteria

Implement src/inventory/service.py and src/inventory/repository.py so that:
- All pytest tests pass
- mypy --strict passes with zero errors
- All models are Pydantic BaseModel subclasses as specified in architecture.md

Do not stop until `pytest tests/ && mypy src/` both exit with code 0.
```

---

## Supplemental Resources

### Pydantic
- **[Pydantic v2 Documentation](https://docs.pydantic.dev/latest/)** - Complete documentation for Pydantic v2, including field validators, model config, and serialization.
- **[Pydantic v2 Migration Guide](https://docs.pydantic.dev/latest/migration/)** - Important if you are coming from Pydantic v1. The API changed significantly.

### Project Harness Tooling
- **[pre-commit Documentation](https://pre-commit.com/)** - The framework used to wire the verification gate to the Git lifecycle.
- **[Cursor Rules Documentation](https://docs.cursor.com/context/rules)** - How to configure `.cursorrules` files for persistent agent context.
- **[Claude Code AGENTS.md](https://www.anthropic.com/claude-code)** - Claude Code reads `AGENTS.md` files at the repo root to establish persistent constraints.

### Architecture Documentation
- **[arc42 - Software Architecture Documentation](https://arc42.org/)** - A structured template for software architecture documentation. A good starting point for creating your `architecture.md`.
- **[C4 Model](https://c4model.com/)** - A lightweight, hierarchical approach to visualizing software architecture. The diagrams from the C4 model work well alongside an `architecture.md`.

### Code Quality
- **[Ruff - Python Linter](https://docs.astral.sh/ruff/)** - A fast Python linter and formatter (replaces flake8, isort, black). Highly recommended as a harness layer alongside mypy.
- **[ESLint](https://eslint.org/)** - The equivalent code quality tool for JavaScript/TypeScript projects.
