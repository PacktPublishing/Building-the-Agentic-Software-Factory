# Chapter 8 - Security and Safety

## Chapter Summary

This chapter tackles the security threats introduced by autonomous code generators. The central insight is that agentic engineering shatters the traditional "Castle and Moat" security model: the threat is no longer outside the perimeter. It is sitting inside your IDE, operating with read/write access to your repository.

The walkthrough builds a Python-based pre-commit hook (`scan_secrets.py`) that acts as a physical approval gate, preventing agent-generated code from leaking secrets into the repository.

### Key Concepts

- **The Gullible Insider:** Unlike a human engineer, an agent has no cognitive friction, no intuition for deception, and no fear of consequences. It processes malicious instructions as legitimate next steps.
- **Direct Prompt Injection:** A user intentionally manipulates the agent by embedding override commands in their prompt. Less dangerous than indirect injection because it requires insider intent.
- **Indirect Prompt Injection:** An attacker embeds malicious override commands in external data (GitHub issues, documentation, scraped web pages) that the agent is likely to consume. The innocent developer triggers the attack without realizing it.
- **Dependency Hallucination:** LLMs invent plausible-but-non-existent package names. Attackers register these names on PyPI/npm with malware (e.g., the real-world `huggingface-cli` / JarkaStealer incident).
- **Context Boundary Violations:** When an agent reads a `.env` file, its contents are transmitted to the cloud LLM provider and may be hardcoded into generated files. Two distinct violations: external transmission risk and repository leakage risk.
- **Structural Sandboxing:** Running agents inside network-isolated Docker containers limits the blast radius of a compromised agent.
- **Technical Approval Gates:** Pre-commit hooks and CI checks that block code from being committed unless it passes security scans.

---

## Code Snippets

### Step 1: Initialize the Vulnerable Environment

```bash
mkdir security-boundary-demo
cd security-boundary-demo
git init
```

Create `/.env` (the sensitive file):

```text
PRODUCTION_STRIPE_KEY=sk_live_<YOUR_STRIPE_KEY_HERE>
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
                print(f"SECURITY BREACH DETECTED IN: {file_path}")
                print(f"   Found {len(matches)} potential Stripe Secret Key(s).")
                leak_detected = True

    if leak_detected:
        print("\nThe agent's code has been rejected by the Project Harness.")
        print("   Remove the hardcoded secrets and use environment variables.")
        sys.exit(1)  # Exit with failure code to block the commit
    else:
        print("Security Gate Passed: No exposed secrets detected.")
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
    stripe.api_key = "sk_live_<YOUR_STRIPE_KEY_HERE>"

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
SECURITY BREACH DETECTED IN: payment_gateway.py
   Found 1 potential Stripe Secret Key(s).

The agent's code has been rejected by the Project Harness.
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
- **[OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)** - The definitive reference for LLM-specific vulnerabilities. Prompt injection is LLM01. Required reading for any team deploying agents.
- **[Indirect Prompt Injection Attacks - Greshake et al. (Paper)](https://arxiv.org/abs/2302.12173)** - The original academic paper formally defining indirect prompt injection and its threat model.
- **[Prompt Injection Primer - Simon Willison](https://simonwillison.net/2022/Sep/12/prompt-injection/)** - Simon Willison's accessible explanation of the prompt injection problem, with examples.

### Dependency Security
- **[JarkaStealer / huggingface-cli Incident Analysis - Lasso Security](https://www.lasso.security/)** - Research documenting the real-world dependency hallucination attack referenced in the chapter.
- **[pip-audit](https://github.com/pypa/pip-audit)** - Audits Python dependencies for known vulnerabilities. Use in your CI pipeline alongside the secret scanner.
- **[Socket.dev](https://socket.dev/)** - Real-time supply chain security for npm packages. Detects malicious packages before they are installed.

### Secret Scanning Tools (Production-Grade Alternatives)
- **[TruffleHog](https://github.com/trufflesecurity/trufflehog)** - Searches Git history and staged files for hundreds of secret types using entropy scanning. Use this in production instead of the custom script.
- **[Gitleaks](https://github.com/gitleaks/gitleaks)** - A SAST tool for detecting hardcoded secrets. Configurable, and can be run as a pre-commit hook or CI step.
- **[GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)** - Native GitHub feature that automatically scans pushed commits for known secret patterns.

### Sandboxing and Container Security
- **[Docker Security Best Practices](https://docs.docker.com/engine/security/)** - Official Docker documentation on security hardening, including network isolation and capability restrictions.
- **[gVisor](https://gvisor.dev/)** - Google's container sandbox that provides a kernel-level boundary for untrusted workloads. Stronger isolation than standard Docker.
