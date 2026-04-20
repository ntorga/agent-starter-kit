# Agent Starter Kit

> Speak naturally. A Maestro agent breaks your request into tasks and routes each one to a specialized AI model.

The scaffold for a fully customizable, **multi-model** AI harness in **pure natural language**. The smartest model orchestrates the workflow while cheaper/faster ones handle the less complicated bits, **extending your premium coding plan (such as Claude Code)**.

It's model-agnostic: orchestrate from Claude, plan on GLM, review on Qwen, or any combination you want.

![Boot sequence demo](docs/demo.png)
*Maestro booted on a Clean Architecture Go project — gitignore, auto-update, memory, rules, and 35 context files created automatically.*

This is a **foundation**, not a finished product. It ships only what an average developer needs out of the box — general-purpose personas, common workflow skills, and unopinionated rules. Anything domain-specific or highly opinionated belongs in your own fork. Clone it, extend it, make it yours — or better yet, make one for your entire company to use.

## Setup

1. Clone into `.agents/` inside your project:

   ```bash
   cd /path/to/your/project
   git clone git@github.com:ntorga/agent-starter-kit.git .agents
   ```

2. Symlink the entry file to the project root:

   ```bash
   ln -s .agents/AGENTS.md AGENTS.md
   ```

3. Start the AI agent (e.g., `claude`, or whatever CLI you use).
4. Say **"Please comply with AGENTS.md."** — this boots the Maestro and loads the framework.
5. The Maestro orchestrates everything. On first run, it automatically dispatches the Contextualizer to map the codebase.
6. Customize — add personas, rules, skills, and providers to fit your project (see Customization below).

## How It Works

The **Maestro** is the conductor. It receives user requests, decomposes them, and dispatches work to specialized personas:

- **Architect** — plans implementations, defines before/after states
- **Coder** — writes software following the plan
- **Reviewer** — checks work for correctness and quality
- **Contextualizer** — documents project structure for orientation

Each persona has an identity (who they are), a playbook (what they do), a handoff format (what they deliver), and red lines (what they must not do). Each persona also declares a `preferredModel` — the Maestro uses this to route work to the right provider automatically.

The framework **learns as it works**. Corrections, preferences, and lessons are captured to long-term memory and carried into every future session. Interrupted work is tracked in session files so the next boot can resume where the last one stopped.

## Structure

```
personas/    Specialized AI roles (who does the work)
rules/       Constraints organized by authority level
skills/      Reusable procedures and protocols
```

## Rules Hierarchy

- **Commandments** — absolute, never bypassed
- **Edicts** — authoritative within scope, not bent
- **Counsel** — wise guidance, may be deviated from with justification

## Skills

Skills codify procedures that personas reference. They answer "how to do X" so personas can focus on "what to do."

- **agent-memory** — long-term and session memory across sessions
- **boot** — session startup sequence
- **code-coherence-review** — logic coherence, correctness, and structural integrity checks
- **code-quality-review** — rules-walk procedure for coding standards compliance
- **code-sec-review** — OWASP-aligned security code review checklist
- **context-maintenance** — schema and rules for `.context.md` files
- **dispatch** — how the Maestro assembles and sends work to personas
- **loop-recovery** — structured recovery and escalation for retry loops
- **plan-critique** — adversarial plan validation before implementation
- **reviewer-handoff** — structured review summary format with verdict logic
- **task-tracking** — file-based to-do for multi-step work

## Customization

- **Dispatch** — edit `skills/dispatch.md` to match your CLI agents. The Providers table and CLI Dispatch section are pre-configured for Claude Code and [opencode](https://github.com/anomalyco/opencode) (Qwen). Add rows for any other provider/model you use.
- Add new personas to `personas/` following the schema in `personas/README.md`
- Add rules to `rules/commandments/`, `rules/edicts/`, or `rules/counsel/`
- Add skills to `skills/` following the schema in `skills/README.md`
- Modify existing files to match your project's needs

Each directory has a README with the full schema definition.

## Thinking Budget for MoE Providers

If you use Mixture-of-Experts models (Kimi, Qwen, DeepSeek, or similar), cap thinking tokens at **16,000** in your CLI's configuration. Research across 121+ code review dispatches found that MoE models regress past this threshold — higher budgets cause models to qualify findings, soften severity, and rationalize away bugs they previously found. Dense models (Claude, Seed) do not exhibit this regression and can use higher budgets safely. See [Overfed, Overthought, Overasked](https://ntorga.com/overfed-overthought-overasked-stop-sabotaging-your-ai/) for the full research.

## FAQ

### Why this over other harnesses?

Frameworks like GSD, HumanLayer, and OpenDev are software — they require a programming language, dependencies, and runtime integration. This kit is **pure natural language**. Every persona, rule, and skill is a Markdown file. There is no code to install, no SDK to learn, no build step to maintain.

Other harnesses are bulldozers — heavy with built-in packages, skills, and guidance that consume tokens every session, even when most of it is irrelevant to your project. This kit is a **scalpel**: minimal by design, meant to be extended with your own tailored personas, rules, and skills. You pay only for the context you actually need. Add a persona by writing a `.md` file. Change a rule by editing a line. Swap a provider by updating a table row.

### Why multi-model?

Coding plans are routinely quantized and rate-limited weeks after launch — the version you fell in love with gradually loses sharpness as the provider optimizes for throughput. A multi-model harness fights this in three ways:

1. **Resilience.** Spreading work across providers means you're less affected when any single plan degrades. If one provider tightens limits or loses quality, shift that persona's `preferredModel` to another row in the Providers table.
2. **Token conservation.** The orchestrator (Maestro) only handles routing and decomposition. Token-heavy roles like Architect and Coder are delegated to other capable models, so your premium plan lasts longer.
3. **Fresh eyes.** Different models catch different things. A reviewer running on a separate provider will flag issues that the coder's model normalized.

### What do I need to run this?

A coding plan or API key for each provider you route to. We recommend coding plans — **Claude Code** (Anthropic), **Codex** (OpenAI), and **Alibaba Model Studio** (Qwen) offer flat-rate pricing with generous token allowances designed for agentic workflows. API keys work too, but plans are more cost-effective for sustained use. Each provider needs its CLI tool installed (e.g., `claude` for Claude Code, `opencode` for Qwen). If you only route to one provider, one plan is enough.

### How does the Maestro use multiple models from a single CLI?

The dispatch skill (`skills/dispatch.md`) handles this automatically. When a persona's `preferredModel` matches the host runtime (e.g., you're running Claude Code and the persona wants `claude`), the Maestro dispatches natively using the host's built-in subagent mechanism (e.g., the Task tool). When the `preferredModel` points to a different provider (e.g., `qwen`), the Maestro shells out to that provider's CLI tool (e.g., `opencode`) by piping the assembled prompt via `stdin`. The Providers table in `skills/dispatch.md` maps each model to its CLI — add rows for any provider you want to use.

### Can I use this with just one model?

Yes. Set every persona's `preferredModel` to your host runtime (e.g., `claude`) and the framework runs entirely within a single provider. You still benefit from the structured decomposition, review pipeline, and long-term memory — just without the multi-model routing.

### How should I assign models to personas?

Each persona declares a `preferredModel` in its frontmatter — this is what the Maestro uses to route work. Keep premium models as the orchestrator (Maestro makes routing decisions and manages context — short, high-leverage interactions worth the cost). For the rest, match the model to the persona's job using role-specific benchmarks:

- **Coder** — [LiveCodeBench](https://artificialanalysis.ai/evaluations/livecodebench) (real-world coding tasks)
- **Architect** — [Artificial Analysis Long Context Reasoning](https://artificialanalysis.ai/evaluations/artificial-analysis-long-context-reasoning) (multi-step reasoning across large contexts)
- **Reviewer** — [IFBench](https://artificialanalysis.ai/evaluations/ifbench) (instruction following and constraint verification)

These benchmarks are examples — new ones emerge frequently. Pick whatever benchmark best measures the capability each role needs, then set `preferredModel` accordingly in the persona's frontmatter.

### Does the framework auto-update?

Yes. On every session start, the boot sequence runs `git -C .agents pull`. If the pull brings changes, the Maestro reads the changelog, purges any long-term memory entries that the update made obsolete, and reboots with the new instructions.

### What does "model-agnostic" mean? Which models are supported?

Any model with a CLI tool that can accept a prompt via `stdin` works. As a quality floor, we recommend models scoring 1300+ ELO on [GDPval-AA](https://artificialanalysis.ai/evaluations/gdpval-aa) — a benchmark for general-purpose reasoning. Below that threshold, personas may struggle with multi-step tasks.
