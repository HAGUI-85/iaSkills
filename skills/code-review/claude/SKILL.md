---
name: code-review
description: >
  Performs a deep, structured code review of a file, module, or entire project.
  Covers architecture, code quality, security, exception handling, patterns,
  improvement axes, and a pros/cons summary. Produces a detailed markdown report.
  TRIGGER when the user asks to: review code, audit a module, analyse architecture,
  check security, or study patterns in any codebase.
user-invocable: true
argument-hint: "[file | module | . (whole project)]"
allowed-tools: Read, Glob, Grep, Bash, Agent, Write
---

# Code Review Skill

You are performing a **deep, structured code review**. The scope is: `$ARGUMENTS`

If `$ARGUMENTS` is empty or `.`, review the **entire project** starting from the working directory.
If `$ARGUMENTS` is a module path (e.g. `core/logger`), scope the review to that module.
If `$ARGUMENTS` is a file path, review that single file in depth.

---

## Step 0 — Exploration & Cartography

Before writing anything, **read and map the codebase**. Do not skip this step.

1. List all source files matching the scope using Glob (`**/*.kt`, `**/*.java`, `**/*.ts`, `**/*.py`, etc.)
2. Read `settings.gradle.kts` / `package.json` / `pyproject.toml` / equivalent to understand the module graph
3. Read all `build.gradle.kts` / `build.gradle` / equivalent build files in scope
4. Read every source file in scope — use parallel Read calls where possible
5. Build a mental model of:
   - Module graph (who depends on whom)
   - Layer structure (presentation / domain / data / infra)
   - Entry points (Application class, main(), index, etc.)
   - Key abstractions (interfaces, base classes, protocols)

Only begin writing the review once you have read **all** relevant files.

---

## Step 1 — Architecture Analysis

Write section **## 1. Architecture** covering:

### 1.1 Paradigm & Pattern
- What architectural pattern is used? (Clean Architecture, MVC, MVVM, Hexagonal, etc.)
- Is the pattern applied consistently? Where does it break down?
- Draw an ASCII dependency diagram showing module/layer relationships

### 1.2 Module Structure
- List all modules/packages with their stated responsibility
- Identify modules that do **too much** (God module) or **too little** (unnecessary split)
- Check for circular dependencies between modules
- Identify missing modules that should exist (e.g. a `:core:model` when DTOs are used as domain models)

### 1.3 Dependency Flow
- Does data flow respect the Clean Architecture rule (outer layers depend inward, never the reverse)?
- Are there any upward dependencies (domain knowing about data layer, etc.)?
- Are framework concerns (Android, Spring, etc.) leaking into domain/business logic?

### 1.4 Verdict
Rate: `Excellent` / `Good` / `Acceptable` / `Needs Refactoring` / `Critical`
Explain in 2–3 sentences.

---

## Step 2 — Code Quality Analysis

Write section **## 2. Code Quality** covering:

### 2.1 Naming & Conventions
- Class names: do they reflect their responsibility?
- Method names: verb-object? Consistent language (EN/FR mixed?)?
- Variable names: descriptive vs abbreviated vs Hungarian notation?
- Constants: SCREAMING_SNAKE_CASE for constants, camelCase for vals?
- Flag inconsistencies: mixed casing, mixed languages, abbreviations

### 2.2 SOLID Principles
Check each principle and flag violations with file + line number:
- **S** — Single Responsibility: does each class/function have ONE reason to change?
- **O** — Open/Closed: are extensions made by adding code, not modifying existing?
- **L** — Liskov Substitution: do subclasses respect contracts of base types?
- **I** — Interface Segregation: are interfaces small and focused?
- **D** — Dependency Inversion: do modules depend on abstractions, not concretions?

### 2.3 DRY & Duplication
- Find copy-pasted code blocks (same logic in 2+ places)
- Find duplicated constants/strings
- Find parallel class hierarchies doing the same thing
- Note: quote the duplicated code with file paths

### 2.4 Dead Code
- Unused imports
- Unused functions/classes/variables
- Commented-out code left in production files
- `TODO` / `FIXME` comments that indicate unfinished work

### 2.5 Complexity
- Functions longer than 40 lines (flag them)
- Nested conditionals deeper than 3 levels
- Functions with more than 5 parameters
- Classes with more than 10 methods or 500 lines

### 2.6 Language Idioms
- Is the language used idiomatically? (Kotlin: data classes, sealed classes, extension functions, Flow vs LiveData)
- Are legacy patterns from the old language used instead of idiomatic equivalents? (Java getters/setters in Kotlin, callbacks instead of coroutines, etc.)

---

## Step 3 — Security Analysis

Write section **## 3. Security** covering each finding with: **severity** (`CRITICAL` / `HIGH` / `MEDIUM` / `LOW`), **file:line**, **description**, and **fix**.

### 3.1 Secrets & Credentials
- Hardcoded API keys, passwords, tokens, secret keys in source code
- Placeholder values that should be replaced (e.g. `"yourSecretKey"`, `"TODO: replace"`)
- Credentials committed to the repository
- Sensitive data in `BuildConfig`, `gradle.properties`, or config files tracked by git

### 3.2 Data Storage
- Sensitive data (passwords, tokens, PII) stored unencrypted in SharedPreferences / UserDefaults / localStorage / flat files
- Database fields storing sensitive data without encryption
- Missing encryption at rest

### 3.3 Network
- HTTP URLs (not HTTPS) used for API calls
- SSL/TLS certificate validation disabled or bypassed
- Sensitive data transmitted in URL query parameters (visible in logs)
- Missing certificate pinning for sensitive endpoints
- `isAllowInsecureProtocol` or equivalent enabled in build tools

### 3.4 Input Validation & Injection
- User input passed directly to SQL queries (SQL injection)
- User input passed to shell commands (command injection)
- User input reflected in HTML without escaping (XSS)
- Deserialization of untrusted data without validation

### 3.5 Authentication & Authorization
- Missing authentication checks on sensitive endpoints/screens
- Insufficient authorization (any logged-in user can access admin resources)
- Weak token generation (predictable, short-lived not enforced, hardcoded expiry strings)
- Session management issues

### 3.6 Android/Mobile Specific (if applicable)
- `android:allowBackup="true"` exposing sensitive data
- Exported components (`android:exported="true"`) without permission checks
- `@SuppressLint` annotations hiding security warnings (list each one)
- Private API access via reflection (`@SuppressLint("PrivateApi")`)
- World-readable/writable files

### 3.7 Supply Chain
- `isAllowInsecureProtocol` on Maven/npm/pip repositories
- Unpinned dependency versions (floating `+` or `latest`)
- SNAPSHOT / pre-release versions in production dependencies

---

## Step 4 — Exception & Error Handling Analysis

Write section **## 4. Exception Handling** covering:

### 4.1 Swallowed Exceptions
Find all patterns where exceptions are caught but not properly handled:
```
// Anti-patterns to find:
catch (e: Exception) { }                    // empty catch
catch (e: Exception) { e.printStackTrace() } // logged to console only, not Crashlytics
} catch (_: Exception) { }                  // ignored with underscore
```
List each occurrence with file:line.

### 4.2 Force-Unwrap / Null Crashes
- `!!` operator in Kotlin (each occurrence = potential NPE)
- `!` non-null assertion without guard in Swift
- `.get()` on Optional without `.isPresent()` check in Java
- Array access without bounds checking

### 4.3 Error Propagation
- Functions that can fail but return `void`/`Unit` (caller cannot know if it failed)
- Checked exceptions swallowed and converted to `null` return
- Missing `Result<T>` / `Either<L,R>` / `Option<T>` on functions that can fail
- Error states not communicated to the UI (silent failures)

### 4.4 Resource Leaks
- Streams, connections, cursors opened but not closed in `finally` / `use {}` blocks
- Coroutine scopes / jobs not cancelled when lifecycle ends
- BroadcastReceivers registered but never unregistered
- Observers added but never removed

### 4.5 Crash-Prone Patterns
- `runBlocking` on the main thread
- Network or disk I/O on the main thread
- Division without zero-check
- Integer overflow in calculations

---

## Step 5 — Design Patterns Analysis

Write section **## 5. Design Patterns** covering:

### 5.1 Patterns Correctly Applied
List patterns that are well-implemented with brief explanation:
- Repository Pattern, Factory, Builder, Observer, Strategy, Decorator, etc.
- For each: file path, pattern name, quality assessment

### 5.2 Patterns Incorrectly Applied or Violated
- Pattern declared (interface name / class name implies a pattern) but not implemented correctly
- Dead pattern code (interface defined, never implemented)
- Anti-patterns: God Class, Singleton abuse, Service Locator replacing DI, etc.

### 5.3 Missing Patterns
- Where a pattern would significantly simplify the code
- Example: multiple `if (buildType == "X")` checks → Strategy pattern
- Example: 5 overloads of the same function → Builder pattern
- Example: direct coupling to external SDK → Adapter/Facade pattern

---

## Step 6 — Performance Analysis

Write section **## 6. Performance** covering findings with **severity** and **file:line**:

- Blocking calls on the main/UI thread
- N+1 queries (loop that makes a DB or network call per iteration)
- Large objects created in hot paths (inside loops, `onDraw()`, `onBind()`)
- Missing pagination on list endpoints
- Missing caching for expensive operations
- Linear search (`firstOrNull`, `find`) on collections that could use a Map
- Unnecessary recomposition in Compose / re-render in React
- Images loaded without size constraints (OOM risk)
- Memory leaks (static references to Activity/Context, uncancelled coroutines)

---

## Step 7 — Test Coverage Analysis

Write section **## 7. Tests** covering:

### 7.1 Coverage Inventory
List all test files found and classify them (Unit / Integration / Instrumented / E2E / Boilerplate).

### 7.2 What Is NOT Tested
Identify the most critical untested code:
- Business logic (use cases, domain services)
- Repository layer
- Serialization / deserialization
- Security-critical functions (token generation, encryption)
- Error handling paths

### 7.3 Test Quality
- Tests with no assertions (vacuous tests)
- Tests that test implementation details instead of behavior
- Missing edge cases (empty list, null, max value, concurrent access)

---

## Step 8 — Dependencies & Build Configuration

Write section **## 8. Dependencies** covering:

### 8.1 Outdated or Abandoned Libraries
For each dependency in scope:
- Is it maintained? Last release > 2 years ago = flag it
- Is there a better, more modern alternative?
- Is it used only in one module but declared globally?

### 8.2 Duplicate Responsibilities
- Multiple libraries doing the same thing (e.g., 3 serialization libraries, 2 HTTP clients)
- Recommend which to keep and migration path

### 8.3 Version Risks
- SNAPSHOT versions in production
- `+` / `latest` / unpinned versions
- Mismatched companion versions (e.g., KSP version must match Kotlin version)

### 8.4 Build Configuration
- Minification / R8 disabled in release
- Build features enabled but unused (adds compile time)
- Duplicate dependency declarations (already provided by convention plugin)
- `abortOnError = false` / `lint` warnings suppressed

---

## Step 9 — Pros & Cons Summary

Write section **## 9. Bilan (Pros & Cons)** as a structured executive summary:

### What is well done ✅
List 5–10 genuine strengths with brief justification. Be specific — not generic praise.
Format: `- **Thing**: why it's good`

### What needs improvement ⚠️
Prioritized list of all findings, grouped by severity:

**CRITICAL — Fix before next release:**
| # | Finding | File | Impact |
|---|---------|------|--------|

**HIGH — Fix within current sprint:**
| # | Finding | File | Impact |

**MEDIUM — Plan for next 2 sprints:**
| # | Finding | File | Impact |

**LOW — Backlog:**
| # | Finding | File | Impact |

### Recommended Action Plan
Three concrete next steps (not vague recommendations):
1. **Immediate (this week):** specific action + specific file + expected outcome
2. **Short-term (1–2 sprints):** specific refactoring + benefit
3. **Long-term (architectural):** structural change + benefit

---

## Step 10 — Generate the report file

After completing all sections:

1. Determine the report filename:
   - Whole project → `review.md` in project root
   - Single module (e.g. `core/logger`) → `core/logger/review.md`
   - Single file (e.g. `app/src/.../Foo.kt`) → same directory as the file, named `Foo.review.md`

2. Write the complete report using the Write tool.

3. Confirm to the user:
   - Report file path
   - Total findings count by severity
   - Top 3 most urgent findings in one sentence each

---

## Formatting rules

- Use `**file/path:line**` format for all code references
- Use code blocks for all code snippets (with language tag)
- Use tables for lists of findings
- Every finding must have: severity label, location, description, recommended fix
- Severity labels: `CRITICAL` `HIGH` `MEDIUM` `LOW`
- Do NOT invent findings — only report what you actually observed in the code
- Do NOT pad with generic advice — every sentence must reference actual code from the project
