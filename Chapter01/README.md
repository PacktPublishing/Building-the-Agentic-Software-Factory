# Chapter 1 - Beyond Vibe Coding

## Chapter Summary

This chapter takes apart the myth of "vibe coding" - the practice of typing vague natural language prompts into a chat window and hoping an AI produces production-grade software. Through the lens of Dave Farley's engineering philosophy, the chapter explains why this approach breaks down at enterprise scale and introduces the foundational concept of **Agentic Engineering**: a rigorous, discipline-driven methodology for deploying autonomous AI agents in professional software development.

### Key Concepts

- **Vibe Coding vs. Agentic Engineering:** Vibe coding generates the first 90% of a feature quickly but fails at the final 10% required for production reliability. Agentic Engineering treats the agent as a tool, not a magic oracle.
- **Farley's Two Pillars:** Dave Farley's software engineering philosophy centers on two ideas - *optimize for learning* (working software surfaces truth faster than documentation) and *manage complexity* (keep the cognitive surface area of the system navigable by humans).
- **The Shift in Developer Role:** The developer's job moves from *syntax author* to *systems architect*. You are no longer the one writing the code; you are the one designing the environment that forces the agent to write the right code.
- **The 70% Problem:** Autonomous agents are statistically good at the "happy path" (70-90% of common patterns), but they need rigorous human-designed constraints to handle the remaining edge cases that production demands.
- **Why Discipline Matters More, Not Less:** As agent capability increases, the blast radius of a poorly structured prompt also increases. Engineering discipline is not the enemy of AI productivity - it is its prerequisite.

---

## Code Snippets

> Chapter 1 is primarily conceptual and does not contain hands-on code walkthroughs. The practical exercises begin in Chapter 3 (Context Engineering).

---

## Supplemental Resources

### Foundational Engineering Philosophy
- **[Continuous Delivery](https://continuousdelivery.com/)** by Dave Farley - The source of the two engineering pillars referenced throughout the book. Good reading for understanding why engineering rigor compounds over time.
- **[Modern Software Engineering](https://www.amazon.com/Modern-Software-Engineering-Discipline-Development/dp/0137314914)** by David Farley - Farley's text on applying scientific thinking to software. Chapter 1 of the book builds directly on this framework.

### Understanding AI Limitations in Production
- **[Andrej Karpathy on LLM Limitations](https://karpathy.ai/)** - Karpathy's essays and talks on the probabilistic nature of language models and the gap between demo performance and production reliability.
- **[The 70% Problem - Swyx](https://www.latent.space/p/70-problem)** - An analysis of why AI coding tools consistently excel at the first 70% of any task but struggle to close the remaining gap without rigorous human guidance.

### Agentic Engineering Introductions
- **[LLM-powered Autonomous Agents - Lilian Weng (OpenAI)](https://lilianweng.github.io/posts/2023-06-23-agent/)** - A rigorous technical breakdown of agent architecture, planning, memory, and tool use. The best single reference for understanding what makes an agent an agent.
- **[Building Effective Agents - Anthropic](https://www.anthropic.com/research/building-effective-agents)** - Anthropic's practical guidance on when to use agents, when not to, and how to design workflows that are both autonomous and safe.
