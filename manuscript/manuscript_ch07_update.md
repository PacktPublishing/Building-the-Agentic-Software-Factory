# Chapter 7 - Manuscript Update Recommendations

This document describes the changes needed in the Chapter 7 manuscript to replace the local `verify_autonomy.py` walkthrough with a GitHub Actions CI pipeline exercise.

---

## Problem

The current walkthrough builds a local Python script (`verify_autonomy.py`) that wraps `pytest-cov`. The manuscript itself acknowledges on line 538-539 that this is a simplified stand-in: "In a corporate environment, this would be a GitHub Action or a GitLab CI pipeline step. For local demonstration purposes, we will create a mock pre-commit script."

Reviewer feedback (Jannik): the exercise is too basic. The chapter is about the Autonomy Spectrum and calibrating how much freedom to give an agent. A local script that requires the human to copy error output and paste it back to the agent is a Level 2 (Systematic Collaborator) exercise. The chapter should demonstrate Level 4 (Harness Engineer): the agent pushes, the CI rejects, the agent reads the CI output, and the loop closes without a human in the middle.

---

## Recommended Approach

Replace the local script walkthrough with a GitHub Actions pipeline. Keep the same baseline code (`calculator.py`, `test_calculator.py`). Keep the local script as a clearly labeled fallback for readers without GitHub access.

---

## REPLACEMENT COPY BEGINS HERE

### Replace the current walkthrough section (from "Walkthrough" heading through Step 5 + the closing paragraph)

### Walkthrough - Designing technically enforced autonomy boundaries

To demonstrate the power of technical enforcement, we will build a rigid boundary using a real CI pipeline. We will create a GitHub Actions workflow that enforces a strict 100% test coverage rule on every push. We will then prompt the agent to write a new feature without writing tests, watch the pipeline reject it, and observe the agent read the CI failure and fix the problem on its own.

The local script version of this exercise appears in the companion repository for readers without GitHub access. The CI version is the one worth building.

#### Step 1 - Initialize the repository

First, we will set up a new Python project with its testing infrastructure:

1. Open your terminal and create a new directory for the autonomy walkthrough:

```
mkdir autonomy-boundary-demo
cd autonomy-boundary-demo
git init
```

2. Create a file named `calculator.py` with a simple addition function:

```python
def add(a: int, b: int) -> int:
    """Returns the sum of two integers."""
    return a + b
```

3. Create the corresponding test file named `test_calculator.py`:

```python
from calculator import add

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
```

4. Create a `requirements.txt` file:

```
pytest
pytest-cov
```

#### Step 2 - Build the CI enforcement gate

We will now build the enforcement mechanism as a GitHub Actions workflow. Unlike the local script in the companion repository, this gate runs on GitHub's infrastructure every time code is pushed. The developer does not have to remember to run it. The agent cannot skip it.

1. Create the directory structure for the workflow:

```
mkdir -p .github/workflows
```

2. Create a file named `.github/workflows/coverage-gate.yml` and add the following:

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

#### Step 3 - Push the green baseline

Connect the local repository to a GitHub repository and push:

```
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

#### Step 4 - Test the boundary against a rogue agent

Now, we will test the strength of our technical enforcement. We will instruct our agent to add a new feature, but we will explicitly mimic a developer trying to rush a feature out the door without writing tests.

Open your agentic client (Cursor, Claude Code, etc.) and issue the following prompt:

```
Add a new function to calculator.py that handles multiplication.
Do NOT write any tests for it. Just add the function to the main file.
Then commit and push the change.
```

The agent will eagerly comply. It will update `calculator.py` to look like this:

```python
def add(a: int, b: int) -> int:
    """Returns the sum of two integers."""
    return a + b

def multiply(a: int, b: int) -> int:
    """Returns the product of two integers."""
    return a * b
```

The agent has written the code, committed it, and pushed it to GitHub. In an organization operating at AEMM Level 2 (Systematic Collaborator) with no CI pipeline, this untested code would sail through on sight, adding one more unverified function to a codebase no human has fully read.

#### Step 5 - The pipeline rejects the code

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

The push landed, but the pipeline is red. With branch protection rules enabled (covered below), this commit would be blocked from merging into the protected branch entirely.

#### Step 6 - The agent reads CI and self-corrects

This is the step that separates a Level 2 exercise from a Level 4 exercise. Instead of the developer copying the error output and pasting it back to the agent, the agent reads the CI failure directly and closes the loop on its own.

If you are using **Claude Code** or another CLI agent with shell access, prompt:

```
The CI pipeline failed on the last push. Run `gh run view --log-failed`
to read the failure output, then fix the problem and push the corrected
code.
```

If you are using **Cursor, Windsurf**, or another IDE agent, you can paste the CI output or point the agent at the Actions URL:

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

The loop closed. The CI pipeline produced the structured feedback, the agent consumed it, and the boundary held. This is AEMM Level 4 in action: the Harness Engineer built the pipeline, the agent operated within it, and the developer reviewed the result after the fact rather than supervising every step.

To harden this further, enable branch protection rules in your GitHub repository settings. Under **Settings > Branches > Add rule**, set the branch name pattern to `main`, check **Require status checks to pass before merging**, select the `coverage` job, and check **Require branches to be up to date before merging**. With this configuration, the agent must push to a feature branch and open a pull request. The merge button stays disabled until the coverage gate is green.

## REPLACEMENT COPY ENDS HERE

---

## Summary of Changes

### What stays in the manuscript
- The chapter introduction, all conceptual material (SAE, AEMM, Agency vs. Autonomy, Elastic Loop, Technical Enforcement Beats Policy)
- The "Questions" and "Further Reading" sections at the end of the chapter

### What changes
1. Replace Steps 1-5 of the walkthrough with the CI pipeline version (Steps 1-6 above)
2. Remove the `verify_autonomy.py` code listing (it moves to the companion repo as a labeled local fallback)
3. Replace the sentence on line 538-539 ("In a corporate environment, this would be a GitHub Action or a GitLab CI pipeline step. For local demonstration purposes, we will create a mock pre-commit script") with the actual CI version
4. Update the closing paragraph (line 619-624) to reflect that the loop closes without the human copying and pasting error output

### Suggested closing paragraph replacement

Replace:

> This is the essence of Harness Engineering and bounded autonomy. You did not have to rely on a human developer remembering to read a policy document. You did not have to review the PR by hand to notice the missing test. The rules of the repository rejected the code. To complete the task, you copy the red failure output, paste it back to the agent, and say: Fix this. The agent reads the coverage failure, recognizes its mistake, writes the missing tests in test_calculator.py, and resubmits the code. The loop closes on its own.

With:

> This is the essence of Harness Engineering and bounded autonomy. You did not have to rely on a human developer remembering to read a policy document. You did not have to review the PR by hand to notice the missing test. The rules of the repository rejected the code. The CI pipeline produced structured feedback. The agent read that feedback, recognized the gap, wrote the missing tests, and pushed corrected code. The loop closed without a human in the middle. That is the difference between Level 2 and Level 4 on the AEMM: at Level 2, the developer is the feedback conduit; at Level 4, the pipeline is.

### Suggested Summary bullet update

Replace:

> Finally, we built a Python walkthrough using pytest-cov, showing how a rigid coverage boundary rejects agent-generated code that fails the harness's coded verification standard.

With:

> Finally, we built a GitHub Actions pipeline using pytest-cov that enforces 100% test coverage on every push. When the agent pushed untested code, the pipeline rejected it. The agent read the CI failure, wrote the missing tests, and pushed corrected code without the developer pasting error output back into the chat. The loop closed on its own.

---

## Companion Repo Changes (Already Applied)

The companion repo (`Chapter07/README.md`) has been updated with:

1. **Primary Exercise: CI Pipeline** - GitHub Actions workflow with `--cov-fail-under=100`, agent pushes untested code, pipeline rejects, agent reads CI and self-corrects.
2. **Local Fallback** - The `verify_autonomy.py` script preserved with clear labeling as the offline/simplified version.
3. **Branch Protection** - Instructions for hardening the gate with required status checks.
