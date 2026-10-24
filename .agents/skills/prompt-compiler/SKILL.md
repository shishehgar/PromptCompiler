---
name: prompt-compiler
description: Compile, refine, or review Persian/Farsi software-development prompts for Codex, Claude Code, OpenCode, or other coding agents. Preserve every stated requirement and hard prohibition; produce an executable English canonical prompt and, when useful, a strictly equivalent Persian translation. Use for requests to write, improve, standardize, translate, or audit coding, debugging, architecture, or automation prompts; do not hijack ordinary implementation requests.
---

# Prompt Compiler — Persian to executable engineering prompts

## Source and scope

This is the reusable, **instruction-only** skill derived from `shishehgar/PromptCompiler`, specifically `prompt_compiler/prompt_engine.py` on the `main` branch. It encapsulates the original *prompt-compilation policy*, not the FastAPI service. **It does not require Gemini, an API key, a running server, or network access.** Read the source file if you need to reconcile behavior with a newer project version; do not silently add capabilities not supported by the user's request.

## Trigger and operating modes

- **COMPILE** — when the user wants a prompt for a coding agent, produce a portable, actionable English prompt. If requested or useful for review, also provide the faithful Persian equivalent and a concise requirements audit.
- **REVIEW** — when the user supplies an existing prompt for checking, identify omitted requirements, weakened exclusions, invented scope, and contradictions. Return a corrected prompt and a short explanation of material changes.
- **DIRECT TASK** — when the user asks you to write, debug, or modify code *now* rather than asking for a prompt, use the requirement-integrity checklist internally and execute the authorized task; **do not replace implementation with a compiled prompt** or inflate the answer.
- Trigger examples: `$prompt-compiler`, `کامپایل پرامپت`, `پرامپت برنامه‌نویسی را استاندارد کن`, `write a prompt for Codex`, `refine this debugging prompt`.
- Do not trigger for ordinary translation or creative writing unrelated to programming unless the user explicitly requests this skill.

## Mandatory compilation pipeline

1. Identify the user's actual goal and the category, such as `coding`, `debugging`, `architecture`, or `automation`.
2. Extract all stated functional requirements, existing environment details, affected paths, inputs/outputs, acceptance criteria, dependencies, and prohibitions.
3. Classify information separately as:
   - `preserved_requirements`: user-stated or unambiguously implied requirements;
   - `assumptions`: truly necessary temporary assumptions, never invented facts;
   - `missing_information`: only material unknowns that block a correct result;
   - `suggestions`: optional improvements **outside** the canonical prompt.
4. Construct and finalize `english_prompt` first. Treat it as the **single source of truth**.
5. Only after the English version is final, translate it into `persian_prompt` when needed. Preserve technical identifiers unchanged; do not independently recompile the original Persian input.
6. Check both versions for identical scope, requirements, prohibitions, qualifiers, and expected behavior. Resolve discrepancies by correcting the Persian translation against the English baseline.
7. Keep length proportional to the task's complexity. Skip empty or unnecessary sections.

## Non-negotiable integrity rules

- **No invented stack.** Never assume React, Docker, PostgreSQL, Redis, deployment architecture, an API, libraries, authentication, business rules, or any other technical choice absent explicit user instruction or verified existing-project context.
- **Preserve negative constraints as hard constraints.** `Do not use Docker` must not become `prefer not to use Docker`. A proposed suggestion must not contradict any explicit prohibition.
- **No silent scope expansion.** Do not turn a bug fix into a refactor, redesign, dependency upgrade, or infrastructure change without authorization.
- **Existing projects first.** If files or repository context are accessible, inspect relevant source and instructions before proposing changes. Do not misrepresent assumptions as repository facts.
- **Compatibility with existing controls.** Follow the project's `AGENTS.md`, coding standards, security instructions, and, when available, its memory/checkpoint policy. This skill supplements them; it does not override higher-priority instructions or user constraints.
- **Keep security and secrets intact.** Never put keys, tokens, credentials, private endpoints, or their values into reusable prompts, Git history, or logs. Use placeholders and secret-management mechanisms.
- **No fabricated verification.** Never claim that code was executed, tests passed, or a deployment succeeded unless actually verified. Request tests or deployment procedures only where useful, required by the repository policy, or necessary for the user's stated outcome.
- **Ask selectively.** List genuinely blocking missing information; where reasonable, proceed using clearly labeled assumptions without guessing technical facts.

## Prompt structure

Use only helpful sections, drawn from:

`Context / Existing Environment / Task / Functional Requirements / Technical Constraints / Forbidden Changes / Expected Behavior / Acceptance Criteria / Validation / Deliverables`.

For debugging, distinguish **observed behavior**, **expected behavior**, **reproduction information**, **known logs/errors**, and **uncertain root cause**. Require evidence-based diagnosis, not a fabricated explanation.

For infrastructure or repository modifications, emphasize the specified scope and explicitly preserve unrelated services, files, configuration, data, and deployment behavior.

## Output contract

For ordinary COMPILE mode, format:

1. **English prompt (canonical)** — directly reusable in a coding agent.
2. **پرامپت فارسی** — only if requested or useful for user validation; a faithful translation of the canonical prompt.
3. **Integrity audit** — short grouped notes on preserved requirements, assumptions, blocking missing information, and *optional* suggestions, omitting empty groups when clarity benefits.

When JSON is requested, return valid JSON (without a Markdown fence if machine consumption is intended) with the source application's compatible keys:

```json
{
  "detected_category": "coding",
  "detected_subcategory": "",
  "detected_purpose": "",
  "english_prompt": "",
  "persian_prompt": "",
  "preserved_requirements": [],
  "assumptions": [],
  "missing_information": [],
  "suggestions": []
}
```

Do not translate the audit arrays into the Persian prompt: `persian_prompt` translates `english_prompt` only.

## Interoperability

- When `prompt-master` is also available, use **Prompt Compiler** for intent, requirements, prohibitions, and bilingual consistency, and use **Prompt Master** only for stylistic/agent-specific optimization that cannot change or add requirements.
- When `memory-save-optimizer` is available, honor its context-budget and checkpoint principles without sacrificing relevant project facts.
- Any existing shared coding standard takes precedence over optional recommendations; do not import undocumented standards.
- The source project's optional FastAPI endpoint `POST /translate` may be used only when the user explicitly wants the live application and its prerequisites are configured. The skill itself operates without that endpoint.

## Final preflight

Confirm: correct user intent; no lost requirement; no weakened prohibition; no invented framework or fact; no unrelated changes; only material missing information; optional ideas separated; executable clarity; proportional length; and English/Persian semantic equivalence when both are produced.
