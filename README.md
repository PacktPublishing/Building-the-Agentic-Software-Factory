# Agentic Software Engineering - Companion Repository

Welcome to the official companion repository for the book **"Agentic Software Engineering: A Practical Guide to AI-Driven Development"** published by Packt Publishing.

> **Authors:** Alex Merced & Benedikt Stemmildt

---

## What Is This Repository?

This repository is a hands-on companion to the book. For every chapter you will find:

- **Code snippets:** the exact implementations from the chapter walkthroughs, ready to copy and run.
- **Supplemental links:** curated external resources that deepen the concepts covered in the chapter.
- **Reference material:** key schema examples, configuration templates, and specification artifacts referenced in the text.

While the book explains *why* and *how*, this repository gives you the *what*: the working code so you can follow along, experiment, and extend the ideas into your own projects.

---

## Book Overview

The book is organized into three parts:

### Part 1 - The Paradigm Shift
Chapters 1 and 2 establish the conceptual foundation. You will learn why traditional "vibe coding" breaks down at enterprise scale and understand how autonomous agents actually work at a mechanical level (the ReAct loop, tool use, and the anatomy of an agent session).

### Part 2 - The Harness (Core Methodology)
Chapters 3 through 7 introduce the three engineering disciplines that form the backbone of a professional agentic workflow, then wire them together and calibrate the result:

| Chapter | Discipline |
|---|---|
| Ch. 3 - Curating Context | Structuring your project's information architecture so the agent always has the right knowledge |
| Ch. 4 - Shaping Intent | Replacing ambiguous English prompts with precise specifications using the Plan-Build-Verify cycle |
| Ch. 5 - Closing Loops | Using AI-TDD and layered verification loops to prove that agent-generated code is correct |
| Ch. 6 - The Complete Harness | Wiring all three disciplines into a self-reinforcing Project Harness |
| Ch. 7 - The Autonomy Spectrum | Calibrating how much freedom you grant an agent based on risk and context quality |

### Part 3 - The Transformation
Chapters 8 through 11 address the organizational and strategic implications:

| Chapter | Topic |
|---|---|
| Ch. 8 - Security & Safety | Prompt injection, dependency hallucination, and structural defenses |
| Ch. 9 - The Team Operating Model | Solving the Code Review Crisis and restructuring your team for agentic velocity |
| Ch. 10 - The Developer's New Identity | The shift from hand work to head work |
| Ch. 11 - The Software Factory | Designing always-on, self-sustaining Orchestrator-Worker-Validator systems |

---

## Repository Structure

```
Building-the-Software-Factory/
├── README.md                              ← You are here
│
├── Chapter01/
│   └── README.md
├── Chapter02/
│   └── README.md
├── Chapter03/
│   └── README.md
├── Chapter04/
│   └── README.md
├── Chapter05/
│   └── README.md
├── Chapter06/
│   └── README.md
├── Chapter07/
│   └── README.md
├── Chapter08/
│   └── README.md
├── Chapter09/
│   └── README.md
├── Chapter10/
│   └── README.md
└── Chapter11/
    └── README.md
```

---

## Prerequisites

Different chapters have different technical requirements. Here are the recurring prerequisites across the walkthroughs:

| Tool | Minimum Version | Used In |
|---|---|---|
| Node.js + npm | v18+ | Ch. 3, 4 |
| Python | v3.10+ | Ch. 5, 6, 7, 8, 9 |
| Git | Any recent version | Ch. 8, 9 |
| An agentic coding client | - | Ch. 3-7 |

> **Recommended agentic clients:** [Cursor](https://cursor.sh), [Windsurf](https://codeium.com/windsurf), or [Claude Code](https://www.anthropic.com/claude-code).

---

## How to Use This Repository

Each chapter directory is self-contained. Navigate into the chapter folder that matches the chapter you are reading and follow the `README.md` inside.

The most effective way to learn is to **type the code yourself** rather than copy-paste. Use the snippets here as a reference and verification point, not a shortcut.

---

## Stay Connected

- **Alex Merced:** [alexmercedcoder.dev](https://alexmercedcoder.dev)
- Follow updates and new patterns from the authors on social media and their respective platforms.

---

*"Build the harness. Calibrate the autonomy. Design the factory. That is the work."*