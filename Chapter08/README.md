# Chapter 8 - Security and Safety

## Chapter Summary

This chapter tackles the security threats introduced by autonomous code generators. The central insight is that agentic engineering shatters the traditional "Castle and Moat" security model: the threat is no longer outside the perimeter. It is sitting inside your IDE, operating with read/write access to your repository.

The primary walkthrough builds a **PII Scanner**, a pre-commit hook that detects personally identifiable information (Social Security numbers, credit card numbers, and email addresses) seeded across project files and blocks the commit until the violations are remediated. A secondary walkthrough preserves the book's **Secret Scanner** for Stripe API key leakage.

### Key Concepts

- **The Gullible Insider:** Unlike a human engineer, an agent has no cognitive friction, no intuition for deception, and no fear of consequences. It processes malicious instructions as legitimate next steps.
- **Direct Prompt Injection:** A user intentionally manipulates the agent by embedding override commands in their prompt. Less dangerous than indirect injection because it requires insider intent.
- **Indirect Prompt Injection:** An attacker embeds malicious override commands in external data (GitHub issues, documentation, scraped web pages) that the agent is likely to consume. The innocent developer triggers the attack without realizing it.
- **Dependency Hallucination:** LLMs invent plausible-but-non-existent package names. Attackers register these names on PyPI/npm with malware (e.g., the real-world `huggingface-cli` / JarkaStealer incident).
- **Context Boundary Violations:** When an agent reads a `.env` file, its contents are transmitted to the cloud LLM provider and may be hardcoded into generated files. Two distinct violations: external transmission risk and repository leakage risk.
- **Structural Sandboxing:** Running agents inside network-isolated Docker containers limits the blast radius of a compromised agent.
- **Technical Approval Gates:** Pre-commit hooks and CI checks that block code from being committed unless it passes security scans.

---

## Primary Exercise: PII Scanner

This exercise is fully reproducible. It does not depend on the agent misbehaving. You seed realistic PII across project files, build a scanner, and prove the gate catches violations.

### Step 1: Initialize the Project

```bash
mkdir pii-scanner-demo
cd pii-scanner-demo
git init
```

### Step 2: Seed PII Across Project Files

Create several files with realistic PII embedded in them, simulating what an agent might generate from training data or context leakage.

**`config.py`:**

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

**`seed_data.json`:**

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

**`README.md`:**

```markdown
# My Application

Contact the team lead at sarah.connor@realcompany.com for access.
The test account SSN is 111-22-3333.
```

### Step 3: Build the PII Scanner (`scan_pii.py`)

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


def get_staged_files() -> list[str]:
    """Retrieves a list of files currently staged for commit."""
    result = subprocess.run(
        ["git", "diff", "--cached", "--name-only"],
        capture_output=True,
        text=True
    )
    return [f for f in result.stdout.split('\n') if f.strip()]


def is_safe_email(email: str) -> bool:
    """Returns True if the email uses a known-safe domain."""
    domain = email.split('@')[-1].lower()
    return domain in SAFE_EMAIL_DOMAINS


def scan_for_pii() -> None:
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

### Step 4: Wire the Gate to the Git Lifecycle

```bash
touch .git/hooks/pre-commit
```

Populate `.git/hooks/pre-commit`:

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

Make the hook executable:

```bash
chmod +x .git/hooks/pre-commit
```

### Step 5: The Gate Intercepts the Commit

```bash
git add config.py seed_data.json README.md
git commit -m "Add initial project files"
```

Expected output:

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

### Step 6: Remediate and Resubmit

Fix `config.py` by replacing hardcoded PII with environment variables and fake test data:

```python
"""Application configuration."""
import os

DATABASE_URL = os.environ.get("DATABASE_URL", "postgresql://localhost:5432/appdb")

SUPPORT_EMAIL = "support@example.com"
SUPPORT_PHONE = "555-0100"

# Test credentials removed. Use environment variables.
ADMIN_SSN = os.environ.get("ADMIN_SSN", "000-00-0000")
TEST_CREDIT_CARD = os.environ.get("TEST_CREDIT_CARD", "0000-0000-0000-0000")
```

Fix `seed_data.json` by replacing with synthetic test data:

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

Fix `README.md`:

```markdown
# My Application

Contact the team lead at [REDACTED] for access.
The test account uses synthetic data only. See seed_data.json.
```

Run the commit again. The gate passes.

---

## Secondary Exercise: Secret Scanner (From the Book)

This exercise matches the book's walkthrough. It builds a pre-commit hook that scans for hardcoded Stripe API keys.

> **Note on reproducibility:** Modern agents often refuse to hardcode secrets, warning about the security risk or reaching for `os.environ.get` instead. Treat a refusal as the interesting result. It tells you the model's own guardrail caught this one. It tells you nothing about the *next* prompt, which is why the boundary is built outside the model.

### Step 1: Initialize the Vulnerable Environment

```bash
mkdir security-boundary-demo
cd security-boundary-demo
git init
```

Create `.env` (the sensitive file):

```text
PRODUCTION_STRIPE_KEY=sk_live_REPLACE_WITH_YOUR_TEST_KEY_HERE_1234567890abcdef
```

Create `.gitignore`:

```text
.env
```

### Step 2: The Secret Scanner (`scan_secrets.py`)

```python
import os
import re
import subprocess
import sys


def get_staged_files() -> list[str]:
    """Retrieves a list of files currently staged for commit."""
    result = subprocess.run(
        ["git", "diff", "--cached", "--name-only"],
        capture_output=True,
        text=True
    )
    return [file for file in result.stdout.split('\n') if file.strip()]


def scan_for_secrets() -> None:
    print("Executing Security Approval Gate...")

    staged_files = get_staged_files()
    if not staged_files:
        print("No files staged for commit. Passing gate.")
        sys.exit(0)

    # Regex pattern targeting the standard format of a Stripe Live Secret Key
    stripe_key_pattern = re.compile(r'sk_live_[a-zA-Z0-9]{24,}')

    leak_detected = False

    for file_path in staged_files:
        # Skip checking the scanner itself
        if file_path == "scan_secrets.py" or not os.path.exists(file_path):
            continue

        with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
            content = f.read()
            matches = stripe_key_pattern.findall(content)

            if matches:
                print(f"[FAIL] SECURITY BREACH DETECTED IN: {file_path}")
                print(f"   Found {len(matches)} potential Stripe Secret Key(s).")
                leak_detected = True

    if leak_detected:
        print("\n[FAIL] The agent's code has been rejected by the Project Harness.")
        print("   Remove the hardcoded secrets and use environment variables.")
        sys.exit(1)  # Exit with failure code to block the commit
    else:
        print("[OK] Security Gate Passed: No exposed secrets detected.")
        sys.exit(0)  # Exit with success code to allow the commit


if __name__ == "__main__":
    scan_for_secrets()
```

### Step 3: Wire the Gate to the Git Lifecycle

```bash
touch .git/hooks/pre-commit
```

Populate `.git/hooks/pre-commit`:

```bash
#!/bin/bash
# Trigger the Python approval gate before allowing the commit
python3 scan_secrets.py

# Capture the exit code of the Python script
RESULT=$?

# If the Python script failed (exit code 1), abort the commit
if [ $RESULT -ne 0 ]; then
  echo "Commit aborted due to security gate failure."
  exit 1
fi
```

Make the hook executable:

```bash
chmod +x .git/hooks/pre-commit
```

### Step 4: Simulate the Compromised Agent Output

The agent, having read the `.env` file in its context, generates `payment_gateway.py`:

```python
import stripe

def initialize_payments():
    # Agent hardcodes the key it found in the .env file
    stripe.api_key = "sk_live_REPLACE_WITH_YOUR_TEST_KEY_HERE_1234567890abcdef"

    print(f"Initialized Stripe client with key: {stripe.api_key}")
    return stripe
```

### Step 5: The Gate Intercepts the Commit

```bash
git add payment_gateway.py
git commit -m "Initialize Stripe payment gateway"
```

Expected output:

```
Executing Security Approval Gate...
[FAIL] SECURITY BREACH DETECTED IN: payment_gateway.py
   Found 1 potential Stripe Secret Key(s).

[FAIL] The agent's code has been rejected by the Project Harness.
   Remove the hardcoded secrets and use environment variables.
Commit aborted due to security gate failure.
```

### The Correct Agent Output (Using Environment Variables)

```python
import os
import stripe

def initialize_payments() -> stripe.Stripe:
    api_key = os.environ.get("PRODUCTION_STRIPE_KEY")
    if not api_key:
        raise EnvironmentError("PRODUCTION_STRIPE_KEY environment variable is not set.")
    stripe.api_key = api_key
    return stripe
```

### Docker Sandbox Configuration (Structural Sandboxing)

```bash
# Run agent execution in a network-isolated container
docker run --rm \
  --network none \
  --memory 512m \
  --cpus 1 \
  -v "$(pwd)":/workspace \
  python:3.11-slim \
  bash -c "cd /workspace && python your_agent_script.py"
```

---

## Supplemental Resources

### Prompt Injection
- **[OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)** - The definitive reference for LLM-specific vulnerabilities. Prompt injection is LLM01.
- **[Indirect Prompt Injection Attacks - Greshake et al. (Paper)](https://arxiv.org/abs/2302.12173)** - The original academic paper formally defining indirect prompt injection and its threat model.
- **[Prompt Injection Primer - Simon Willison](https://simonwillison.net/2022/Sep/12/prompt-injection/)** - Simon Willison's accessible explanation of the prompt injection problem.

### Dependency Security
- **[JarkaStealer / huggingface-cli Incident Analysis - Lasso Security](https://www.lasso.security/)** - Research documenting the real-world dependency hallucination attack referenced in the chapter.
- **[pip-audit](https://github.com/pypa/pip-audit)** - Audits Python dependencies for known vulnerabilities.
- **[Socket.dev](https://socket.dev/)** - Real-time supply chain security for npm packages. Detects malicious packages before they are installed.

### Secret & PII Scanning Tools (Production-Grade Alternatives)
- **[TruffleHog](https://github.com/trufflesecurity/trufflehog)** - Searches Git history and staged files for hundreds of secret types using entropy scanning. Use this in production instead of the custom scripts.
- **[Gitleaks](https://github.com/gitleaks/gitleaks)** - A SAST tool for detecting hardcoded secrets. Configurable, and can be run as a pre-commit hook or CI step.
- **[GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)** - Native GitHub feature that automatically scans pushed commits for known secret patterns.
- **[detect-secrets (Yelp)](https://github.com/Yelp/detect-secrets)** - A tool for preventing secrets from being committed. Supports PII patterns and custom regex rules.

### Sandboxing and Container Security
- **[Docker Security Best Practices](https://docs.docker.com/engine/security/)** - Official Docker documentation on security hardening, including network isolation and capability restrictions.
- **[gVisor](https://gvisor.dev/)** - Google's container sandbox that provides a kernel-level boundary for untrusted workloads.
