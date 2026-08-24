# Chapter 5 - Closing Loops

## Chapter Summary

This chapter tackles the most dangerous pitfall of agentic software engineering: the illusion that speed of generation equals productivity. When agents can generate code faster than a team can verify it, the **Velocity Trap** springs. The chapter introduces **AI-TDD** (AI-driven Test-Driven Development), a workflow where tests are written *before* the implementation to steer the agent, and orders the full set of verification loops by latency, from the type checker firing at every keystroke to the CI pipeline that gates the merge.

The walkthrough executes a strict AI-TDD loop, building a **Slug Generator** utility in Python using `pytest` as the verification framework.

### Key Concepts

- **The Velocity Trap:** When generation outruns verification, technical debt accumulates exponentially. Manually reviewing AI-generated PRs line by line is a doomed strategy. The reviewer tires, skims, and approves code nobody understands.
- **The March of Nines:** The first 90% (the happy path demo) is cheap and fast. Enterprise production requires 99.99%+ reliability. Each additional nine costs roughly the same engineering effort. This is the same gap as the 70% Problem from Chapter 1, measured nine by nine.
- **AI-TDD (Inverted TDD):** The human owns the *test intent*, deciding which behaviors and edge cases the tests pin. The agent owns the *typing*, drafting assertions and writing the implementation. Red: failing tests. Green: agent makes them pass. Refactor: agent cleans up while keeping tests green.
- **The Dual Nature of a Test:** A test simultaneously provides *verification* (binary pass/fail proving behavior) and *design pressure* (forcing the implementation to the smallest change that turns the test green).
- **The Feedback-Latency Spectrum:** Verification loops ordered from fastest to slowest: Language Server, Linter, Compiler, Test, CI. Each catches a different class of fault. Faster loops are not more valuable; they are earlier.
- **Deterministic Gates:** Git hooks (pre-commit, pre-push) and agent hooks (pre-tool-use, post-edit) fire automatically and cannot be skipped by the agent. They turn a verification the agent might forget into one it does not control.
- **Acceptance Verification:** Proving the feature works for a human, not just that assertions pass. Re-runnable evidence artifacts (screenshots, demo documents) committed to the PR, not a log line that says "looks good."

---

## Code Snippets

### Setup: Python Environment

```bash
mkdir verification-exercise && cd verification-exercise
python -m venv .venv
source .venv/bin/activate   # On Windows: .venv\Scripts\activate
pip install pytest
touch utils.py test_utils.py
```

### The Test Suite (Write This First, Before Any Implementation)

Save as `test_utils.py`. These tests define the exact constraints of the Slug Generator:

```python
from utils import generate_slug

def test_generate_slug_basic():
    # It should lowercase and replace spaces with hyphens
    assert generate_slug("Hello World") == "hello-world"

def test_generate_slug_special_characters():
    # It should remove special characters completely
    assert generate_slug("Agentic Engineering: The Future!") == "agentic-engineering-the-future"

def test_generate_slug_multiple_spaces():
    # It should collapse multiple spaces into a single hyphen
    assert generate_slug("This    is   spaced") == "this-is-spaced"

def test_generate_slug_trailing_hyphens():
    # It should strip leading and trailing hyphens
    assert generate_slug("  Trim me please  ") == "trim-me-please"

def test_generate_slug_empty_string():
    # It should handle empty strings safely
    assert generate_slug("") == ""
```

### RED Phase: Prove the Tests Fail

```bash
pytest test_utils.py
```

You will see `ImportError: cannot import name 'generate_slug' from 'utils'`. This is exactly right. No implementation exists yet.

### GREEN Phase: The Implementation Prompt

Feed this to the agent:

```
Read the failing test suite in test_utils.py. Your task is to write the
implementation for 'generate_slug' in utils.py so that all tests pass
perfectly. You must use strict Python type hinting. Do not modify the
test file.
```

### Reference: Expected Implementation

The agent should produce something equivalent to this:

```python
import re

def generate_slug(text: str) -> str:
    if not text:
        return ""

    # Lowercase the string
    text = text.lower()

    # Remove all non-alphanumeric characters and spaces
    text = re.sub(r'[^a-z0-9\s-]', '', text)

    # Replace one or more spaces/hyphens with a single hyphen
    text = re.sub(r'[\s-]+', '-', text)

    # Strip leading and trailing hyphens
    return text.strip('-')
```

### Verify the Tests Pass

```bash
pytest test_utils.py
```

All 5 tests should pass green.

### REFACTOR Phase: The Refactor Prompt

```
The tests are passing. Now, refactor generate_slug in utils.py to be
more concise and performant. Attempt to chain the string methods if
possible, or combine the regex patterns. You must ensure the test suite
remains 100% green.
```

Run `pytest test_utils.py` one final time to confirm the refactor didn't break anything.

---

## Supplemental Resources

### Testing Frameworks
- **[pytest Documentation](https://docs.pytest.org/)** - The official pytest documentation. The most capable Python testing tool referenced throughout this chapter.
- **[pytest-cov](https://pytest-cov.readthedocs.io/)** - Coverage plugin for pytest. Use it to enforce minimum coverage thresholds in your CI gate.

### Static Type Checking
- **[mypy Documentation](https://mypy.readthedocs.io/)** - Official documentation for mypy, Python's standard static type checker.
- **[Python Type Hinting (PEP 484)](https://peps.python.org/pep-0484/)** - The foundational PEP for Python type annotations.
- **[TypeScript Strict Mode](https://www.typescriptlang.org/tsconfig#strict)** - The TypeScript equivalent of `mypy --strict`. Enabling all strict checks is the equivalent of a free verification layer for JavaScript projects.

### Test-Driven Development
- **[Test-Driven Development: By Example - Kent Beck](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530)** - The foundational text for TDD. The AI-TDD pattern in this chapter is a direct evolution of Beck's Red-Green-Refactor cycle.
- **[Growing Object-Oriented Software, Guided by Tests - Freeman & Pryce](http://www.growing-object-oriented-software.com/)** - The canonical text on letting tests drive software design. Directly relevant to the AI-TDD workflow.

### Acceptance Verification
- **[Showboat and Rodney - Simon Willison](https://simonwillison.net/)** - Tools that let an agent produce reproducible demo artifacts (replayable Markdown demos and browser-driven screenshots) so acceptance verification rests on evidence the tooling captures directly.

### Verification at Scale
- **[pre-commit Framework](https://pre-commit.com/)** - The tool for managing and running pre-commit hooks. Use it to automate verification on every commit.
- **[GitHub Actions - Quickstart](https://docs.github.com/en/actions/writing-workflows/quickstart)** - How to run your verification suite in CI. The verification gate is only complete when it runs automatically in your pipeline.
