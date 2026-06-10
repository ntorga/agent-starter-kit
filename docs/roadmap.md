# Roadmap: Google Engineering Practices Integration

This document outlines the consolidation of style guides, engineering practices, and development standards to align the **Agent Starter Kit** with Google software engineering standards.

[TOC]

## Status Overview

| Phase | Item | Status |
|---|---|---|
| **Phase 1** | Language-Specific Style Rules (Py, Go, Sh, Rs, Md) | **Done** |
| **Phase 2** | Universal Code Quality Principles | **Done** |
| **Phase 3** | Google-Style Git Workflow | **Done** |
| **Phase 4** | Enhanced Review Pipeline (Design & Doc passes) | **Done** |
| **Phase 5** | Coder & Dispatch Alignment | **Done** |
| **Phase 6** | Boot & Configuration (Language Detection) | **Done** |
| **Phase 7** | Context Maintenance Alignment | **Done** |
| **Phase 8** | Adversarial Plan Review Alignment | **Done** |
| **Phase 9** | CL Author Guidelines (Small CLs) | **Done** |
| **Phase 10** | API Design Standards | **Pending** |
| **Phase 11** | Software Testing Principles (Testing on the Toilet) | **Pending** |
| **Phase 12** | Resiliency and Production Standards (SRE) | **Pending** |
| **Phase 13** | Frontend & Web Style Guides (TS, HTML/CSS) | **Pending** |

---

## Completed Phases

### Phase 1 — Language-Specific Style Rules
- Created individual style guides in `rules/edicts/` for Python, Go, Shell, Rust, and Markdown.
- Aligned with official Google Style Guides (or Rust community standards).

### Phase 2 — Universal Code Quality
- Updated `rules/edicts/code-quality.md` to focus on cross-language principles (KISS, DRY, SRP).
- Added mandatory docstring requirements and data trust boundary rules.

### Phase 3 — Git Workflow
- Migrated `rules/commandments/git.md` to follow [cbea.ms](https://cbea.ms/git-commit/) (Google style).
- Standardized subject lines (50 chars) and body wrapping (72 chars).

### Phase 4 — Review Pipeline
- Updated `personas/reviewer.md` with new **Design** and **Documentation** passes.
- Created `skills/code-design-review.md` and `skills/code-documentation-review.md`.
- Updated SHIELD rubric (`skills/reviewer-self-review.md`) to include new passes.

### Phase 5 — Coder and Dispatch
- Updated `personas/coder.md` to require consulting Google Style Guides.
- Updated GRASP rubric (`skills/coder-self-review.md`) for style scores.
- Updated `skills/dispatch.md` with automatic language detection for rules loading.

### Phase 6 — Boot and Config
- Updated `skills/boot.md` to detect project languages during startup.
- Updated `rules/README.md` with the new rules index.

### Phase 7 — Context Maintenance
- Updated `skills/context-maintenance.md` to reference Google Style Guides in `.context.md` files.

### Phase 8 — Adversarial Plan Review
- Updated `skills/reviewer-architect-adversarial.md` to verify plans against language-specific style guides.

### Phase 9 — CL Author Guidelines
- Updated `personas/coder.md` playbook with "Small CLs" principles.
- Updated `skills/coder-self-review.md` to enforce technical reasoning and impact.

---

## Future Implementation Plan

### Phase 10 — API Design Standards
*   **Goal**: Standardize REST APIs and data structures.
*   **Actions**:
    *   Create `rules/edicts/api-design.md` for resource naming and standard methods.
    *   Update `personas/architect.md` to consult the API design edict.
*   **Reference**: [Google API Design Guide](https://cloud.google.com/apis/design).

### Phase 11 — Software Testing Principles
*   **Goal**: Ensure deterministic and maintainable test suites.
*   **Actions**:
    *   Create `rules/edicts/testing-practices.md` covering hermetic tests and behavior verification.
    *   Align with "Testing on the Toilet" best practices.
*   **Reference**: [Google Testing on the Toilet](https://testing.googleblog.com/).

### Phase 12 — Resiliency and Production Standards (SRE)
*   **Goal**: Integrate production stability into code quality.
*   **Actions**:
    *   Update `rules/edicts/code-quality.md` with resilient error handling (backoff/jitter) and structured logging.
*   **Reference**: [Google SRE Books](https://sre.google/books/).

### Phase 13 — Frontend & Web Style Guides
*   **Goal**: Official guides for TypeScript, HTML, and CSS.
*   **Actions**:
    *   Create `rules/edicts/code-style-typescript.md`.
    *   Create `rules/edicts/code-style-html-css.md`.
*   **Reference**: [Google Style Guides](https://google.github.io/styleguide/).

## See also

*   [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)
*   [Google Go Style Guide](https://google.github.io/styleguide/go/guide)
*   [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
*   [Google Markdown Style Guide](https://google.github.io/styleguide/docguide/style.html)
*   [Google Engineering Practices](https://google.github.io/eng-practices/)
