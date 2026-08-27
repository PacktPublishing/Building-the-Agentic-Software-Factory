# Chapter 7 - The Autonomy Spectrum

## Chapter Summary

Autonomy is not a binary switch. This chapter introduces the **Agentic Engineering Maturity Model (AEMM)**, a five-level framework for calibrating exactly how much freedom you grant an agent based on the risk of the task, the quality of the context, and the strength of the harness. It maps the SAE levels of driving automation to software development, distinguishes **agency** (the capacity to act) from **autonomy** (what the actor does on its own), and introduces the **Elastic Loop** framework for per-task calibration.

The walkthrough builds a **GitHub Actions CI pipeline** that enforces 100% test coverage on every push. An agent pushes untested code, the pipeline rejects it, the agent reads the CI failure and self-corrects. The loop closes without a human in the middle.

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
- **Technical Enforcement Beats Policy:** A PDF policy document does not bind a machine. A CI pipeline with `--cov-fail-under=100` physically rejects code that violates the standard. Technical enforcement carries the load.

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

---

## Primary Exercise: CI-Enforced Coverage Gate

This exercise builds a real GitHub Actions pipeline that rejects untested code. The agent reads the CI failure and fixes the problem without human intervention. This is the closed-loop autonomy the chapter describes.

### Step 1: Initialize the Repository

```bash
mkdir autonomy-boundary-demo
cd autonomy-boundary-demo
git init
```

Create `calculator.py`, the baseline module:

```python
def add(a: int, b: int) -> int:
    """Returns the sum of two integers."""
    return a + b
```

Create `test_calculator.py`, the tests that pin the baseline at 100% coverage:

```python
from calculator import add

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
```

Create `requirements.txt`:

```
pytest
pytest-cov
```

### Step 2: Add the GitHub Actions Coverage Gate

This is the structural boundary. Create `.github/workflows/coverage-gate.yml`:

```yaml
name: Coverage Gate

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  coverage:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests with coverage enforcement
        run: |
          pytest --cov=calculator --cov-fail-under=100 test_calculator.py
```

The `--cov-fail-under=100` flag is the physical boundary. If any function in `calculator.py` is not covered by a test, pytest exits with a non-zero code, the workflow step fails, and the pipeline blocks the merge.

### Step 3: Push the Green Baseline

Connect the local repo to a GitHub repository and push:

```bash
git add .
git commit -m "Baseline: calculator with 100% coverage"
git remote add origin https://github.com/YOUR_USERNAME/autonomy-boundary-demo.git
git push -u origin main
```

Open the **Actions** tab in your GitHub repository. The pipeline runs and passes:

```
Run pytest --cov=calculator --cov-fail-under=100 test_calculator.py

test_calculator.py ..                                            [100%]

----------- coverage: platform linux, python 3.11.x -----------
Name            Stmts   Miss  Cover
-----------------------------------
calculator.py       2      0   100%
-----------------------------------
TOTAL               2      0   100%

Required test coverage of 100% reached. Total coverage: 100.00%
```

The baseline is locked. Every future push must maintain 100% coverage or the pipeline rejects it.

### Step 4: Prompt the Agent to Break the Gate

Open your agentic client and issue this prompt:

```
Add a new function to calculator.py that handles multiplication.
Do NOT write any tests for it. Just add the function to the main file.
Then commit and push the change.
```

The agent complies. It updates `calculator.py`:

```python
def add(a: int, b: int) -> int:
    """Returns the sum of two integers."""
    return a + b

def multiply(a: int, b: int) -> int:
    """Returns the product of two integers."""
    return a * b
```

It commits and pushes. The code reaches GitHub.

### Step 5: The Pipeline Rejects the Code

Open the **Actions** tab. The pipeline fails:

```
Run pytest --cov=calculator --cov-fail-under=100 test_calculator.py

test_calculator.py ..                                            [100%]

----------- coverage: platform linux, python 3.11.x -----------
Name            Stmts   Miss  Cover
-----------------------------------
calculator.py       4      1    75%
-----------------------------------
TOTAL               4      1    75%

FAIL Required test coverage of 100% not reached. Total coverage: 75.00%

Error: Process completed with exit code 2.
```

The push landed, but the pipeline is red. In a real project with branch protection rules enabled, this commit would be blocked from merging into the protected branch entirely.

### Step 6: The Agent Reads CI and Self-Corrects

This is where the exercise diverges from the local version. Instead of a human copying the error output and pasting it back to the agent, the agent reads the CI failure directly.

**With Claude Code** (or any CLI agent with shell access):

```
The CI pipeline failed on the last push. Run `gh run view --log-failed`
to read the failure, then fix the problem and push the corrected code.
```

**With Cursor, Windsurf, or other IDE agents:**

```
The GitHub Actions coverage gate failed on the last push. The pipeline
reported 75% coverage because multiply() has no tests. Write tests for
the multiply function in test_calculator.py, then commit and push.
```

The agent writes the missing tests:

```python
from calculator import add, multiply

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0

def test_multiply():
    assert multiply(3, 4) == 12
    assert multiply(-2, 5) == -10
    assert multiply(0, 100) == 0
```

It commits and pushes. The pipeline runs again. This time it passes:

```
----------- coverage: platform linux, python 3.11.x -----------
Name            Stmts   Miss  Cover
-----------------------------------
calculator.py       4      0   100%
-----------------------------------
TOTAL               4      0   100%

Required test coverage of 100% reached. Total coverage: 100.00%
```

The loop closed. No human read the error. No human decided what to fix. The CI pipeline produced the structured feedback, the agent consumed it, and the boundary held. This is AEMM Level 4 in practice: the Harness Engineer built the pipeline, the agent operated within it, and the developer reviewed the result after the fact.

### Adding Branch Protection (Production Hardening)

To make the gate truly blocking (not just a red check that can be ignored), enable branch protection rules in your GitHub repository:

1. Go to **Settings > Branches > Add rule**
2. Set **Branch name pattern** to `main`
3. Check **Require status checks to pass before merging**
4. Select the `coverage` job from the list
5. Check **Require branches to be up to date before merging**

With this configuration, no code can be merged into `main` unless the coverage gate passes. The agent must push to a feature branch and open a pull request. The pipeline runs against the PR, and the merge button stays disabled until the check is green.

---

## Local Fallback: `verify_autonomy.py`

If you do not have a GitHub account or want to run the exercise entirely offline, use this local enforcement script as a substitute for the CI pipeline.

Create `verify_autonomy.py`:

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

        output_lines = result.stdout.split('\n')
        for line in output_lines[-10:]:
            print(line)

        sys.exit(1)

if __name__ == "__main__":
    enforce_coverage_boundary()
```

Run `python verify_autonomy.py` after each change to simulate the CI gate locally. The same coverage logic applies: if coverage drops below 100%, the script exits with a non-zero code and the agent's work is rejected.

The difference: with the local script, you copy the error output and paste it to the agent. With the CI pipeline, the agent reads the failure itself and closes the loop without you.

---

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
- **[GitHub Actions Documentation](https://docs.github.com/en/actions)** - The official reference for building CI/CD pipelines. Start here to extend the coverage gate with linting, type checking, or security scanning.
- **[GitHub Branch Protection Rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-a-branch-protection-rule)** - How to enforce required status checks before merging, making the coverage gate truly blocking.
- **[GitHub CLI (`gh`)](https://cli.github.com/)** - The command-line tool that lets agents read CI logs programmatically via `gh run view --log-failed`.
- **[pre-commit Framework](https://pre-commit.com/)** - Local pre-commit hooks for enforcing verification before code is pushed.
- **[pytest-cov Documentation](https://pytest-cov.readthedocs.io/)** - The coverage plugin used in both the CI pipeline and the local fallback script.
