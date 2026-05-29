# Chapter 9 - The Team Operating Model

## Chapter Summary

Adopting AI as an individual is a technical exercise. Adopting AI as an entire engineering team is an organizational change. This chapter maps the adjustments required to run agentic engineering at scale - addressing the Code Review Crisis, replacing vanity metrics with DORA metrics, establishing team agreements, and designing a phased rollout.

The walkthrough builds an AI-Aware Pull Request Template that solves the Code Review Crisis by shifting the burden of proof from reviewer to author.

### Key Concepts

- **The Code Review Crisis:** When one developer generates 2,000 lines in 45 minutes but the reviewer still needs three days to read it, the PR backlog becomes the bottleneck. Velocity moves from the "writing" phase to the "reviewing" phase.
- **Shifting the Cognitive Load:** The solution is not to review faster; it is to change *what* reviewers look at. Reviewers stop checking syntax (the harness does that) and focus exclusively on architectural alignment and business logic.
- **DORA Metrics:** The four outcomes-based metrics that hold up under AI adoption:
  1. **Deployment Frequency:** How often code reaches production.
  2. **Lead Time for Changes:** How long from commit to production.
  3. **Change Failure Rate:** What percentage of deployments cause outages.
  4. **Mean Time to Recovery (MTTR):** How fast the team restores service after a failure.
- **Team Agreements:** Social contracts that establish shared expectations for AI use - accountability (humans are liable for agent output), prompt transparency (document the prompts used), context boundaries (define off-limits data), and continuous education.
- **The Phased Rollout:** Four phases: (1) Tiger Team Experimentation, (2) Standardization, (3) Harnessing, (4) Team-wide rollout. Never skip Phase 3 (Harnessing) to rush to Phase 4.
- **The Broken Window Theory of AI:** Agents mirror the code quality of their context. A messy codebase produces messy agent output. A clean codebase acts as a force multiplier.

---

## Code Snippets

### Step 1: Initialize the Repository Structure

```bash
mkdir team-operating-model-demo
cd team-operating-model-demo
git init
```

```bash
mkdir .github
```

```bash
touch .github/PULL_REQUEST_TEMPLATE.md
```

### Step 2: The AI-Aware Pull Request Template

Populate `.github/PULL_REQUEST_TEMPLATE.md`:

```markdown
## Description
Provide a clear, human-written summary of the business logic introduced in this Pull Request. What specific problem does this solve for the end user, and how does it fit into the broader epic?

## Agentic Generation Context
*To assist the human reviewer in understanding the architectural approach, please provide the details of your agentic workflow. This is mandatory for any PR containing more than 20% AI-generated code.*

- **Primary Agent Used:** (e.g., Cursor, Claude 3.5 Sonnet, Copilot)
- **High-Level Prompt / Algorithmic Specification:**
  *(Paste the exact core architectural prompt or the algorithmic specification logic you provided to the agent to generate this feature).*
  > [Insert Prompt Here]

## Technical Verification and Accountability
*Syntax, code formatting, and basic linting have been automatically handled by the Project Harness CI pipeline. As the human author, please confirm the following structural and safety boundaries before requesting a review:*

- [ ] **Human Accountability:** I have read, reviewed, and fully understand the generated code. I take full professional accountability for its execution, performance, and stability in the production environment.
- [ ] **Architectural Alignment:** This code aligns with the design patterns and architectural pillars defined in our `architecture.md`.
- [ ] **Data Privacy and Context Boundaries:** I have verified that no sensitive data (PII, production database dumps, `.env` secrets) was accidentally ingested into the agent's context window during the generation phase.
- [ ] **Dependency Verification:** If this PR introduces new external dependencies or packages, I have manually verified that they exist on the official registry (e.g., PyPI, npm) and are not the result of an AI hallucination.
- [ ] **Test Coverage:** The automated test suite has been updated to cover the new logic, and total repository coverage remains above our required 85% threshold.

## Reviewer Focus
*Reviewers: Please do not spend your time checking for missing semicolons, incorrect variable casing, or formatting issues. The CI pipeline has already verified syntax. Please focus your attention on:*
1. Does the implementation cleanly align with the author's stated High-Level Prompt?
2. Are the new API boundaries cleanly separated, or does this introduce tight coupling?
3. Are there any edge cases, race conditions, or unhandled exceptions that the agent's probabilistic logic might have missed?
```

### Step 3: Commit the Template

```bash
git add .github/PULL_REQUEST_TEMPLATE.md
git commit -m "chore: add AI-aware pull request template"
```

### DORA Metrics Quick Reference

| Metric | What It Measures | AI Era Insight |
|---|---|---|
| Deployment Frequency | How often code reaches production | Should increase with agentic adoption; flat means PR bottleneck |
| Lead Time for Changes | Commit to production time | Reveals process bureaucracy the agent cannot overcome |
| Change Failure Rate | % of deployments causing failures | The most critical metric; rising = too much autonomy, too little harness |
| MTTR | Time to restore service after failure | Agents can dramatically improve this via rapid root-cause analysis |

### Team Agreement Template

```markdown
# AI Engineering Team Agreement - [Team Name]

## Version: 1.0 | Effective: [Date]

---

## Pillar 1: Accountability
The human developer whose name is on the git commit is 100% accountable for the
behavior of the code in production. You cannot blame the agent. The agent is a tool.

## Pillar 2: Prompt Transparency
For any PR containing more than 20% AI-generated code, the author MUST document
the core prompt(s) used in the PR description using the team's PR template.

## Pillar 3: Context Boundaries
The following directories and file types are STRICTLY OFF-LIMITS for agent context:
- [ ] `customer_migrations/` - Contains raw PII schema examples
- [ ] `.env` files - All environments
- [ ] `internal-contracts/` - Proprietary business logic

## Pillar 4: Continuous Education
- **Bi-weekly Prompt Engineering Sync:** Every other [day], 30 minutes
- **Shared Resource:** [Link to team's internal prompt library]
- **Hallucination Log:** Document significant hallucinations in [shared doc] so the
  whole team learns from each failure.

## Approved Tooling (Standardized in Phase 2)
- **IDE Client:** [Cursor / Windsurf / Claude Code]
- **Foundational Model:** [Claude 3.5 Sonnet / GPT-4o]
- **Data Privacy Agreement:** [Link to signed enterprise agreement]
```

### Phased Rollout Summary

| Phase | Duration | Goal | Key Output |
|---|---|---|---|
| 1 - Experimentation | 4-6 weeks | Risk discovery, tool evaluation | Empirical findings report |
| 2 - Standardization | 2-4 weeks | Single toolset, legal agreements | Signed enterprise agreements |
| 3 - Harnessing | 4-8 weeks | Build Project Harness, tighten CI | Harnessed repositories |
| 4 - Team-wide Rollout | Ongoing | Deploy to all engineers | Operational AI organization |

---

## Supplemental Resources

### DORA Metrics
- **[Accelerate - Nicole Forsgren, Jez Humble, Gene Kim](https://itrevolution.com/product/accelerate/)** - The foundational text establishing DORA metrics and the science behind measuring engineering productivity.
- **[DORA State of DevOps Report](https://dora.dev/research/)** - Annual industry benchmark report. Use it to understand where your team sits relative to industry peers.
- **[Google Cloud DORA Metrics Dashboard](https://cloud.google.com/blog/products/devops-sre/using-the-four-keys-to-measure-your-devops-performance)** - How to instrument your pipeline to track the four keys.

### Team Topologies and Organizational Design
- **[Team Topologies - Matthew Skelton & Manuel Pais](https://teamtopologies.com/)** - Useful reading for structuring engineering pods and interaction models in an agentic org. Directly relevant to the "Tiger Team" concept.
- **[The Phoenix Project - Gene Kim, Kevin Behr, George Spafford](https://itrevolution.com/product/the-phoenix-project/)** - A novel that makes the Theory of Constraints tangible. The bottleneck management insights apply directly to the Code Review Crisis.

### Pull Request Templates
- **[GitHub PR Template Documentation](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository)** - Official GitHub guide for creating and configuring PR templates, including multi-template setups.
- **[GitLab Merge Request Templates](https://docs.gitlab.com/ee/user/project/description_templates.html)** - The GitLab equivalent.

### Code Review Best Practices
- **[Google Engineering Practices - Code Review](https://google.github.io/eng-practices/review/)** - Google's internal code review standards, made public. Directly applicable to the "reviewer focus" section of the PR template.
- **[Conventional Commits](https://www.conventionalcommits.org/)** - A specification for commit messages that makes automated changelog generation and semantic versioning possible.
