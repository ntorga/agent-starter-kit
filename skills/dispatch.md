---
shortDescription: Assembles sub-agent prompts with task brief and routes to the correct provider.
usedBy: [maestro]
version: 0.2.3
lastUpdated: 2026-03-27
---

## Purpose

Every sub-agent starts cold. It has no rules, no memory, and no awareness of the project it is about to work on. This skill defines how the Maestro assembles the initial prompt that boots a sub-agent into a ready state, and routes it to the correct provider based on the persona's preferred model.

## Terminology

A **sub-agent** is a persona defined in this framework — nothing else. The terms "sub-agent" and "persona" are interchangeable throughout this skill. Sub-agents are **not** host-runtime features (IDE subprocesses, tool-provided agents, or built-in workers). The Maestro must never route work to a host-runtime agent when a framework persona exists for the job.

To discover available sub-agents, read:

- **`personas/README.md`** — lists every persona and its purpose.

This is the only registry. If a persona is not listed there, it does not exist. The Maestro must consult this registry during the Route step before dispatching.

## Procedure

1. **Identify the host runtime.** Run `ps -p $PPID -o comm=` and match the output against the Providers table (e.g., `claude` → Claude Code, `codex` → Codex CLI, `cursor-agent` → Cursor CLI). Store the result in session state — the host runtime does not change mid-conversation.

2. **Extract routing fields.**

   ```bash
   sed -n '/^---$/,/^---$/{ /^\(preferredModel\|modelTier\):/p }' personas/<name>.md
   ```

3. **Select the provider and model.** Resolve `preferredModel` and `modelTier` against the Providers table. If `preferredModel` is omitted, use the host runtime's provider. The persona's `modelTier` is a floor — upgrade one tier when the task demands multi-step reasoning across system boundaries (e.g., cross-layer architectural changes, security/auth logic, or production deployment pipelines).

4. **Decide how to dispatch.** Look up the persona's `preferredModel` in the Providers table to find its CLI column. Then:
   - **Native dispatch** — if the current host runtime can satisfy the target provider through its native subagent mechanism, use that path instead of shelling out. Examples: Claude Code Task tool, Codex subagent environment, Cursor's native agent/subagent flow.
   - **CLI dispatch** — if the current host runtime cannot satisfy the target provider natively, shell out to the provider's external CLI tool (see CLI Dispatch section).
   - If the preferred provider's external CLI is not installed or reachable, but the host can still satisfy the request natively, fall back to native dispatch and record the deviation in session memory.

5. **Strip the frontmatter.** Wrap the result in `<agent>` tags verbatim — do not summarize, paraphrase, or shorten the persona file. The full text must arrive exactly as written. Each dispatch targets exactly one agent — never multiple personas in a single prompt.

   ```bash
   sed '/^---$/,/^---$/d' personas/<name>.md
   ```

6. **List the rules (scoped).** Consult `rules/README.md` and select commandments, edicts, and counsel whose scope matches the task category. List their file paths in `<rules>` tags — do not inline the file contents. The persona has file access and will read them directly. If no rules match, omit the block entirely. When the task involves code changes — even if the persona does not write code (e.g. architect planning implementations) — include `coding`-scoped rules so the persona's output aligns with the conventions the coder will follow.

7. **List relevant skills.** Consult `skills/README.md` and identify skills that would help the persona complete the task. List their file paths in `<skills>` tags. If no extra skills are relevant, omit the block entirely.

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

<rules>
  [file paths to scoped rules — omit block if no scope matches]
</rules>

<skills>
  [file paths to relevant skills — omit block if none apply]
</skills>

<notes>
  - You are running non-interactively — there is no user on the other end to answer prompts. Never pause to wait for input. If you lack information that is critical to proceed, stop immediately and return a handoff explaining what is missing. A new run will be dispatched with the missing context.
  - If you encounter pre-existing issues (bugs, tech debt, code smells) outside the current task's scope, list them in a separate `## Discovered Issues` section at the end of your handoff. Do not fix them — just report what you found and where.
  - If you hit the same failure three times, read and follow `skills/loop-recovery.md`.
</notes>

<task>
  [task brief]
</task>
```

## Providers

Each row maps a provider to its `preferredModel` value, CLI tool, and concrete models per tier. Tier classes: **tier-1** = fast/cheap, **tier-2** = balanced, **tier-3** = reasoning/smartest.

| Provider    | `preferredModel` | CLI            | tier-1                               | tier-2            | tier-3                         |
| ----------- | ---------------- | -------------- | ------------------------------------ | ----------------- | ------------------------------ |
| Claude Code | `claude`         | `claude`       | Haiku                                | Sonnet            | Opus                           |
| Codex CLI   | `codex`          | `codex`        | `gpt-5.4-mini`                       | `gpt-5.3-codex`   | `gpt-5.4`                      |
| Cursor CLI  | `cursor`         | `cursor-agent` | `auto`                               | `auto`            | `claude-4.6-opus-max-thinking` |
| Qwen        | `qwen`           | `opencode`     | bailian-coding-plan/qwen3-coder-next | bailian-coding-plan/qwen3.5-plus | bailian-coding-plan/qwen3.5-plus |

## CLI Dispatch

When the host runtime differs from the target provider, dispatch using the provider's accepted input mode. Prefer `stdin` when the CLI supports it; otherwise pass the assembled prompt as a single non-interactive argument.

```bash
cat << 'EOF' | [cli-tool] [flags]
[assembled prompt]
EOF
```

Provider-specific flags (add entries as you integrate providers):

- **`claude`**: `--model [model]` (accepts `haiku`, `sonnet`, `opus`). Do **not** use `--print` (`-p`) — it bypasses permission checks.
- **`codex`**: `exec - --model [model] --sandbox workspace-write --skip-git-repo-check -C [workspace]`. Use `exec -` to read from `stdin` and keep dispatch non-interactive/reproducible. Add `--full-auto` only when safety boundaries are already enforced by the environment.
- **`cursor-agent`**: `--model [model] "[assembled prompt]"` in headless/non-interactive mode. Add `--workspace [workspace]` only when explicitly provided. Add `--trust` only under externally enforced safety controls.
- **`opencode`**: `OPENCODE_EXPERIMENTAL_BASH_DEFAULT_TIMEOUT_MS=600000 opencode run --model [provider/model]`. The env var raises the bash timeout from 120s to 600s. Optional: `--thinking` (shows thinking blocks).

## Guardrails

- Never dispatch without acceptance criteria. If the user was vague, that is the Maestro's problem to solve before dispatch, not the sub-agent's.
- Never copy-paste the user's raw message as the task brief. The Maestro's job is to interpret and structure, not relay.
