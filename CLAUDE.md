# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

This is a **Claude Code skill** — not a traditional software project. There are no build steps, package dependencies, or test runners. The repo defines a skill (`SKILL.md`) and a living knowledge base (`references/`) that is loaded at skill invocation time.

The skill is evaluated via `evals/evals.json` using an external harness (not a local test command). To run evals, use the skill-creator skill: `/skill-creator`.

## Architecture

### Core files

- **SKILL.md** — the skill definition. Contains the full behavioral spec: modes, templates, calibration, web search queries, and reference file guidance. This is the source of truth for how the skill behaves.
- **references/frameworks.md** — status and selection guidance for 12+ agentic frameworks (LangGraph, CrewAI, AutoGen, OpenAI Agents SDK, Claude Agent SDK, Pydantic AI, Smolagents, etc.)
- **references/concepts.md** — core vocabulary and patterns: ReAct, memory systems, multi-agent topologies, evaluation, failure modes
- **references/reading-list.md** — papers, blogs, and repos worth tracking
- **knowledge/briefing-log.md** — append-only log of completed briefings; read at the start of every briefing to avoid repetition
- **evals/evals.json** — 3 eval test cases (framework selection, briefing, tool failure handling)

### Three operating modes

The skill dispatches on user intent:

1. **Briefing mode** — triggered by "catch me up", "what's new", "briefing". Reads briefing-log.md first, attempts web search, synthesizes into the 5-section format (TL;DR / Framework Updates / Research Highlights / Industry Moves / What to Watch), then appends a one-line entry to briefing-log.md.

2. **Q&A mode** — triggered by specific questions. Answers from training first; reads `references/frameworks.md` only for framework comparison/selection questions. Does not read reference files for patterns, concepts, or debugging questions (no value added, just latency).

3. **Knowledge Update mode** — triggered by "update your knowledge base", "refresh what you know about X". Reads the relevant reference file, web searches for new developments, rewrites/appends the file with updated content and a date.

### No-web-search fallback

When web search is unavailable during briefing mode, the skill must open with a disclosure block (see SKILL.md lines 27-37), use reference files as source material, mark time-sensitive claims with `[verify]`, and still use the full briefing template.

### Reference file update rule

Update reference files only when you discover something that genuinely contradicts or meaningfully extends what's there. Always include the date of the update. Don't update as a default — only when the delta matters.

## Eval structure

Each eval in `evals/evals.json` has a `prompt`, `expected_output`, and `assertions` array. The assertions are the pass/fail criteria. Current evals cover:
- Framework selection (LangGraph vs CrewAI for a specific workflow)
- Briefing output (all 5 sections, 2+ sourced URLs, 2025/2026 content)
- Tool failure / infinite loop handling (iteration limits, error message design, fallback strategies)
