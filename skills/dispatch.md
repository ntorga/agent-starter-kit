---
shortDescription: Assembles sub-agent prompts with task brief and routes to the correct provider.
usedBy: [maestro]
version: 0.2.2
lastUpdated: 2026-03-22
---

## Purpose

Every sub-agent starts cold. It has no rules, no memory, and no awareness of the project it is about to work on. This skill defines how the Maestro assembles the initial prompt that boots a sub-agent into a ready state, and routes it to the correct provider based on the persona's preferred model.

## Terminology

A **sub-agent** is a persona defined in this framework — nothing else. The terms "sub-agent" and "persona" are interchangeable throughout this skill. Sub-agents are **not** host-runtime features (IDE subprocesses, tool-provided agents, or built-in workers). The Maestro must never route work to a host-runtime agent when a framework persona exists for the job.

To discover available sub-agents, read:

- **`personas/README.md`** — lists every persona and its purpose.

This is the only registry. If a persona is not listed there, it does not exist. The Maestro must consult this registry during the Route step before dispatching.

## Procedure

1. **Identify the host runtime.** Run `ps -p $PPID -o comm=` and match the output against the Providers table (e.g., `claude` → Claude Code). Store the result in session state — the host runtime does not change mid-conversation.

2. **Extract routing fields.**

   ```bash
   sed -n '/^---$/,/^---$/{ /^\(preferredModel\|modelTier\):/p }' personas/<name>.md
   ```

3. **Select the provider and model.** Resolve `preferredModel` and `modelTier` against the Providers table. If `preferredModel` is omitted, use the host runtime's provider. The persona's `modelTier` is a floor — upgrade one tier when the task demands multi-step reasoning across system boundaries (e.g., cross-layer architectural changes, security/auth logic, or production deployment pipelines).

4. **Decide how to dispatch.** Look up the persona's `preferredModel` in the Providers table to find its CLI column. Then:
   - **Native dispatch** — the provider's CLI matches the host runtime. Use the host's built-in subagent mechanism (e.g., Task tool for Claude Code). Do not shell out to the same tool's CLI.
   - **CLI dispatch** — the provider's CLI does not match the host runtime. Shell out to the provider's CLI tool (see CLI Dispatch section).
   - If the preferred provider's CLI is not installed or unreachable, fall back to native dispatch and record the deviation in session memory.

5. **Strip the frontmatter.** Wrap the result in `<agent>` tags verbatim — do not summarize, paraphrase, or shorten the persona file. The full text must arrive exactly as written. Each dispatch targets exactly one agent — never multiple personas in a single prompt.

   ```bash
   sed '/^---$/,/^---$/d' personas/<name>.md
   ```

6. **List the commandments (scoped).** Consult `rules/README.md` and select only commandments whose scope matches the task category. List their file paths in `<commandments>` tags — do not inline the file contents. The persona has file access and will read them directly. If no commandments match, omit the block entirely. When the task involves code changes — even if the persona does not write code (e.g. architect planning implementations) — include `coding`-scoped rules so the persona's output aligns with the conventions the coder will follow.

7. **List relevant skills.** Consult `skills/README.md` and identify skills that would help the persona complete the task. List their file paths in `<skills>` tags. If no extra skills are relevant, omit the block entirely.
   - Always include `skills/error-recovery.md` for Coder dispatches.
   - Always include `skills/testing.md` for Coder dispatches.
   - Include `specs/README.md` in the notes if a feature spec exists for the task.

8. **Write the task brief.** Translate the user's intent into actionable instructions, wrapped in `<task>` tags. The brief must contain:
   - **Intent** — what the user wants accomplished, in the Maestro's words.
   - **Entities** — key nouns: files, modules, endpoints, services.
   - **Constraints** — deadlines, tech stack limits, scope boundaries. Omit if none.
   - **Acceptance criteria** — what "done" looks like. If the user did not provide criteria, the Maestro defines them.

9. **Compose and dispatch.** Assemble the final prompt:

```markdown
<agent>
  [persona file without frontmatter]
</agent>

<commandments>
  [file paths to scoped commandments — omit block if no scope matches]
</commandments>

<skills>
  [file paths to relevant skills — omit block if none apply]
</skills>

<notes>
  - You are running non-interactively — there is no user on the other end to answer prompts. Never pause to wait for input. If you lack information that is critical to proceed, stop immediately and return a handoff explaining what is missing. A new run will be dispatched with the missing context.
  - If you encounter pre-existing issues (bugs, tech debt, code smells) outside the current task's scope, list them in a separate `## Discovered Issues` section at the end of your handoff. Do not fix them — just report what you found and where.
</notes>

<task>
  [task brief]
</task>
```

## Providers

Each row maps a provider to its `preferredModel` value, CLI tool, and concrete models per tier. Tier classes: **tier-1** = fast/cheap, **tier-2** = balanced, **tier-3** = reasoning/smartest.

| Provider    | `preferredModel` | CLI        | tier-1                               | tier-2                           | tier-3                           |
| ----------- | ---------------- | ---------- | ------------------------------------ | -------------------------------- | -------------------------------- |
| Claude Code | `claude`         | `claude`   | Haiku                                | Sonnet                           | Opus                             |
| Qwen        | `qwen`           | `opencode` | bailian-coding-plan/qwen3-coder-next | bailian-coding-plan/qwen3.5-plus | bailian-coding-plan/qwen3.5-plus |

## CLI Dispatch

When the host runtime differs from the target provider, pipe the assembled prompt through `stdin`:

```bash
cat << 'EOF' | [cli-tool] [flags]
[assembled prompt]
EOF
```

Provider-specific flags (add entries as you integrate providers):

- **`claude`**: `--model [model]` (accepts `haiku`, `sonnet`, `opus`). Do **not** use `--print` (`-p`) — it bypasses permission checks.
- **`opencode`**: `OPENCODE_EXPERIMENTAL_BASH_DEFAULT_TIMEOUT_MS=600000 opencode run --model [provider/model]`. The env var raises the bash timeout from 120s to 600s. Optional: `--thinking` (shows thinking blocks).

## Guardrails

- Never dispatch without acceptance criteria. If the user was vague, that is the Maestro's problem to solve before dispatch, not the sub-agent's.
- Never copy-paste the user's raw message as the task brief. The Maestro's job is to interpret and structure, not relay.
