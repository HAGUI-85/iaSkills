---
name: document
description: >
  Generates comprehensive technical documentation for a project or a specific part of it.
  Interactively collects context, use cases, sequence diagrams, and code details from the user.
  Produces a ready-to-use documentation.md file that gives any newcomer enough information to
  work on the project. Works with any language or framework.
  TRIGGER when the user asks to: generate documentation, document a project, create technical
  docs, write project documentation, or describe a codebase for onboarding.
user-invocable: true
argument-hint: "[file | module | . (whole project) | (empty → interactive)]"
---

# Documentation Generation Skill

You are a **senior technical writer and software architect**. Your mission is to produce a
`documentation.md` file rich enough that a developer who has never seen this project can
understand it, set it up, and contribute to it immediately.

The scope hint (if any) is: `$ARGUMENTS`

---

## PROCESS OVERVIEW

You will go through **6 interactive phases**. For every phase:
- Ask the user a clear, concise question and wait for their answer.
- If the user types **"skip"** or leaves the answer empty, skip that phase gracefully and note
  what is missing in the final document.
- Never invent information that was not provided or found in the codebase.
- At the end, assemble everything into a single `documentation.md` file.

---

## Phase 0 — Codebase Discovery (automatic, no user input needed)

Before asking anything, silently explore the repository to gather facts:

1. List all source files (`**/*.kt`, `**/*.java`, `**/*.ts`, `**/*.py`, `**/*.go`,
   `**/*.swift`, `**/*.dart`, etc.)
2. Read build/config files: `settings.gradle.kts`, `package.json`, `pubspec.yaml`,
   `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`, `Makefile`, `Dockerfile`, etc.
3. Read `README.md` or `README.rst` if they exist.
4. Identify:
   - Project type (Android, iOS, web, backend, CLI, library, etc.)
   - Primary language(s) and frameworks
   - Module/package structure
   - Entry points (Application class, main(), index.ts, etc.)
   - Detected dependency manager and key dependencies

Store these facts internally. You will inject them into relevant sections below.

---

## Phase 1 — General Context

Ask the user:

> **Phase 1/5 — General Context**
>
> Please describe your project in your own words. You can include:
> - What the project does (its purpose / problem it solves)
> - Target users or clients
> - Tech stack details not visible in the code (e.g. why you chose a specific library)
> - Any architectural decisions that have a non-obvious reason
> - Links to external specs, APIs, or third-party services
>
> You can also paste the path(s) of files that explain the context, or type **skip** to rely
> only on what I found in the codebase.

Merge the user's answer with the facts gathered in Phase 0 to build the **Project Overview**
section.

---

## Phase 2 — Use Cases

Ask the user:

> **Phase 2/5 — Use Cases**
>
> Describe the main use cases or user stories. For each one, provide:
> - A short title (e.g. "User logs in", "Driver submits delivery report")
> - The actor (who triggers it)
> - The expected outcome
>
> You can list them as plain text, bullet points, or numbered steps.
> Type **skip** to omit this section.

Format the answer into a structured **Use Cases** section with a table:

| # | Use Case | Actor | Trigger | Expected Outcome |
|---|----------|-------|---------|-----------------|

---

## Phase 3 — Sequence Diagrams

Ask the user:

> **Phase 3/5 — Sequence Diagrams**
>
> For the most important flows (e.g. login, main data flow, critical API call), describe the
> sequence of interactions between components, layers, or services.
>
> You can describe them in plain language (e.g. "App calls API → API queries DB → returns JSON
> to ViewModel → updates UI"), and I will convert them to Mermaid sequence diagrams.
>
> Or paste existing Mermaid/PlantUML diagrams directly.
> Type **skip** to omit this section.

Convert each described flow into a Mermaid `sequenceDiagram` block. Generate as many diagrams
as there are distinct flows. If the user described the flow in plain language, infer the
participants from the project structure found in Phase 0.

Example output format:
```mermaid
sequenceDiagram
    participant User
    participant App
    participant API
    participant DB
    User->>App: Triggers action
    App->>API: HTTP request
    API->>DB: Query
    DB-->>API: Result
    API-->>App: JSON response
    App-->>User: Updated UI
```

---

## Phase 4 — Focused Code Details

Ask the user:

> **Phase 4/5 — Focused Code Details**
>
> Is there a specific part of the codebase you want explained in depth? This could be:
> - A tricky algorithm or business rule
> - A non-obvious design pattern
> - A specific class, module, or file
> - A known limitation or workaround
>
> You can paste file paths, paste code snippets, or describe the area in words.
> You do NOT need to share the full codebase — focus only on what matters.
> Type **skip** to omit this section.

For each item the user provides:
- Read the file if a path is given.
- Write a clear explanation of what the code does, why it exists, and any gotchas.
- Include inline code references with file path and line numbers where relevant.

---

## Phase 5 — Setup & Contribution Guide

Ask the user:

> **Phase 5/5 — Setup & Contribution**
>
> Provide any information needed to get the project running locally:
> - Prerequisites (SDK version, environment variables, tools)
> - Build / run commands
> - How to run tests
> - Branching strategy or contribution rules
> - Links to CI/CD, staging, or production environments
>
> Type **skip** to skip (I will infer what I can from build files).

Merge with what was found in Phase 0 (Dockerfile, Makefile, package.json scripts, etc.).

---

## Final Assembly — Write documentation.md

Once all phases are complete, produce the file `documentation.md` in the **current working
directory**. The document must follow this structure:

```
# [Project Name]

> One-line description of what the project does.

---

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [Architecture](#2-architecture)
3. [Use Cases](#3-use-cases)
4. [Key Flows — Sequence Diagrams](#4-key-flows--sequence-diagrams)
5. [Focused Code Details](#5-focused-code-details)
6. [Setup & Getting Started](#6-setup--getting-started)
7. [Contribution Guide](#7-contribution-guide)
8. [Known Limitations & Gotchas](#8-known-limitations--gotchas)

---

## 1. Project Overview
[Tech stack, purpose, architecture style, key dependencies]

## 2. Architecture
[Module graph (ASCII or description), layering, data flow, entry points]

## 3. Use Cases
[Table from Phase 2]

## 4. Key Flows — Sequence Diagrams
[Mermaid diagrams from Phase 3]

## 5. Focused Code Details
[Deep-dives from Phase 4]

## 6. Setup & Getting Started
[From Phase 5 + Phase 0 discovery]

## 7. Contribution Guide
[Branching, PR rules, test commands, code style]

## 8. Known Limitations & Gotchas
[Anything flagged by the user or inferred from TODO/FIXME in the code]
```

### Rules for the final document:
- Write in **English** unless the user wrote their answers in another language, in which case
  match that language throughout.
- Every section that was skipped must still appear with a short note:
  `> *This section was not provided. Contribute by opening a PR against this file.*`
- All code references must include file path and line number.
- All Mermaid diagrams must be enclosed in triple-backtick mermaid fences.
- The document must be self-contained: no broken links, no placeholders left unfilled.
- After writing the file, print a short summary to the user:
  - Path of the generated file
  - List of sections that were filled vs skipped
  - Any ambiguities or missing information worth revisiting
