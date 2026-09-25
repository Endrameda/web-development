---
tags:
  - ai/architecture
status: growing
---

# CC - Agentic Loop

> How Claude Code works: prompt → agentic loop → done.
## When to reach for it

![](../source/Pasted%20image%2020260925110043.png)
## How it works
The best explanation is the loop itself.

1. Write a prompt into CC.
2. CC gathers the context it needs and interacts with the model, which either calls a tool or returns text CC can act on.
3. For example, editing a `.ts` file.
4. CC verifies the result and determines whether it achieved what the prompt asked for.
5. If it did, CC returns the result and waits for the next prompt. If it didn't, it loops back and tries again, until the result is reached.

In the screenshot above, context can be added into the loop, and the loop itself can be interrupted, steered, or given more context.

### Context
CC has a **Context Window** that determines how much content conversation, files, commands, outputs, etc.  It can store and reference. After it reaches this limit, CC compacts the conversation, automatically deciding what to remove or summarize, to bring the **Context Window** back down to a usable size.

### Tools
Tools are the main thing that make agents work. On their own, AI assistants mostly take in text and return text as a result. Tools let CC determine _when_ to execute code to get closer to completing a task. This could be a file reading tool, a web search tool, or any number of other capabilities. Claude Code uses semantic understanding to determine when to call a tool and how to use the output.

### Permissions 
CC has several permission modes

- **Manual:** Claude asks for explicit permission before editing a file or running a shell command.
- **Auto-accept:** Files are edited without asking, but commands still require approval.
- **Plan mode:** Uses read-only tools to compile a plan of action before starting any work.
- **Auto mode:** Claude works without permission prompts while a classifier checks each action in the background, targeted at blocking actions that are irreversible, destructive, or aimed outside your environment. When something is blocked, Claude usually finds a safer approach or asks you for the go-ahead.
![](../source/Pasted%20image%2020260925111536.png)