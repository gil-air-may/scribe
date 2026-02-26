# Session Scribe — Claude Skill Objective

## Overview

**Session Scribe** is a Claude skill designed to automatically observe active Claude Code sessions and generate structured, persistent documentation based on the development activity taking place.

The goal is to transform transient AI-assisted coding sessions into durable, navigable project knowledge — including decisions, reasoning, context evolution, and file changes — stored in a separate documentation repository.

---

## Core Problem

Claude sessions are powerful but ephemeral.

While session reloading exists, it:

* Does not structure decisions long-term
* Does not extract architectural rationale
* Does not create durable knowledge artifacts
* Does not organize context evolution over time

As projects grow, valuable reasoning becomes buried inside session transcripts.

---

## Objective

Build a Claude skill that:

1. **Observes live Claude Code sessions**
2. **Detects file modifications and tool usage**
3. **Extracts contextual reasoning and decisions**
4. **Generates structured documentation**
5. **Maintains a separate documentation repository**
6. **Runs asynchronously during active sessions**
7. **Optionally leverages a local model for summarization and organization**

---

## Functional Goals

### 1. File Change Awareness

* Detect which files are modified during the session
* Track tool calls (Write, Edit, Bash, etc.)
* Correlate file changes with reasoning

### 2. Context & Decision Extraction

* Identify:

  * Goals
  * Constraints
  * Trade-offs
  * Plans
  * Final decisions
* Capture architectural reasoning
* Record assumptions and open questions

### 3. Documentation Repository Generation

Create and maintain a structured documentation repository in a separate folder, such as:

```
project-docs/
  README.md
  index.md
  sessions/
  decisions/
  changes/
  knowledge/
```

Artifacts may include:

* Session summaries
* Architectural Decision Records (ADRs)
* Changelogs
* Context summaries
* Prompt history (optional)
* Key commands executed

---

## Asynchronous Operation

The system should:

* Run via Claude Code hooks
* Trigger on relevant tool usage (Write/Edit)
* Execute asynchronously
* Avoid blocking active coding workflows
* Process only incremental transcript updates

---

## Local Model Integration

Integrate with a locally running model (e.g., gpt-oss) to:

* Summarize session reasoning
* Structure raw transcript data
* Extract decisions more intelligently
* Organize documentation artifacts

This reduces dependency on cloud models for meta-processing tasks.

---

## Future Extensions

* MCP integration to expose documentation as structured context
* Searchable documentation interface
* Context pack generation for future sessions
* Decision diff tracking across sessions
* Automatic commit and versioning of documentation repository

---

## Vision

Session Scribe transforms AI coding sessions into:

> A continuously evolving, structured memory system for software development.

Instead of ephemeral chat logs, projects gain:

* Institutional memory
* Traceable architectural decisions
* Explainable evolution
* Context portability
* Searchable development history

This moves Claude from a reactive assistant to a persistent knowledge partner.
