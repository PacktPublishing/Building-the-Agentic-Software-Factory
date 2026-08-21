# Chapter 3 - Context Engineering

## Chapter Summary

Context Engineering is the discipline of deliberately designing the information architecture that an agent operates within. This chapter proves - through a hands-on walkthrough - that the agent's output quality is not primarily determined by the underlying language model. It is determined by the quality, precision, and relevance of the context you provide.

The walkthrough shows the difference between an open-loop hallucination (no context constraints) and a closed-loop execution (context bound by an Architectural Decision Record), using an Express.js server as the test subject.

### Key Concepts

- **Information Architecture:** Treating your repository's documentation, ADRs, and configuration files as a first-class engineering artifact that the agent consumes, not just human readers.
- **Architectural Decision Records (ADRs):** Formal markdown documents that record the *why* behind technical decisions. When loaded into the agent's context window, ADRs act as rigid constraints that override the model's statistical baseline.
- **The `.cursorrules` / `AGENTS.md` Pattern:** Agent-specific configuration files placed at the repository root that are automatically loaded into every conversation, establishing permanent project-wide constraints.
- **Open-Loop vs. Closed-Loop Execution:** Open loop means you prompt the agent with no context and it generates the statistically most probable output. Closed loop means you inject specific context and the agent is constrained to comply with your organization's standards.
- **RAG (Retrieval-Augmented Generation):** The architectural pattern where the client harness selectively retrieves relevant files and injects them into the context window at prompt time, rather than loading the entire codebase at once.

---

## Code Snippets

### Step 1: Initialize the Project

```bash
mkdir context-exercise && cd context-exercise
```

```bash
npm init -y
```

```bash
touch server.js
```

### Step 2: The Open-Loop Hallucination

Issue this prompt to your agent with no context provided:

```
Build a basic Express server in server.js that listens on port 3000 and has a single /health route. Make sure to log every request that comes in.
```

The agent will likely produce `console.log()` - its statistical baseline:

```javascript
app.use((req, res, next) => {
    console.log(`Received request: ${req.method} ${req.url}`);
    next();
});
```

### Step 3: Create the ADR Context Boundary

```bash
mkdir docs
touch docs/ADR-001-Logging.md
```

Populate `docs/ADR-001-Logging.md`:

```markdown
# ADR-001: Logging Standards - APPROVED

## Context
All enterprise applications must output logs in a structured JSON format to be accurately ingested by our Datadog observability pipeline.

## The Absolute Constraint
You MUST NOT use standard `console.log`, `console.info`, or `console.error` anywhere in the application.
You MUST use the Winston logging library.
All logs MUST be formatted explicitly as JSON strings.
```

### Step 4: The Closed-Loop Execution

Start a new agent session and issue this prompt with the ADR referenced:

```
Read the rules in docs/ADR-001-Logging.md. Then, build a basic Express server in server.js that listens on port 3000 and has a single /health route. Make sure to log every request that comes in.
```

The agent now produces the constrained, ADR-compliant output:

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  format: winston.format.json(),
  transports: [
    new winston.transports.Console()
  ]
});

app.use((req, res, next) => {
    logger.info({
        message: 'Incoming request',
        method: req.method,
        url: req.url
    });
    next();
});
```

### Reference: ADR Template

Use this template for all ADRs in your own projects:

```markdown
# ADR-XXX: [Decision Title] - [PROPOSED | APPROVED | DEPRECATED]

## Context
[Why does this decision need to be made? What is the problem being solved?]

## Decision
[What was decided?]

## The Absolute Constraint
[Machine-readable rules the agent MUST follow. Use MUST/MUST NOT language.]

## Consequences
[What are the trade-offs?]
```

---

## Supplemental Resources

### Architectural Decision Records
- **[ADR GitHub - Michael Nygard](https://github.com/joelparkerhenderson/architecture-decision-record)** - A large collection of ADR templates and examples. The source of the ADR pattern referenced in the chapter.
- **[adr-tools](https://github.com/npryce/adr-tools)** - CLI tooling to create and manage ADRs directly from the terminal.
- **[Documenting Architecture Decisions - Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)** - The original blog post that popularized ADRs in the software industry.

### Context Window Management
- **[Anthropic's Context Window Documentation](https://docs.anthropic.com/en/docs/about-claude/models)** - Current context window sizes and best practices for the Claude model family.
- **[Cursor Rules Documentation](https://docs.cursor.com/context/rules)** - Official documentation for `.cursorrules` files, the primary mechanism for persistent project-level context in Cursor.

### RAG Architecture
- **[Retrieval-Augmented Generation (RAG) - IBM](https://research.ibm.com/blog/retrieval-augmented-generation-RAG)** - A clear, non-academic explanation of how RAG works and why it is the dominant architecture for grounding LLMs in organizational knowledge.
- **[LlamaIndex Documentation](https://docs.llamaindex.ai/)** - The leading Python library for building RAG pipelines. Useful for teams building custom context injection systems.

### Winston (Logging Library)
- **[Winston GitHub Repository](https://github.com/winstonjs/winston)** - Official documentation and examples for the Winston logging library used in the walkthrough.
