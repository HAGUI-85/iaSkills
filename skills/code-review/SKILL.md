---
name: code-review
description: >
  Performs a deep, structured code review of a file, module, or entire project.
  Covers architecture, code quality, security, exception handling, design patterns,
  performance, tests, dependencies, and produces a prioritized pros/cons report.
  TRIGGER when the user asks to: review code, audit a module, analyse architecture,
  check security, find bugs, or study patterns in any codebase.
user-invocable: true
argument-hint: "[file | module-path | . (whole project)]"
---

# Code Review Skill

You are performing a **deep, structured code review**.

**Scope:** `$ARGUMENTS`

- If `$ARGUMENTS` is empty or `.` → review the **entire project** from the current working directory.
- If `$ARGUMENTS` is a directory path (e.g. `core/logger`) → scope the review to that directory.
- If `$ARGUMENTS` is a file path → review that single file in depth.

---

## Ground rules before starting

1. **Read everything before writing anything.** Do not produce any review output until Step 0 is fully complete.
2. **Every finding must be anchored** to a real file and line number you actually read. Never invent findings.
3. **No generic advice.** Every sentence in the review must reference actual code from this project.
4. **Severity labels** to use on every finding: `CRITICAL` · `HIGH` · `MEDIUM` · `LOW`
5. **Output** is a single markdown file written at the end (see Step 10).

---

## Step 0 — Exploration & Cartography (READ EVERYTHING FIRST)

Use `list_directory` and `read_file` extensively. Do not skip files.

```
1. List the root directory to understand the project structure.
2. Find and read all build/config files:
   - Android: settings.gradle.kts, all build.gradle.kts, gradle/libs.versions.toml
   - Node: package.json, tsconfig.json
   - Python: pyproject.toml, setup.py, requirements.txt
   - Go: go.mod, go.sum
3. Find all source files in scope:
   - Use shell: find <scope> -name "*.kt" -o -name "*.java" -o -name "*.ts" -o -name "*.py" -o -name "*.go"
   - Exclude: build/, .git/, node_modules/, .gradle/, __pycache__/
4. Read EVERY source file found. Read them all before writing the review.
5. Build a mental model of:
   - Module/package graph (who depends on whom)
   - Layer structure (presentation / domain / data / infrastructure)
   - Entry points (Application class, main(), index, etc.)
   - Key abstractions (interfaces, base classes, sealed classes)
   - External dependencies (what SDKs/libs are used)
```

Only begin writing the review once ALL files are read.

---

## Step 1 — Architecture

Write section `## 1. Architecture`

### 1.1 Pattern & Paradigm
- Identify the architectural pattern: Clean Architecture, MVC, MVVM, Hexagonal, Layered, etc.
- Is the pattern applied consistently? Where does it break down? Quote the file.
- Draw a dependency diagram in ASCII showing module/layer relationships and direction of arrows.

### 1.2 Module Structure
For each module/package, state its **declared responsibility** and whether it **respects it**.
Flag:
- God modules (too many responsibilities)
- Modules split unnecessarily (could be merged)
- Circular dependencies
- Missing modules that should exist (e.g., `:core:model` when DTOs serve as domain models)

### 1.3 Dependency Flow
- Do outer layers (presentation, data) depend inward (domain)? Never the reverse?
- Is Android/Spring/framework code leaking into the domain layer?
- Are there dependencies on concrete implementations instead of abstractions?

### 1.4 Architecture Verdict
`Excellent` / `Good` / `Acceptable` / `Needs Refactoring` / `Critical` — 2–3 sentence justification.

---

## Step 2 — Code Quality

Write section `## 2. Code Quality`

### 2.1 Naming & Conventions
- Classes: reflect their responsibility? Consistent language (no EN/FR mix)?
- Methods: verb-object form? Consistent casing?
- Variables: descriptive vs. abbreviated?
- Constants: SCREAMING_SNAKE_CASE vs. camelCase used consistently?
- Flag every inconsistency with file:line.

### 2.2 SOLID Principles
Check each — flag violations with **file:line**:
| Principle | Violation found? | File:Line |
|-----------|-----------------|-----------|
| S — Single Responsibility | | |
| O — Open/Closed | | |
| L — Liskov Substitution | | |
| I — Interface Segregation | | |
| D — Dependency Inversion | | |

### 2.3 DRY — Duplication
- Copy-pasted logic blocks (quote both occurrences with file paths)
- Duplicated constants or string literals (list them)
- Parallel class hierarchies doing the same thing

### 2.4 Dead Code
- Unused imports
- Unused functions / classes / variables
- Commented-out code blocks left in production files
- `TODO` / `FIXME` comments indicating unfinished work (list each with file:line)

### 2.5 Complexity
Flag every:
- Function longer than 40 lines
- Nesting deeper than 3 levels (if/when/try inside each other)
- Function with more than 5 parameters
- Class with more than 500 lines or 10+ public methods

### 2.6 Language Idioms
Is the language used in its modern idiomatic form?
- Kotlin: data classes, sealed classes, extension functions, Flow, coroutines, `?:` instead of null checks?
- Java patterns used where Kotlin idioms exist (getters/setters, callbacks instead of coroutines)?
- TypeScript: generics, nullish coalescing, optional chaining?
- Python: dataclasses, type hints, context managers?

---

## Step 3 — Security

Write section `## 3. Security`

For each finding use this format:
> **[SEVERITY]** `file:line` — Description — Recommended fix

### 3.1 Secrets & Credentials
- Hardcoded API keys, passwords, tokens, HMAC secret keys in source code
- Placeholder values never replaced (e.g. `"yourSecretKey"`, `"TODO: real key"`)
- Sensitive values in `BuildConfig`, `.env` files committed to git
- Secret material in logs or stack traces

### 3.2 Data Storage
- Passwords, tokens, PII stored unencrypted in SharedPreferences / UserDefaults / localStorage / flat files
- Database fields with sensitive data without encryption at rest
- Sensitive values in `onSaveInstanceState` or `Intent` extras without protection

### 3.3 Network
- HTTP (not HTTPS) URLs in code or config
- SSL/TLS validation disabled (`setHostnameVerifier`, `trustAllCerts`, etc.)
- Sensitive data in URL query parameters (visible in logs and server access logs)
- `isAllowInsecureProtocol = true` in Gradle/Maven/npm config

### 3.4 Injection & Input Validation
- User input passed directly to SQL (SQL injection)
- User input in shell commands (command injection)
- Unescaped output in HTML/templates (XSS)
- Deserialization of untrusted data without schema validation

### 3.5 Authentication & Authorization
- Missing auth checks on sensitive endpoints / screens
- Weak or predictable token generation
- Tokens with no expiry or overly long expiry
- Session fixation or insufficient session invalidation

### 3.6 Platform-Specific (Android / iOS / Web)

**Android:**
- `android:allowBackup="true"` with sensitive SharedPreferences
- Exported components (`android:exported="true"`) without required permissions
- `@SuppressLint(...)` annotations suppressing security warnings — list every one
- Private API access via reflection (`@SuppressLint("PrivateApi")`)
- World-readable files or content providers without permission

### 3.7 Supply Chain
- SNAPSHOT / pre-release versions in production dependencies
- `isAllowInsecureProtocol` on package repositories
- Unpinned/floating versions (`+`, `latest`, `*`)

---

## Step 4 — Exception & Error Handling

Write section `## 4. Exception Handling`

### 4.1 Swallowed Exceptions
Find every occurrence of:
```
catch (e) { }                        // empty catch
catch (e) { e.printStackTrace() }    // console only, not crash reporter
} catch (_) { }                      // ignored
```
List each with file:line.

### 4.2 Force-Unwrap & Null Crashes
- Kotlin `!!` operator — each occurrence is a potential NPE (list all with file:line)
- Java `.get()` on Optional without `.isPresent()` check
- Array/list access without bounds checking
- Direct cast without safe-cast (`as T` instead of `as? T`)

### 4.3 Error Propagation
- Functions that can fail but return `void`/`Unit` with no error signal
- Exceptions swallowed and silently converted to `null` return
- Missing `Result<T>`, `Either<L,R>`, or `sealed class` on functions that can fail
- Silent failures not surfaced to the UI (no error state, no toast, no log)

### 4.4 Resource Leaks
- Streams, connections, cursors not closed in `finally` / `use {}` / `try-with-resources`
- Coroutine scopes / Jobs not cancelled when lifecycle ends
- BroadcastReceivers registered but never unregistered
- Observers / listeners added but never removed

### 4.5 Threading & Blocking
- `runBlocking` on the main/UI thread
- Network or disk I/O directly on the main thread
- `Thread.sleep()` in production code
- Unprotected shared mutable state accessed from multiple threads

---

## Step 5 — Design Patterns

Write section `## 5. Design Patterns`

### 5.1 Patterns Correctly Applied
For each pattern found, state: file path · pattern name · quality assessment (1 line).
Examples: Repository, Factory, Builder, Observer, Strategy, Decorator, Adapter, Facade, Command, State.

### 5.2 Patterns Incorrectly Applied or Violated
- Interface/class name implies a pattern but implementation is wrong
- Pattern declared (e.g. `LogStrategy` interface) but never implemented or used — dead code
- Anti-patterns present: God Class, Singleton abuse, Service Locator, Magic Number, Feature Envy

### 5.3 Missing Patterns (Opportunities)
For each suggestion, explain: current pain point → pattern that would solve it → where to apply it.
Examples:
- Multiple `if (buildType == "X")` checks scattered → **Strategy Pattern**
- 5 overloads of the same function → **Builder Pattern**
- Direct coupling to external vendor SDK → **Adapter / Facade Pattern**
- Parallel Java + Kotlin implementations of same concept → **Bridge Pattern to unify**

---

## Step 6 — Performance

Write section `## 6. Performance`

For each finding: **[SEVERITY]** `file:line` — description — fix.

- Blocking call on main/UI thread (network, disk, `runBlocking`)
- N+1 queries: loop that triggers a DB or network call per iteration
- Large object creation in hot paths (inside loops, `onDraw()`, `onBindViewHolder()`)
- Missing pagination on list endpoints
- Repeated expensive computation without caching or `lazy`
- Linear search (`firstOrNull`, `find`, `filter { }.first()`) on a structure that could be a `Map`
- Unnecessary recomposition in Compose / re-render in React
- Images loaded without size constraints (OOM risk)
- Memory leaks: static references to Activity/Context, uncancelled coroutines, unremoved listeners

---

## Step 7 — Test Coverage

Write section `## 7. Tests`

### 7.1 Coverage Inventory
List all test files found. Classify each:
`Unit` | `Integration` | `Instrumented` | `E2E` | `Boilerplate (no real assertion)`

### 7.2 Critical Code Without Tests
Identify the most important untested paths:
- Business logic (use cases, domain services, state machines)
- Repository / data layer
- Serialization / deserialization
- Security-critical functions (token generation, encryption, validation)
- Error handling paths (what happens when the network fails? When the DB is empty?)
- Edge cases (empty collections, null values, max values, concurrent access)

### 7.3 Test Quality Issues
- Tests with no assertions (pass trivially)
- Tests that test implementation details instead of observable behavior
- Mocks that never verify their calls were made
- Missing `@BeforeEach` / `setUp` causing test pollution between runs

---

## Step 8 — Dependencies & Build Configuration

Write section `## 8. Dependencies`

### 8.1 Outdated or Abandoned Libraries
For each dependency in scope, flag if:
- Last release > 2 years ago (abandoned)
- A better modern alternative exists
- It's declared globally but used only in one module

### 8.2 Duplicate Responsibilities
List sets of libraries that do the same thing (e.g., 3 serialization libs, 2 HTTP clients).
For each set: recommend which to keep and the migration path.

### 8.3 Version Risks
- SNAPSHOT versions in production builds (not reproducible)
- Floating / unpinned versions (`+`, `latest`, `*`)
- Mismatched companion versions (e.g. KSP version ≠ Kotlin version prefix)
- BOM used inconsistently (some deps pinned, others use BOM)

### 8.4 Build Configuration
- Minification / R8 / tree-shaking disabled in release builds
- Build features enabled but unused (adds compile time)
- Dependencies declared twice (once explicitly, once injected by a convention plugin)
- Lint / static analysis disabled (`abortOnError = false`, `checkReleaseBuilds = false`)

---

## Step 9 — Bilan (Pros & Cons)

Write section `## 9. Bilan`

### ✅ What is well done
List 5–10 genuine strengths. Be specific — reference real files/patterns found.
Format: `- **Pattern/practice name** (file or module): why it's good`

### ⚠️ Prioritized Findings

**CRITICAL — Fix before next release:**
| # | Finding | File:Line | Impact |
|---|---------|-----------|--------|

**HIGH — Fix within current sprint:**
| # | Finding | File:Line | Impact |
|---|---------|-----------|--------|

**MEDIUM — Plan for next 2 sprints:**
| # | Finding | File:Line | Impact |
|---|---------|-----------|--------|

**LOW — Backlog:**
| # | Finding | File:Line | Impact |
|---|---------|-----------|--------|

### Recommended Action Plan
Exactly 3 concrete steps. No vague advice.

**Step 1 — Immediate (this week):**
`file or module` → specific change → measurable outcome

**Step 2 — Short-term (1–2 sprints):**
specific refactoring → benefit to the codebase

**Step 3 — Long-term (architectural):**
structural change → why it matters for the project's future

---

## Step 10 — Generate the report file

After completing all 9 sections:

1. Determine output path:
   - Whole project (`$ARGUMENTS` is `.` or empty) → write `review.md` at project root
   - Module path (e.g. `core/logger`) → write `core/logger/review.md`
   - Single file (e.g. `app/src/Foo.kt`) → write `app/src/Foo.review.md` next to the file

2. Write the complete report using `write_file`. The report must include all 9 sections.

3. Print a summary to the terminal:
   ```
   ✅ Review complete → <output path>

   Findings:
     CRITICAL : N
     HIGH     : N
     MEDIUM   : N
     LOW      : N

   Top 3 urgent:
   1. [CRITICAL] file:line — one-sentence description
   2. [CRITICAL] file:line — one-sentence description
   3. [HIGH]     file:line — one-sentence description
   ```

---

## Usage examples

```bash
# Review the entire project (run from project root)
gemini /code-review .

# Review a specific module
gemini /code-review core/logger

# Review a single file
gemini /code-review app/src/main/java/com/acme/TokenGenerator.kt

# Non-interactive (headless) — write report and exit
gemini --yolo -p "/code-review ."

# Resume a previous review session
gemini --resume latest /code-review core/network
```
