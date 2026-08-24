# Chapter 7 - The Autonomy Spectrum

## Chapter Summary

Autonomy is not a binary switch. This chapter introduces the **Agentic Engineering Maturity Model (AEMM)**, a five-level framework for calibrating exactly how much freedom you grant an agent based on the risk of the task, the quality of the context, and the strength of the harness. It maps the SAE levels of driving automation to software development, distinguishes **agency** (the capacity to act) from **autonomy** (what the actor does on its own), and introduces the **Elastic Loop** framework for per-task calibration.

The walkthrough builds a **coverage-gated enforcement script** using `pytest-cov` that physically prevents an agent from committing code that drops test coverage below 100%, demonstrating that technical enforcement beats policy.

### Key Concepts

- **The SAE Analogy:** The Society of Automotive Engineers' six levels of driving automation (L0-L5) map cleanly to software development, from no automation to a fully autonomous pipeline. Each level describes what the machine does, not a ranking.
- **The Agentic Engineering Maturity Model (AEMM):** Five levels of engineering maturity:
  - **Level 1 - Ad-Hoc Adopter:** Isolated browser use, high friction, no harness.
  - **Level 2 - Systematic Collaborator:** Agent integrated in the IDE. Human approves every line.
  - **Level 3 - Agent Manager:** Developer provides specifications and reviews PRs. Significant autonomy, supervised loop.
  - **Level 4 - Harness Engineer:** Agent runs autonomously within a fully harnessed repository, self-correcting via verification gates. Developer reviews exceptions, not every line.
  - **Level 5 - Fleet Commander:** Developer orchestrates multiple parallel agents. Attention is the scarcest resource. The peak an individual engineer can reach.
- **Agency vs. Autonomy:** Agency is the capacity to act (competence + authority + information). Autonomy is what the actor does on its own. The dangerous case: high autonomy with low agency. The golden rule: **autonomy must never exceed the structural capacity of the harness to verify it.**
- **The Elastic Loop:** The agent's execution loop *expands* (takes on more task scope) and *contracts* (self-corrects when verification gates fail) based on risk and context quality. The developer dials this per task.
- **Technical Enforcement Beats Policy:** A PDF policy document does not bind a machine. A `pytest-cov --cov-fail-under=100` gate physically rejects code that violates the standard. Technical enforcement carries the load.

---

## Code Snippets

### The AEMM Quick Reference

| Level | Name | Agent Role | Human Role | Loop Type |
|---|---|---|---|---|
| 1 | Ad-Hoc Adopter | Answering questions | Full author | No loop |
| 2 | Systematic Collaborator | Inline autocomplete | Approves every line | Micro-loop |
| 3 | Agent Manager | Generates full features | Reviews PRs | Supervised loop |
| 4 | Harness Engineer | Implements + self-corrects | Reviews exceptions | Tight loop |
| 5 | Fleet Commander | Orchestrates parallel agents | Monitors dashboard | Multi-agent loop |

### Step 1: Initialize the Rigid Environment

```bash
mkdir autonomy-boundary-demo
cd autonomy-boundary-demo
python -m venv .venv
source .venv/bin/activate   # On Windows: .venv\Scripts\activate
pip install pytest pytest-cov
```

### Step 2: Establish the Baseline Codebase

Create `calculator.py`, a simple, fully-tested module:

```python
def add(a: int, b: int) -> int:
    """Returns the sum of two integers."""
    return a + b
```

Create `test_calculator.py`:

```python
from calculator import add

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
```

### Step 3: Implement the Technical Enforcement Script

Create `verify_autonomy.py`, a coverage-gated enforcement boundary:

```python
import subprocess
import sys

def enforce_coverage_boundary():
    print("Executing Autonomy Verification Gate...")

    # Run pytest with the coverage flag, requiring 100% coverage
    # The --cov-fail-under=100 flag is the physical boundary.
    result = subprocess.run(
        ["pytest", "--cov=calculator", "--cov-fail-under=100", "test_calculator.py"],
        capture_output=True,
        text=True
    )

    if result.returncode == 0:
        print("[OK] Autonomy Gate Passed: 100% Coverage Maintained.")
        print("Code is permitted to proceed.")
        sys.exit(0)
    else:
        print("[FAIL] Autonomy Gate FAILED: Test coverage dropped below 100%.")
        print("The agent's code has been rejected by the Project Harness.")
        print("\nCoverage Report Details:")

        # Print the last few lines of the pytest output showing the coverage failure
        output_lines = result.stdout.split('\n')
        for line in output_lines[-10:]:
            print(line)

        sys.exit(1)

if __name__ == "__main__":
    enforce_coverage_boundary()
```

Verify the baseline passes:

```bash
python verify_autonomy.py
```

Expected: `[OK] Autonomy Gate Passed: 100% Coverage Maintained.`

### Step 4: Test the Boundary Against a Rogue Agent

Prompt the agent to add a feature **without tests**, simulating a developer rushing a feature out:

```
Add a new function to calculator.py that handles multiplication. Do NOT
write any tests for it. Just add the function to the main file.
```

The agent will eagerly comply. `calculator.py` becomes:

```python
def add(a: int, b: int) -> int:
    """Returns the sum of two integers."""
    return a + b

def multiply(a: int, b: int) -> int:
    """Returns the product of two integers."""
    return a * b
```

### Step 5: The Harness Rejects the Code

Run the enforcement script:

```bash
python verify_autonomy.py
```

Expected output:

```
Executing Autonomy Verification Gate...
[FAIL] Autonomy Gate FAILED: Test coverage dropped below 100%.
The agent's code has been rejected by the Project Harness.

Coverage Report Details:
----------- coverage: platform linux, python 3.10.x-final-0 -----------
Name            Stmts   Miss  Cover
-----------------------------------
calculator.py       4      1    75%
-----------------------------------
TOTAL               4      1    75%

FAIL Required test coverage of 100% not reached. Total coverage: 75.00%
```

### Closing the Loop

Copy the failure output, paste it back to the agent, and say: **"Fix this."**

The agent reads the coverage failure, recognizes it needs tests for `multiply()`, writes the missing tests in `test_calculator.py`, and resubmits. Run the gate again and it passes.

This is bounded autonomy in action. The rules of the repository rejected the code, not a human remembering to check.

### Autonomy Calibration Decision Matrix

Use this table to determine the appropriate AEMM level before starting a task:

| Risk Level | Context Quality | Harness Strength | Recommended Level |
|---|---|---|---|
| Low | Any | Any | Level 3-4 |
| Medium | High | Strong | Level 4 |
| Medium | Low | Any | Level 2-3 |
| High | High | Strong | Level 3 (supervised) |
| High | Any | Weak | Level 1-2 only |
| Production DB migration | Any | Any | Level 2 + human approval gate |

---

## Supplemental Resources

### AEMM and Autonomy Frameworks
- **[Building Effective Agents - Anthropic](https://www.anthropic.com/research/building-effective-agents)** - Anthropic's framework for categorizing agents by their level of autonomy (workflows vs. agents).
- **[The Five Levels: from Spicy Autocomplete to the Dark Factory - Dan Shapiro](https://danshapiro.com/)** - A driving-automation-modeled L0-L5 maturity ladder that runs parallel to the AEMM.
- **[The Self-Driving Codebase - Ona (background-agents.com)](https://background-agents.com/)** - On in-loop vs. on-loop operation, fleets and swarms, and the false summit.

### DORA Metrics (For Autonomy Calibration)
- **[DORA Metrics - dora.dev](https://dora.dev/)** - The research program run by Google Cloud, for the definitions of the delivery metrics and the evidence behind them.
- **[Accelerate - Nicole Forsgren et al.](https://itrevolution.com/product/accelerate/)** - The book that introduced DORA metrics as the gold standard for engineering productivity measurement.

### Technical Enforcement
- **[pre-commit Framework](https://pre-commit.com/)** - Managing and running pre-commit hooks to enforce verification locally before code is pushed.
- **[GitHub Actions - Environments and Required Reviewers](https://docs.github.com/en/actions)** - The concrete mechanism behind asynchronous approval gates: a deployment that stops and waits for a named human before it proceeds.
- **[pytest-cov Documentation](https://pytest-cov.readthedocs.io/)** - The coverage plugin used in this chapter's enforcement script.
