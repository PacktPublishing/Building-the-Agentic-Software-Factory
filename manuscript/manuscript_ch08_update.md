# Chapter 8 - Manuscript Update Recommendations

This document describes the changes needed in the Chapter 8 manuscript to align with the updated companion repo exercise.

---

## Problem

The current walkthrough ("Step 4 - Test the boundary against a compromised agent") asks the reader to prompt an agent to hardcode a Stripe key into `payment_gateway.py`. Modern agents consistently refuse to do this:

- **Claude Opus** warns about logging the token and refuses to hardcode it.
- **Claude Sonnet** reaches for `os.environ.get` instead of copying the literal.
- Neither reviewer (Jannik, second reviewer) could reproduce the vulnerability.

The manuscript already acknowledges this at line 582-586 of the PDF text:

> "Whether yours does is not guaranteed, and that is worth watching: current models often refuse, warn that the token would be logged, or reach for os.environ.get instead of copying the literal. Treat a refusal as the interesting result rather than a failed exercise."

However, the exercise remains unreproducible as a hands-on walkthrough.

---

## Recommended Approach

Keep the existing secret-scanner walkthrough as a **teaching illustration** of the concept, but add a new **PII Scanner** exercise as the primary hands-on walkthrough. This exercise is fully reproducible because it does not depend on the agent misbehaving. The PII is seeded by the reader.

### Insert Before Step 4 (Callout Box)

Insert the following callout before the current Step 4:

---

> **A note on reproducibility**
>
> Current frontier models (Claude, GPT-4, Gemini) often refuse this prompt, warning that hardcoding a secret is a security risk, or silently reaching for `os.environ.get` instead. If your agent refuses, treat the refusal as the interesting result: it tells you the model's own guardrail caught this specific case. It tells you nothing about the next prompt. A differently worded request, a different model version, or a less obvious secret may slip through. That is exactly why the boundary below is built outside the model.
>
> For a fully reproducible hands-on exercise that demonstrates the same structural defense without depending on agent behavior, see the PII Scanner exercise that follows.

---

### New Section: Insert After Current Walkthrough

Add the following as a new walkthrough section after the secret scanner walkthrough ends. This is the primary hands-on exercise.

---

## REPLACEMENT COPY BEGINS HERE

### Walkthrough 2 - Building a PII approval gate

The secret scanner above demonstrates the structural defense, but its test scenario depends on the agent cooperating with a bad prompt. That makes it hard to reproduce. This second walkthrough removes that dependency entirely. Instead of hoping the agent leaks a key, you seed realistic personally identifiable information (PII) across your own project files and build a pre-commit gate that catches the violations before they reach version control.

PII leakage is a common and serious risk in agentic workflows. An agent trained on public data may generate realistic Social Security numbers, credit card numbers, or email addresses as placeholder values. It may also copy real PII from context files (customer exports, seed data, test fixtures) into generated code. Unlike a hardcoded API key, PII violations carry regulatory consequences under GDPR, CCPA, and HIPAA, making a structural gate even more important than a policy.

#### Step 1 - Initialize the project

Open your terminal and create a new directory for the PII walkthrough:

```
mkdir pii-scanner-demo
cd pii-scanner-demo
git init
```

#### Step 2 - Seed PII across project files

Create several files with realistic PII embedded in them. These simulate what an agent might generate from training data or context leakage.

Create a file named `config.py`:

```python
"""Application configuration."""

DATABASE_URL = "postgresql://admin:password@localhost:5432/appdb"

# Customer support contact for escalation
SUPPORT_EMAIL = "support@example.com"
SUPPORT_PHONE = "555-0100"

# Test credentials for the staging environment
# TODO: move to environment variables before launch
ADMIN_SSN = "123-45-6789"
TEST_CREDIT_CARD = "4111-1111-1111-1111"
```

Create a file named `seed_data.json`:

```json
{
  "test_users": [
    {
      "name": "Jane Doe",
      "email": "jane.doe@testcorp.com",
      "ssn": "987-65-4321",
      "phone": "555-0199"
    },
    {
      "name": "John Smith",
      "email": "john.smith@testcorp.com",
      "credit_card": "5500-0000-0000-0004",
      "cvv": "123"
    }
  ]
}
```

Create a file named `README.md`:

```markdown
# My Application

Contact the team lead at sarah.connor@realcompany.com for access.
The test account SSN is 111-22-3333.
```

None of these files look alarming at first glance. A developer skimming a pull request might approve every one of them. That is the point: the gate has to catch what human review misses under fatigue.

#### Step 3 - Build the PII scanner

We will build a custom Python script that scans all staged files for common PII patterns. In a production environment, you would use a hardened tool like `detect-secrets` or `trufflehog` with custom rules. Here we build a simple version to show what the gate does mechanically.

Create a file named `scan_pii.py` and add the following logic:

```python
import os
import re
import subprocess
import sys


# PII patterns to detect
PII_PATTERNS = {
    "SSN": re.compile(r'\b\d{3}-\d{2}-\d{4}\b'),
    "Credit Card (Visa)": re.compile(r'\b4\d{3}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}\b'),
    "Credit Card (Mastercard)": re.compile(r'\b5[1-5]\d{2}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}\b'),
    "Email Address": re.compile(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'),
}

# Files and patterns to exclude from scanning
EXCLUDE_FILES = {"scan_pii.py", ".gitignore"}
SAFE_EMAIL_DOMAINS = {"example.com", "localhost"}


def get_staged_files():
    """Retrieves a list of files currently staged for commit."""
    result = subprocess.run(
        ["git", "diff", "--cached", "--name-only"],
        capture_output=True,
        text=True
    )
    return [f for f in result.stdout.split('\n') if f.strip()]


def is_safe_email(email):
    """Returns True if the email uses a known-safe domain."""
    domain = email.split('@')[-1].lower()
    return domain in SAFE_EMAIL_DOMAINS


def scan_for_pii():
    print("=" * 60)
    print("  PII Security Approval Gate")
    print("=" * 60)

    staged_files = get_staged_files()
    if not staged_files:
        print("\nNo files staged for commit. Passing gate.")
        sys.exit(0)

    total_violations = 0

    for file_path in staged_files:
        if file_path in EXCLUDE_FILES or not os.path.exists(file_path):
            continue

        try:
            with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
                lines = f.readlines()
        except (OSError, UnicodeDecodeError):
            continue

        file_violations = []

        for line_num, line in enumerate(lines, start=1):
            for pii_type, pattern in PII_PATTERNS.items():
                matches = pattern.findall(line)
                for match in matches:
                    # Skip safe email domains
                    if pii_type == "Email Address" and is_safe_email(match):
                        continue
                    file_violations.append((line_num, pii_type, match))

        if file_violations:
            print(f"\n[FAIL] PII DETECTED IN: {file_path}")
            for line_num, pii_type, match in file_violations:
                # Partially redact the match for the report
                redacted = match[:4] + "****" + match[-4:] if len(match) > 8 else "****"
                print(f"   Line {line_num}: {pii_type} - {redacted}")
            total_violations += len(file_violations)

    print("\n" + "=" * 60)

    if total_violations > 0:
        print(f"  BLOCKED: {total_violations} PII violation(s) detected.")
        print("=" * 60)
        print("\nRemediation steps:")
        print("  - Replace real SSNs with fake values (e.g., 000-00-0000)")
        print("  - Replace real credit cards with test numbers")
        print("  - Move sensitive emails to environment variables")
        print("  - Use [REDACTED] placeholders in documentation")
        sys.exit(1)
    else:
        print("  PASSED: No PII detected in staged files.")
        print("=" * 60)
        sys.exit(0)


if __name__ == "__main__":
    scan_for_pii()
```

The script follows the same structure as the secret scanner from the previous walkthrough. It calls `git diff --cached --name-only` to find the files staged for commit, then runs each line through a set of regular expressions. The PII patterns cover Social Security numbers (`\d{3}-\d{2}-\d{4}`), Visa and Mastercard number formats, and email addresses outside a safe list of domains. When it finds a match, it partially redacts the value in the report so the output itself does not leak the full PII.

The `SAFE_EMAIL_DOMAINS` set prevents false positives on addresses like `support@example.com`, which are RFC-reserved and safe to commit. In a production scanner, you would expand this set and add your own internal domains.

#### Step 4 - Wire the gate to the Git lifecycle

To ensure this script runs automatically every time a commit is attempted, we bind it to Git's pre-commit hook:

1. Create a new file in the `.git/hooks` directory named `pre-commit` (no file extension):

```
touch .git/hooks/pre-commit
```

2. Open the `.git/hooks/pre-commit` file in your editor and add the bash command to trigger our Python scanner:

```bash
#!/bin/bash
# Trigger the PII approval gate before allowing the commit
python3 scan_pii.py

# Capture the exit code of the Python script
RESULT=$?

# If the Python script failed (exit code 1), abort the commit
if [ $RESULT -ne 0 ]; then
  echo "Commit aborted due to PII security gate failure."
  exit 1
fi
```

3. Make the hook executable so Git has permission to run it:

```
chmod +x .git/hooks/pre-commit
```

#### Step 5 - The repository rejects the code

Now we test the gate. Stage all the files with PII and attempt a commit:

```
git add config.py seed_data.json README.md
git commit -m "Add initial project files"
```

The moment you hit enter, the pre-commit hook fires. The scanner reads every staged file, finds six PII violations across three files, and blocks the commit:

```
============================================================
  PII Security Approval Gate
============================================================

[FAIL] PII DETECTED IN: config.py
   Line 9: SSN - 123-****6789
   Line 10: Credit Card (Visa) - 4111****1111

[FAIL] PII DETECTED IN: seed_data.json
   Line 5: SSN - 987-****4321
   Line 10: Credit Card (Mastercard) - 5500****0004

[FAIL] PII DETECTED IN: README.md
   Line 3: Email Address - sara****y.com
   Line 4: SSN - 111-****3333

============================================================
  BLOCKED: 6 PII violation(s) detected.
============================================================

Remediation steps:
  - Replace real SSNs with fake values (e.g., 000-00-0000)
  - Replace real credit cards with test numbers
  - Move sensitive emails to environment variables
  - Use [REDACTED] placeholders in documentation
Commit aborted due to PII security gate failure.
```

The Git commit is aborted. The PII remains trapped on the local machine, never reaching the remote repository.

#### Step 6 - Remediate and resubmit

The fix is mechanical. Replace real PII with synthetic test data and move sensitive values to environment variables.

Update `config.py`:

```python
"""Application configuration."""
import os

DATABASE_URL = os.environ.get("DATABASE_URL", "postgresql://localhost:5432/appdb")

SUPPORT_EMAIL = "support@example.com"
SUPPORT_PHONE = "555-0100"

# Test credentials removed. Use environment variables in production.
ADMIN_SSN = os.environ.get("ADMIN_SSN", "000-00-0000")
TEST_CREDIT_CARD = os.environ.get("TEST_CREDIT_CARD", "0000-0000-0000-0000")
```

Update `seed_data.json`:

```json
{
  "test_users": [
    {
      "name": "Jane Doe",
      "email": "jane@example.com",
      "ssn": "000-00-0000",
      "phone": "555-0199"
    },
    {
      "name": "John Smith",
      "email": "john@example.com",
      "credit_card": "0000-0000-0000-0000",
      "cvv": "000"
    }
  ]
}
```

Update `README.md`:

```markdown
# My Application

Contact the team lead at [REDACTED] for access.
The test account uses synthetic data only. See seed_data.json.
```

Stage the cleaned files and commit again:

```
git add config.py seed_data.json README.md
git commit -m "Add initial project files with sanitized test data"
```

The gate passes. The commit proceeds. You did not rely on the developer remembering a policy, and you did not rely on the agent following an instruction. The deterministic rules of the repository blocked the violation before it could leave the machine.

This is the same principle from the secret scanner, applied to a broader and more common class of security violation. In a real project, you would replace this custom script with a production-grade tool like `detect-secrets` from Yelp or `trufflehog` from Truffle Security, both of which support custom regex rules for PII patterns alongside their built-in secret detection.

## REPLACEMENT COPY ENDS HERE

---

## Summary of Changes

### What stays in the manuscript
- Steps 1-3 and Step 5 of the current secret-scanner walkthrough (the scan_secrets.py code, the hook wiring, and the rejection output)
- The full conceptual narrative about Context Boundary Violations, .gitignore limitations, and the Stripe key scenario

### What changes
1. Insert the reproducibility callout before Step 4
2. Add the PII Scanner as "Walkthrough 2" after the secret scanner ends
3. Update the Summary bullet point about the walkthrough to mention both exercises

### Suggested Summary bullet update

Replace:

> Finally, we built a Python walkthrough demonstrating how a local pre-commit hook acts as a rigid approval gate, physically preventing an agent from leaking a Stripe API key into the repository.

With:

> Finally, we built two Python walkthroughs demonstrating structural approval gates. The first showed a pre-commit hook that catches a hardcoded Stripe API key. The second, designed for reliable hands-on reproduction, seeded PII across project files and built a scanner that blocks the commit until every Social Security number, credit card number, and private email address is removed or replaced with synthetic data.

---

## Companion Repo Changes (Already Applied)

The companion repo (`ch08-security-and-safety/README.md`) has been updated with:

1. **Primary Exercise: PII Scanner** - seeds SSNs, credit cards, and email addresses across `config.py`, `seed_data.json`, and `README.md`. Builds `scan_pii.py` with regex patterns, wires to git hooks, demonstrates the gate catching violations, and shows clean remediation.

2. **Secondary Exercise: Secret Scanner** - preserves the manuscript's original walkthrough with the reproducibility note prominently displayed.
