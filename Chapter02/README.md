# Chapter 2 - The Anatomy of an Agent

## Chapter Summary

This chapter opens the hood on autonomous coding agents and explains how they actually function. Understanding the internal mechanics is not just academic - it is the prerequisite for being able to steer an agent effectively. You cannot correct what you cannot diagnose, and you cannot diagnose what you do not understand.

### Key Concepts

- **The ReAct Loop:** The foundational engine of any autonomous agent. ReAct stands for **Reason - Act - Observe**. The agent reasons about its current goal, selects and invokes a tool (an action), observes the result of that action, then reasons again, looping until the goal is satisfied or a boundary is hit.
- **The Elastic Loop:** An extension of the ReAct loop specifically for agentic coding. The loop expands to fill the available context window and contracts when the agent self-corrects based on verification feedback (compiler errors, failing tests, linter output).
- **Tool Use:** Agents do not "think" in isolation. They invoke discrete tools: file read/write, terminal execution, web search, code search. Each tool call is an action in the ReAct loop.
- **The Context Window as Working Memory:** Everything the agent knows at a given moment lives in its context window. The context window is finite, expensive, and degrades in quality when overloaded (the "Lost in the Middle" problem).
- **The Four Context Diseases:** Four common failure modes that corrupt the agent's working memory:
  1. **Context Overload:** Injecting more information than the model can process effectively.
  2. **Context Poisoning:** Providing misleading or contradictory information that causes hallucinations.
  3. **Context Distraction:** Loading irrelevant context that causes the agent to solve the wrong problem.
  4. **Context Confusion:** Providing ambiguous context the model interprets unpredictably.
- **Stateless vs. Stateful Agents:** Each new conversation is stateless by default. Long-running agents require external memory systems (vector databases, structured logs) to maintain continuity across sessions.

---

## Code Snippets

> Chapter 2 is primarily conceptual and architectural. Hands-on exercises begin in Chapter 3. The pseudocode below illustrates the ReAct loop that every subsequent chapter builds upon.

### The ReAct Loop (Pseudocode)

```
function agent_loop(goal, tools, context):
    while not goal_satisfied(context):
        # REASON: Generate the next action
        thought = model.reason(goal, context)

        # ACT: Invoke the chosen tool
        action = select_tool(thought, tools)
        result = action.execute()

        # OBSERVE: Feed the result back into context
        context = context.append(result)

        # SELF-CORRECT: If a verification gate fails, loop again
        if verification_gate_fails(result):
            context = context.append(error_explanation(result))
            continue

    return context.extract_final_output()
```

### The Four Context Diseases - Quick Reference

| Disease | Root Cause | Symptom |
|---|---|---|
| Context Overload | Too much context injected | Agent ignores later instructions; "Lost in the Middle" |
| Context Poisoning | Contradictory or misleading context | Agent hallucinates or produces incorrect logic |
| Context Distraction | Irrelevant files/docs loaded | Agent solves a different problem than intended |
| Context Confusion | Ambiguous variable names / unclear specs | Agent produces plausible-but-wrong interpretations |

---

## Supplemental Resources

### The ReAct Pattern
- **[ReAct: Synergizing Reasoning and Acting in Language Models (Paper)](https://arxiv.org/abs/2210.03629)** - The original academic paper introducing the ReAct pattern. Yao et al., 2022. Required reading to understand the theoretical foundation of every modern coding agent.
- **[ReAct Pattern Explained - Prompt Engineering Guide](https://www.promptingguide.ai/techniques/react)** - A practical, annotated explanation of ReAct with examples.

### Context Windows and Memory
- **[Lost in the Middle: How Language Models Use Long Contexts (Paper)](https://arxiv.org/abs/2307.03172)** - The research paper documenting the "Lost in the Middle" problem, proving that models attend most strongly to the beginning and end of long context windows.
- **[The Illustrated Transformer - Jay Alammar](https://jalammar.github.io/illustrated-transformer/)** - The clearest visual explanation of how transformers process tokens and maintain attention.

### Tool Use and Agent Frameworks
- **[Anthropic Tool Use Documentation](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)** - Official documentation for how Claude handles tool calling, including structured input/output formats.
- **[OpenAI Function Calling Documentation](https://platform.openai.com/docs/guides/function-calling)** - OpenAI's equivalent guide. Understanding both helps you reason about tool use at the model level, independent of any specific client.

### Foundational Model Architecture
- **[Attention Is All You Need (Original Transformer Paper)](https://arxiv.org/abs/1706.03762)** - The 2017 paper that introduced the transformer architecture. Not required for practical use, but gives useful context for understanding why context windows are finite and why positional attention matters.
