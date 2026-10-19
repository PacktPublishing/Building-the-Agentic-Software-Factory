<h1 align="center">
Building the Agentic Software Factory, First Edition</h1>
<p align="center">This is the code repository for <a href ="https://www.packtpub.com/en-us/product/building-the-agentic-software-factory-first-edition/9781807782771"> Building the Agentic Software Factory, First Edition</a>, published by Packt.
</p>

<h2 align="center">
Scaling reliable AI coding practices from individual developers to teams
</h2>
<p align="center">
Alex Merced, Benedikt Stemmildt</p>

<p align="center">
  <a href="https://packt.link/free-ebook/9781807782771"><img width="32px" alt="Free PDF" title="Free PDF" src="https://cdn-icons-png.flaticon.com/512/4726/4726010.png"/></a>
 &#8287;&#8287;&#8287;&#8287;&#8287;
  <a href="https://packt.link/gbp/9781807782771"><img width="32px" alt="Graphic Bundle" title="Graphic Bundle" src="https://cdn-icons-png.flaticon.com/512/2659/2659360.png"/></a>
  &#8287;&#8287;&#8287;&#8287;&#8287;
   <a href="https://www.amazon.com/Building-Software-Factory-Individual-developers/dp/1807782778/"><img width="32px" alt="Amazon" title="Get your copy" src="https://cdn-icons-png.flaticon.com/512/15466/15466027.png"/></a>
  &#8287;&#8287;&#8287;&#8287;&#8287;
</p>

<details open> 
  <summary><h2>About the book</summary>
<a href="https://www.packtpub.com/en-us/product/building-the-agentic-software-factory-first-edition/9781807782771">
<img src="https://content.packt.com/B38794/cover_image_small.jpg" alt="Building the Agentic Software Factory, First Edition" height="256px" align="right">
</a>

AI coding agents can generate code in seconds, but without the right engineering practices, they can also introduce bugs, technical debt, and inconsistent code. This book teaches you how to build reliable AI-assisted development workflows that scale from individual projects to entire teams, regardless of which coding assistant or model you use.</br>
Rather than treating AI as a black box, you'll learn how AI coding agents work, why they succeed or fail, and how to build systems that consistently produce better results. You'll explore practical methods such as context and harness engineering, along with reusable frameworks such as AEMM, Elastic Loop, and Context Diseases, which you can apply to any development workflow.</br>
You'll also learn how to write better specifications, use AI-TDD to improve code quality, build development harnesses that guide AI agents, and choose the right level of autonomy for different tasks. The book also covers AI security, team adoption, and how to scale from individual AI-assisted development to a software factory where multiple AI workflows work together.</br>
By the end of this book, you'll understand how to move beyond vibe coding and adopt a disciplined, practical approach to AI-driven software development.</details>
<details open> 
  <summary><h2>Key learnings</summary>
<ul>
    <li>Distinguish between vibe coding and agentic engineering for reliable AI development</li>
    <li>Apply context engineering to improve AI output and reduce common failures</li>
    <li>Write clear specifications to guide AI agents and reduce hallucinations</li>
    <li>Implement AI-TDD workflows to improve code quality and reliability</li>
    <li>Build development harnesses with context, quality checks, and workflow guidance</li>
    <li>Calibrate AI autonomy using the AEMM maturity model and Elastic Loop framework</li>
    <li>Secure AI development workflows against prompt injection and context leakage</li>
</ul>

  </details>

<details open> 
  <summary><h2>Chapters</summary>
     <img src="https://cliply.co/wp-content/uploads/2020/02/372002150_DOCUMENTS_400px.gif" alt="Unity Cookbook, Fifth Edition" height="556px" align="right">
<ol>
  <li>Beyond Vibe Coding</li>
  <li>The Anatomy of an Agent</li>
  <li>Curating Context</li>
  <li>Shaping Intent</li>
  <li>Closing Loops</li>
  <li>The Complete Harness</li>
  <li>The Autonomy Spectrum</li>
  <li>Security and Safety</li>
  <li>The Team Operating Model</li>
  <li>The Developer's New Identity</li>
  <li>The Software Factory</li>
  <li>Appendix</li>
</ol>
</details>

<details open> 
  <summary><h2>Requirements for this book</h2></summary>
This book takes a concept-first, tool-second approach. Each chapter teaches a principle andthen grounds it with a practical walkthrough. You do not need a specific agent or IDE to followalong; any autonomous coding client (Cursor, Windsurf, Claude Code, OpenCode, orequivalent) will work.</br>
For the hands-on walkthroughs, you will need:</br>
<ul>
    <li><b>Node.js</b> (version 18 or higher) and <b>npm</b>, for the walkthroughs in <i>Chapters 3 and 4</i></li>
    <li><b>Python</b> (version 3.10 or higher), for the walkthroughs in <i>Chapters 5, 6, 7, and 8</i></li>
    <li><b>Git</b>, for repository initialization and hook configuration</li>
    <li>A terminal application (Terminal on macOS/Linux, or PowerShell/Windows Terminalon Windows)</li>
    <li>An autonomous coding agent of your choice</li>
 </ul>
Prior knowledge of machine learning or AI internals is not required. Solid softwaredevelopment fundamentals (version control, testing, CI/CD, and proficiency in at least onemodern language) are the only prerequisites.</br>
The book is designed to be read in order. Each chapter builds on the concepts and vocabularyintroduced in the chapters before it, and the walkthroughs accumulate skills progressively. Ifyou already work with an agent daily, you can start at <i>Chapter 6</i>, <i>The Complete Harness</i>, and refer back to earlier chapters as needed.
</details>
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

<details> 
  <summary><h2>Get to know the authors</h2></summary>

_Alex Merced_ is a passionate technologist, educator, and Head of Developer Relations at Dremio. With a proven track record of demystifying complex technical ecosystems, Alex is the author of several definitive guides on data engineering, including books on Apache Iceberg and Apache Polaris. Before moving into full-stack development and teaching at General Assembly, he spent over a decade as a financial trainer in New York City. Today, as an early adopter of autonomous coding agents and prompt-driven development, he speaks at conferences around the world, including Data Council, and hosts popular tech podcasts. Alex is dedicated to exploring and documenting how AI is reshaping software architecture and team velocity.</br>

_Benedikt Stemmildt_ is Co-Founder and Co-CEO of hackers&wizards, a Hamburg-based consultancy that helps engineering teams adopt agentic software engineering practices. With over 20 years of enterprise architecture experience across major German retail and ecommerce enterprises, he brings a strong foundation in engineering to the AI transition. As Program Chair for code.talks (3,000+ attendees) and a speaker at more than 40 conferences, Benedikt has trained and mentored hundreds of engineers on context engineering, AI-TDD, and team-level AI adoption. His clients include Fortune 500 enterprises across retail, manufacturing, and media.
</details>

<details> 
  <summary><h2>Other related books</h2></summary>
<ul>
  <li><a href="https://www.packtpub.com/en-us/product/claude-visual-bible-first-edition/9781808658396">Claude Visual Bible, First Edition</a></li>
  <li><a href="https://www.packtpub.com/en-us/product/agentic-coding-with-openai-codex-cli-first-edition/9781808348891">Agentic Coding with OpenAI Codex CLI, First Edition</a></li>
</ul>
</details>
