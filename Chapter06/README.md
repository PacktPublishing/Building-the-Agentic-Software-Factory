# Chapter 6 - The Complete Harness

## Chapter Summary

The Complete Project Harness wires together the three manual disciplines, context (Chapter 3), intent (Chapter 4), and feedback (Chapter 5), into a single, self-reinforcing system that runs the steps for you. This chapter clarifies the distinction between the **Client Harness** (the tool you install) and the **Project Harness** (the rules, constraints, and tests you build), introduces the cognitive shift from working **in the loop** to **on the loop**, and identifies the **six components** that make a harness physical.

The walkthrough builds a **Book Inventory** CLI using the full harness stack: an `architecture.md` context map, a `test_inventory.py` verification suite, an agent rules file, and `mypy` strict typing, all wired together so the agent drives execution while you supervise.

### Key Concepts

- **Client Harness vs. Project Harness:** The Client Harness is the agent tooling you install (Cursor, Claude Code, Windsurf). The Project Harness is everything you build inside your repository to constrain the agent. You buy the client; you build the project harness.
- **In the Loop, On the Loop, Above the Loop:** In the Loop: you sequence every step by hand. On the Loop: the harness sequences them and you supervise the result. Above the Loop: you manage fleets of specialized agents.
- **The Six Components (Know, Do, Verify):**
  - **Know - Rules:** Standing instructions the agent reads on every task (the agent-config file: `.cursorrules`, `AGENTS.md`, or equivalent).
  - **Know - Context Management:** Retrieval (architecture maps, API summaries) and memory (pruning stale context).
  - **Do - Tools:** Capabilities that give the LLM hands (running tests, reading files, calling APIs). Extensible via MCP.
  - **Do - Skills:** Reusable, named bundles of instructions the agent loads on demand. The process steps from earlier chapters packaged as repeatable procedures.
  - **Verify - Hooks:** Deterministic actions that fire automatically (pre-commit, post-edit). The agent cannot skip them.
  - **Verify - Verification Gates:** Linters, formatters, type checkers, and test suites that block code from landing until it passes.
- **Garbage Collection:** The deliberate removal of dead code, stale documentation, and misleading context. LLMs treat every file as absolute truth. Unpruned context fills the window with noise.
- **Harness Anti-Patterns:** Making the agent do a script's job (formatting), the overly restrictive harness (linter busywork), neglected context (stale architecture maps), and relying on review instead of building loops.

---

## Code Snippets

### Step 1: Initialize the Environment

```bash
mkdir python-harness-demo
cd python-harness-demo
python -m venv .venv
source .venv/bin/activate   # On Windows: .venv\Scripts\activate
pip install pytest mypy
```

### Step 2: The Context Map (`architecture.md`)

Create a file named `architecture.md` at the project root:

```markdown
# Application Architecture

## Purpose
This application is a simple command-line utility for managing an
inventory of books.

## Data Models
- A `Book` must have a `title` (string), an `author` (string), and a
`price` (float).
- The `price` must never be negative.

## Core Modules
- `models.py`: Contains the data structures (DataClasses or Pydantic
models).
- `inventory.py`: Contains the logic for adding books and calculating
total inventory value.
```

### Step 3: The Verification Layer (`test_inventory.py`)

Save as `test_inventory.py`. This is the gate the agent must pass:

```python
import pytest
from models import Book
from inventory import InventoryManager

def test_book_creation():
    book = Book(title="Agentic Engineering", author="Alex Merced", price=29.99)
    assert book.title == "Agentic Engineering"

def test_negative_price_raises_error():
    with pytest.raises(ValueError):
        Book(title="Bad Book", author="Nobody", price=-5.00)

def test_inventory_total_value():
    manager = InventoryManager()
    manager.add_book(Book(title="Book 1", author="Author A", price=10.0))
    manager.add_book(Book(title="Book 2", author="Author B", price=20.5))

    assert manager.calculate_total_value() == 30.5
```

### Step 4: The Rules File

Create your tool's rules file in the project root. The exact name depends on your agent (`.cursorrules`, `AGENTS.md`, `CLAUDE.md`, etc.). If unsure, save as `instructions.md` and include it in your prompt context:

```markdown
# Project Rules

You are operating within the python-harness-demo project. You must
strictly adhere to the following workflow on every task:

1. **Context First**: Before writing any code, you must read
`architecture.md` to understand the domain models.
2. **Strict Typing**: All Python code you generate must include strict
type hints. You must use the `dataclasses` module for data structures.
3. **Verification Loop**: Before you conclude any task or present code to
the user, you must run the following two commands in the terminal:
   - `mypy models.py inventory.py test_inventory.py`
   - `pytest test_inventory.py`
4. **Self-Correction**: If either `mypy` or `pytest` throws an error, you
must read the terminal output, fix the code autonomously, and rerun the
commands until both pass cleanly.
```

### Step 5: Execute On the Loop

With the harness in place (context map, tests, rules, verification tools), issue a single high-level prompt:

```
We need to implement the book inventory system. Please read the
architecture document, write the implementation in models.py and
inventory.py, and ensure the test suite passes completely. Do not stop
until mypy and pytest are perfectly green.
```

### What to Watch For

The agent should follow a constrained, mechanical process:

1. **Context Acquisition:** Reads the rules file, then reads `architecture.md`, then learns it must use DataClasses and that a `Book` requires `title`, `author`, and `price`.
2. **Initial Implementation:** Generates `models.py` and `inventory.py` with strict type hints.
3. **Automated Verification:** Runs `mypy models.py inventory.py test_inventory.py` and `pytest test_inventory.py`.
4. **Self-Correction:** If pytest shows `Failed: test_negative_price_raises_error`, the agent reads the red output, modifies `models.py` to add a `__post_init__` validation check, and reruns the tests.
5. **Final Output:** Tests turn green. The agent presents the verified implementation.

You did not write the code. You did not manually fix type errors. You built the harness, and the harness produced the software. This is working **On the Loop**.

### Running the Verification Gates Manually

```bash
# Run type checker
mypy models.py inventory.py test_inventory.py

# Run all tests
pytest test_inventory.py

# Run both together
mypy models.py inventory.py test_inventory.py && pytest test_inventory.py
```

---

## Supplemental Resources

### Pydantic & Data Modeling
- **[Pydantic v2 Documentation](https://docs.pydantic.dev/latest/)** - Complete documentation for Pydantic v2, including field validators, model config, and serialization.
- **[Python dataclasses (Official)](https://docs.python.org/3/library/dataclasses.html)** - The standard library module for data classes used in this chapter's walkthrough.

### Project Harness Tooling
- **[pre-commit Documentation](https://pre-commit.com/)** - The framework for managing and running pre-commit hooks, wiring verification gates to the Git lifecycle.
- **[Cursor Rules Documentation](https://docs.cursor.com/context/rules)** - How to configure `.cursorrules` files for persistent agent context.
- **[Claude Code AGENTS.md](https://docs.anthropic.com/en/docs/claude-code/overview)** - Claude Code reads `AGENTS.md` files at the repo root to establish persistent constraints.

### Architecture Documentation
- **[arc42 - Software Architecture Documentation](https://arc42.org/)** - A structured template for software architecture documentation. A good starting point for creating your `architecture.md`.
- **[C4 Model](https://c4model.com/)** - A lightweight, hierarchical approach to visualizing software architecture.

### Code Quality
- **[Ruff - Python Linter](https://docs.astral.sh/ruff/)** - A fast Python linter and formatter (replaces flake8, isort, black). Highly recommended as a harness layer alongside mypy.
- **[mypy Documentation](https://mypy.readthedocs.io/)** - Official documentation for mypy, Python's standard static type checker.
