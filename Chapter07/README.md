# Chapter 7 - The Autonomy Spectrum

## Chapter Summary

Autonomy is not a binary switch. This chapter introduces the **Agentic Engineering Maturity Model (AEMM)** - a five-level framework for calibrating exactly how much freedom you grant an agent based on the risk of the task, the quality of the context, and the strength of the harness. Granting too little autonomy wastes the agent's potential; granting too much creates a liability.

The walkthrough builds a Python `verify_autonomy.py` script that acts as a pre-flight check, assessing whether a given task and harness configuration are ready for a higher level of autonomy.

### Key Concepts

- **The Agentic Engineering Maturity Model (AEMM):** Five levels of agent autonomy:
  - **Level 1 - Ad-hoc Adopter:** The agent is used for one-off questions and code snippets. No harness. No systematic workflow.
  - **Level 2 - Integrated Assistant:** The agent is integrated into the IDE (e.g., Copilot autocomplete). Passive assistance. The developer drives every decision.
  - **Level 3 - Loose Loop:** The developer provides high-level specifications and reviews agent-generated PRs. Significant autonomy, but the developer checks all outputs before merging.
  - **Level 4 - Tight Loop:** The agent runs autonomously within a fully harnessed repository, self-correcting via test/compiler feedback. The developer reviews exceptions, not every line.
  - **Level 5 - Code Multiplier:** The developer orchestrates multiple parallel Loose/Tight Loop agents simultaneously. Attention is the developer's scarcest resource.

- **The Elastic Loop:** At higher AEMM levels, the agent's execution loop *expands* (takes on more task scope) and *contracts* (self-corrects when verification gates fail), all without human intervention.
- **Autonomy Calibration:** The correct AEMM level for a given task is determined by three factors - *Risk* (what breaks if the agent is wrong?), *Context Quality* (how good is the information architecture?), and *Harness Strength* (how strong are the verification gates?).
- **Tightening the Loop:** If the Change Failure Rate rises, you tighten the loop (reduce autonomy, require more human review). If velocity is too low and the harness is strong, you loosen the loop (increase autonomy).

---

## Code Snippets

### The AEMM Quick Reference

| Level | Name | Agent Role | Human Role | Loop Type |
|---|---|---|---|---|
| 1 | Ad-hoc Adopter | Answering questions | Full author | No loop |
| 2 | Integrated Assistant | Inline autocomplete | Approves every line | Micro-loop |
| 3 | Loose Loop | Generates full features | Reviews PRs | Supervised loop |
| 4 | Tight Loop | Implements + self-corrects | Reviews exceptions | Tight loop |
| 5 | Code Multiplier | Orchestrates parallel agents | Monitors dashboard | Multi-agent loop |

### Autonomy Pre-Flight Check Script

Save as `verify_autonomy.py`. Run this before elevating to a higher AEMM level on a task:

```python
#!/usr/bin/env python3
"""
verify_autonomy.py - Pre-flight check before granting a higher AEMM level.

Assesses whether the current task and harness are ready for elevated autonomy.
Exit code 0 = cleared for elevation. Exit code 1 = not ready.
"""
import subprocess
import sys
import os
from pathlib import Path


def check_architecture_doc() -> tuple[bool, str]:
    """Verify that an architecture.md or equivalent context document exists."""
    candidates = ["architecture.md", "AGENTS.md", ".cursorrules", "docs/architecture.md"]
    for candidate in candidates:
        if Path(candidate).exists():
            return True, f"PASS: Context document found: {candidate}"
    return False, "FAIL: No context document found (architecture.md, AGENTS.md, or .cursorrules)"


def check_test_suite() -> tuple[bool, str]:
    """Verify that a test suite exists and passes."""
    result = subprocess.run(
        ["pytest", "--tb=no", "-q"],
        capture_output=True,
        text=True
    )
    if result.returncode == 0:
        return True, f"PASS: Test suite passes: {result.stdout.strip().splitlines()[-1]}"
    return False, f"FAIL: Test suite failing:\n{result.stdout[-300:]}"


def check_type_annotations() -> tuple[bool, str]:
    """Verify mypy strict type checking passes."""
    result = subprocess.run(
        ["mypy", "src/", "--strict", "--ignore-missing-imports"],
        capture_output=True,
        text=True
    )
    if result.returncode == 0:
        return True, "PASS: mypy --strict passes with zero errors"
    error_count = result.stdout.count("error:")
    return False, f"FAIL: mypy found {error_count} type error(s). Fix before elevating autonomy."


def check_pre_commit_hooks() -> tuple[bool, str]:
    """Verify that pre-commit hooks are configured."""
    if Path(".pre-commit-config.yaml").exists():
        hook_dir = Path(".git/hooks/pre-commit")
        if hook_dir.exists():
            return True, "PASS: pre-commit hooks installed and configured"
        return False, "FAIL: .pre-commit-config.yaml exists but hooks not installed. Run: pre-commit install"
    return False, "FAIL: No .pre-commit-config.yaml found. Verification gate not installed."


def check_gitignore_secrets() -> tuple[bool, str]:
    """Verify .env files are gitignored."""
    gitignore = Path(".gitignore")
    if not gitignore.exists():
        return False, "FAIL: No .gitignore file found."
    content = gitignore.read_text()
    if ".env" in content:
        return True, "PASS: .env files are gitignored"
    return False, "FAIL: .env not found in .gitignore. Secrets may be exposed."


def main() -> None:
    checks = [
        check_architecture_doc,
        check_test_suite,
        check_type_annotations,
        check_pre_commit_hooks,
        check_gitignore_secrets,
    ]

    print("=" * 60)
    print("  AEMM Autonomy Pre-Flight Check")
    print("=" * 60)

    passed = 0
    failed = 0
    for check in checks:
        ok, message = check()
        print(f"\n{message}")
        if ok:
            passed += 1
        else:
            failed += 1

    print("\n" + "=" * 60)
    print(f"  Result: {passed} passed / {failed} failed")
    print("=" * 60)

    if failed == 0:
        print("\nCLEARED: Harness is ready for elevated autonomy (AEMM Level 4+).")
        sys.exit(0)
    else:
        print(f"\nNOT READY: Resolve {failed} issue(s) before elevating autonomy.")
        print("   Granting high autonomy to an unharnessed agent increases blast radius.")
        sys.exit(1)


if __name__ == "__main__":
    main()
```

### Running the Pre-Flight Check

```bash
# Make executable
chmod +x verify_autonomy.py

# Run before a high-autonomy task
python verify_autonomy.py
```

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
- **[Building Effective Agents - Anthropic](https://www.anthropic.com/research/building-effective-agents)** - Anthropic's framework for categorizing agents by their level of autonomy (workflows vs. agents). Directly maps to the AEMM concept.
- **[Agent Levels of Autonomy - LangChain Blog](https://blog.langchain.dev/)** - LangChain's perspective on how to categorize and manage agent autonomy in production systems.

### DORA Metrics (For Autonomy Calibration)
- **[DORA Metrics - Google Cloud](https://cloud.google.com/blog/products/devops-sre/using-the-four-keys-to-measure-your-devops-performance)** - The four DORA metrics used throughout the book to calibrate autonomy: Deployment Frequency, Lead Time, Change Failure Rate, MTTR. A rising Change Failure Rate is the signal to tighten the loop.
- **[Accelerate - Nicole Forsgren et al.](https://itrevolution.com/product/accelerate/)** - The book that introduced DORA metrics as the gold standard for engineering productivity measurement.

### Safety and Guardrails
- **[AI Safety - Anthropic Constitutional AI](https://www.anthropic.com/research/constitutional-ai-harmlessness-from-ai-feedback)** - How model-level safety constraints interact with your harness-level safety constraints. Understanding the model's built-in guardrails helps you calibrate what the harness needs to add.
- **[Responsible AI Practices - Google](https://ai.google/responsibility/responsible-ai-practices/)** - Google's framework for evaluating AI risk before deployment, applicable to agentic workflow risk assessment.
