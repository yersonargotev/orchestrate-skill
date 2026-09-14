# Astra orchestration: completion, waiting, and runtime capabilities

Checked 2026-09-13. This note separates official product documentation, the current session's tool contract, and community workflow choices.

## Official guidance

- Codex supports parallel subagents and consolidates requested results before answering. Its documentation describes orchestration and waiting, but does not establish that ending the parent's user-facing turn automatically schedules a new parent turn when a child finishes. Explicit project or skill instructions can authorize delegation. [Subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)
- Model and reasoning settings can come from explicit spawn values, agent defaults, parent inheritance, and custom agent files. Custom agent file settings take precedence. Supported effort levels and availability depend on the selected model and product. This makes a universal hardcoded role-to-model table inappropriate unless the workflow deliberately requires that policy. [Subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)
- Astra's API supports asynchronous tool calls, but the application still executes tools and manages pending work. This is evidence for harness-managed concurrency, not proof of Codex parent wake-up behavior after a final response. [Model guidance](https://developers.openai.com/api/docs/guides/latest-model)
- Eric Provencher's September 11 article recommends concise applicability descriptions, progressive disclosure, task-specific document access, and less prescriptive scaffolding. It does not discuss child completion notifications or ending a parent turn. [Rethinking skills and prompts for GPT-6 Astra](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra)

## Current session contract

The collaboration tool definitions supplied to this session provide stronger local evidence than general product prose:

- Child final answers are delivered to the parent; inter-agent messages arrive at execution boundaries.
- `wait_agent` waits for mailbox updates, including final-status notifications. It reports which agents have updates, rather than returning message content itself.
- `send_message` does not trigger an idle agent's turn. `followup_task` explicitly does.
- `spawn_agent` exposes model and effort overrides, but full-history forks inherit settings and reject overrides. Some named roles have fixed models and efforts.

These observations support consuming completion events and avoiding repeated status polling. They do **not** establish a universal guarantee that a root `final` response will automatically resume. A portable skill should use a documented yield/resume mechanism when available, otherwise the available wait primitive. It should distinguish yielding execution from declaring the user's objective complete.

## Community references and interpretation

The supplied pvncher statement about dispatching another thread, ending the turn, and being awakened by a child is user-provided anecdotal evidence. Its original post and applicable runtime were not independently verified. Preserve the useful intent—avoid spinning, polling, and unnecessary coordination messages—while making end-turn resumption conditional on the active runtime contract.

The supplied community skill chooses Astra for orchestration/review and Luna for routine execution. It requires actual delegation under broad triggers, dispatches independent work before waiting, and prohibits final completion while required workers remain running. Those are the author's workflow policies, not OpenAI guarantees. Its model requirements should not override the active runtime's supported parameters or role constraints. [Community Astra orchestrator skill](https://github.com/donvito/codex-astra-luna-orchestrator/blob/main/profiles/pro/agents/skills/astra-orchestrator/SKILL.md)

## Proposal and validation

The proposed [skill](../skills/orchestrate/SKILL.md) keeps a single entrypoint with five short paragraphs: delegation boundaries, model selection, worker ownership, completion-driven waiting, and integration. It replaces broad size-based activation with a delegation-benefit trigger. Model selection remains a task-based preference rather than a required Astra/Luna topology; no extra profiles, routing scripts, or configuration files are needed.

An independent static review considered completion notifications without post-final reactivation, explicit wake support, unavailable overrides, blockers, and shared writers. It found that a worker blocker could prematurely end the root task; the final instruction instead requires resolving or reassigning work before reporting a dependency on unavailable input or access. This is a prompt review, not an end-to-end wake-up test or a measured cost/latency comparison.

The skill frontmatter validator and the repository's pinned Managed Pack validator passed. The pack marks the modified skill resource as `adapted` while preserving its origin and MIT notice. The research note is outside the installed skill closure.
