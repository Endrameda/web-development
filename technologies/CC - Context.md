---
tags:
  - ai/architecture
  - ai/context
status: growing
---

# 

> Context, what is does, and how it works

## When to reach for it
We are using the Context always whenever we are using the LLM models, in our case the Claude Code.

## How
Everything what we are texting in the chat with AI, loading files, docs, everything goes to the context. And we have limitations for  it. Different Language Models will have different size of the Context. The Size of the Context Counts as a tokens, and usually in the modern CC it's 1 mln tokens.

## Gotchas
^d92307

When we are reaching limit of the context, it automatically **Compacts** Compaction is actually summarizes important details and removes unnecessary tool call results to free up the space. And also it can potentially lose the details. 

While working with CC we want to save our context window from fillung up fast. So we have to follow to a few tips.
1. **Be specific** - small and vague prompts might feel faster, but they will take more context and more time to run. And in the long distance without clear instruction CC will be forced to explore the whole our codebase more and make own reasoning and thinking, which takes more context than we will be specific.
2. **Manage MCP servers** - MCP servers are loading all their tools to the context by default, even when we are not using them. If we have some MCP servers which not related to the current project, better to consider turn off them. We can also try the *Skills* which works same as MCP servers but not load everything into the context.
3. **Use Subagents** - Subagents run in Parallel with our main agent but have a completely separate context. For the tasks where we do only need the answer, some result. For example codereview this PR, the Subagent does the work and returns a result. 


### Commands
We have a few commands which can help us to control our context.

- `/context` this commands is shows us how much context we are using, for what we are using, and how much we left it. In one works, it checks the state of the context
- `/clear` this command is clearing our context entirely. We have to use it when we want to clear our context and start everything from scratch. And if we do have some details which Claude have to remember, we have to put them into the `CLAUDE.md` file. So it will not search some important data from the project files
  ![](../source/Pasted%20image%2020260926134136.png)

- `/compact` as said in [Gotchas](#^d92307) this command can help us to compact our context manually. We have to use this command when we are working on some feature but we hit the limitation of the context window, then we can use `/compact` and we will have some important details in our context and still be able to continue to work on the feature.

## Sources
https://academy.claude.com/courses/claude-code-101/context-management