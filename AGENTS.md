# Contributor & CI Guide  <!-- AGENTS.md v1.4 -->

> **Read this file first** before opening a pull request.  
> This universal, project‑agnostic template aligns humans, LLM coding agents,
> and CI. Its goals are **future extensibility**, **error‑preventive design**,
> **graceful degradation**, and **fault isolation** (if one module fails, others
> keep working when feasible). If you change *any* rule below, **bump the
> version number in this heading**.

---

## 0 · Authority & Single Source of Truth (SSOT)

- **Product SSOT:** `/docs/spec/**` (requirements, domain language, acceptance).
- **Process SSOT:** `AGENTS.md` (this file).
- **Work plan:** `TODO.md` (queue of next changes).
- **Decision log:** `NOTES.md` (append‑only record of decisions/outcomes).

**Precedence when in conflict:** Product SSOT → `AGENTS.md` → `TODO.md`
→ `NOTES.md`. If something is **unspecified**, choose the **simplest, safest**
option, then **append** the decision/assumption to `NOTES.md` and, if work is
needed, add a line to `TODO.md`.

Maintain the project so that **after each feature** a user can clone the repo
and run it locally to test manually (no hidden infra).

---

## 1 · Modularity / High Cohesion / Low Coupling

Design as a **modular monolith** of cohesive feature modules aligned to business
capabilities (bounded contexts). For each module, define its purpose and a
**minimal public API (ports)** first; keep domain logic **framework‑free**
behind interfaces (**information hiding**, SOLID—ISP/DIP) and implement
I/O/framework concerns as replaceable **adapters** (hexagonal), composed via
**explicit dependency injection**. Keep dependencies **acyclic** and directed
**inward** to stable abstractions; **package by feature, not by layer**. From
high‑level specs, first output a **module map** and **interface contracts** to
`/docs/architecture/module-map.md`, then implement the **smallest end‑to‑end
vertical slice** to validate the design. **Enforce boundaries** with
architecture tests and CI fitness functions (e.g., no cycles; domain has zero
framework/adapter imports) and **track cohesion/coupling trends** (e.g.,
LCOM/CBO, dependency DAG). Prefer small interfaces and units with a **single
reason to change**; document each module’s API and invariants; extract services
**only when operational needs justify it**.

---

## 2 · Discovery & Alignment (read before you code)

Before starting any task, the contributor/agent **must**:

1. **Read** all relevant Markdown in the repo: `README.md`, `AGENTS.md`,
   `TODO.md`, `NOTES.md`, and `/docs/**` for the affected area.
2. **Scan the code** to locate the impacted modules/packages and their public
   APIs; note invariants and dependencies.
3. **Record an intent**: append a brief “Plan” entry to `NOTES.md` describing
   scope, touched modules, expected API changes, tests to add, and risks.
4. **Update the plan** in `TODO.md` with concrete steps; link to the `NOTES.md`
   plan line.

This ensures new work aligns with current design and conventions.

---

## 3 · Extensibility & Compatibility Policy

- **Stable APIs:** Minimize public surface; prefer additive, backward‑compatible
  changes. Use semantic versioning for published packages.
- **Configurability:** Feature‑flag non‑critical behavior; default to **off**
  and safe. Keep flags in `config/flags.*` (or project equivalent).
- **Schema evolution:** Prefer additive migrations; never break consumers
  without a documented migration path.
- **Pluggability:** Use ports/adapters; avoid hardwired singletons/global state.
- **Contracts:** Document preconditions, postconditions, invariants, and error
  codes for every public API.

---

## 4 · Error Handling & Fault Isolation (graceful degradation)

- **Localize failures:** Each module is an **error boundary**. Catch, classify,
  and **translate** exceptions to module‑specific error types; never leak
  internal stack traces across module boundaries.
- **Fail fast at edges, degrade inside:** Validate inputs at boundaries; where
  possible, return defaults/cached/partial results instead of failing the
  entire request.
- **Resilience patterns:** Use **timeouts**, **bounded retries with jitter**,
  **idempotency** for retried operations, and **circuit breakers** for
  persistent faults. Isolate resources (**bulkheads**) so one failure does not
  exhaust threads/connections globally.
- **Side‑effect safety:** Make external calls idempotent or guard with
  deduplication tokens; wrap multi‑step changes in transactional or
  compensating logic.
- **Clear error contracts:** Every public call documents what errors can happen
  and how callers should react (retry, fallback, user message).
- **Resource hygiene:** Always close/cleanup (files, sockets, DB cursors) and
  cap concurrency to avoid overload.

---

## 5 · Observability & Diagnostics

- **Structured logs:** Include trace/request IDs; log at INFO/WARN/ERROR with
  actionable context; never log secrets.
- **Metrics & health:** Expose basic counters, latencies, and failure rates per
  module. Provide lightweight health/readiness checks per module.
- **Tracing:** If available, propagate trace context (e.g., OpenTelemetry).
- **Repro steps:** When a TODO is completed or an incident occurs, append the
  “why/impact” to `NOTES.md` (with links).

---

## 6 · File Ownership & Merge‑Conflict Safety

| Rule | Detail |
|------|--------|
| **Distinct‑files** | Concurrent tasks **must** edit distinct non‑Markdown files. |
| **Append‑only logs** | `TODO.md` and `NOTES.md` are linear—**never delete or reorder**; append at end. |
| **Generated files** | `generated/**`, `openapi/**`, `schemas/**`, `dist/**` are **code‑generated**; never hand‑edit. |
| **Shared MD** | Anyone may **append** (never rewrite) `AGENTS.md`, `TODO.md`, `NOTES.md`. |

**Conflict markers check (all changes):**
- Local pre‑commit or before commit:  
  `git grep -n -E '<{7}|={7}|>{7}' -- . || true`  (should output nothing)
- CI also fails if any conflict markers remain (see § 11).

Consider `CODEOWNERS` for critical paths and an `.editorconfig` to normalize
whitespace and line endings.

---

## 7 · Bootstrap (first‑run) Checklist

1. Run `./.codex/setup.sh` (or `./setup.sh`) **if present** after cloning and
   when dependencies change:
   ```bash
   if [ -x ./.codex/setup.sh ]; then ./.codex/setup.sh; fi
Add required secrets in repository/organization settings (e.g.,
GIT_TOKEN, GH_PAGES_TOKEN, others as needed).

Ensure forks without secrets still pass using the CI secret check (see § 11).

Update README badges to point to your fork (owner/repo).

Human‑operated settings (agent cannot automate): repository secrets,
branch protection, enabling Pages, Code Scanning/CodeQL.

8 · Contributor Workflow (for humans & agents)
Branch/PR flow: fork → feat/<topic> → PR into main (≥1 reviewer).

Pre‑commit commands (also run by CI):

bash
Copy code
# Run if present; otherwise provide equivalent commands in the PR.
test -f Makefile && make lint || echo "No 'make lint' defined"
test -f Makefile && make test || echo "No 'make test' defined"
Style: auto‑format (e.g., black, prettier, dart format, gofmt);
Markdown lines ≤ 80 chars; exactly one blank line between log entries.

Exit codes: scripts must exit non‑zero on failure so CI catches issues.

Version pinning: pin major/minor for critical runtimes/actions (e.g.,
actions/checkout@v4, node@20, python~=3.11).

Docs consistency: when docs change, update them everywhere; if ambiguity
remains, Product SSOT wins.

Log discipline: when a TODO is completed, append the matching NOTES
entry in the same PR.

9 · Pre‑commit Hooks (required when available)
If .pre-commit-config.yaml exists, contributors must run:

bash
Copy code
pipx install pre-commit || pip install pre-commit
pre-commit install
pre-commit run --all-files
If a Node project uses Husky, install hooks with:

bash
Copy code
npm run prepare
Minimal recommended .pre-commit-config.yaml (language‑agnostic hygiene):

yaml
Copy code
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: mixed-line-ending
      - id: check-yaml
      - id: check-merge-conflict
CI mirrors these checks so even if hooks aren’t installed locally, the same
gates run server‑side (§ 11).

10 · Testing Policy & Quality Gates
Tests are mandatory for every new/changed element:

Unit tests for domain logic (pure, fast, isolated).

Integration tests for adapters and cross‑module seams.

Architecture tests to enforce boundaries (e.g., no cycles, hexagonal rules)
when a framework exists for the language.

Scenario/acceptance tests for each vertical slice added.

Placement: follow language conventions (e.g., */test/**, */src/test/**).

Determinism: tests must be repeatable, hermetic where possible; avoid
network and clock dependence (use fakes and time abstractions).

Coverage (if tooling present): generate a coverage artifact
(e.g., coverage.xml, lcov.info, coverage/); set an optional threshold via
project config. Do not game metrics—favor meaningful tests.

CI integration: ensure make test (or equivalent) runs all tests and
produces artifacts. When introducing new tech, update CI (see § 11).

Regression safety: when fixing a bug, add a test that fails before and
passes after the fix.

11 · Lean but Fail‑Fast CI (skeleton)
Create .github/workflows/ci.yml and adjust tool commands as needed.

yaml
Copy code
name: CI
on:
  pull_request:
  push:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      md_only: ${{ steps.filter.outputs.md_only }}
    steps:
      - uses: actions/checkout@v4
      - id: filter
        uses: dorny/paths-filter@v3
        with:
          filters: |
            md_only:
              - '**/*.md'

  secret-check:
    runs-on: ubuntu-latest
    outputs:
      has_pages: ${{ steps.echo.outputs.has_pages }}
    steps:
      - id: echo
        run: echo "has_pages=${{ secrets.GH_PAGES_TOKEN != '' }}" >> $GITHUB_OUTPUT

  lint-docs:
    needs: [changes]
    if: needs.changes.outputs.md_only == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          npx --yes markdownlint-cli '**/*.md'
          grep -R --line-number -E '<<<<<<<|=======|>>>>>>>' . && exit 1 || true

  precommit:
    needs: [changes]
    if: needs.changes.outputs.md_only != 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run pre-commit if configured
        run: |
          if [ -f .pre-commit-config.yaml ]; then
            pipx install pre-commit || pip install pre-commit
            pre-commit run --all-files || (echo "pre-commit failed" && exit 1)
          else
            echo "No .pre-commit-config.yaml found"
          fi

  test:
    needs: [changes, precommit]
    if: needs.changes.outputs.md_only != 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Bootstrap (optional)
        run: |
          if [ -x ./.codex/setup.sh ]; then ./.codex/setup.sh; fi
      - name: Lint
        run: |
          if [ -f Makefile ]; then make lint; else echo "No Makefile"; fi
          grep -R --line-number -E '<<<<<<<|=======|>>>>>>>' . && exit 1 || true
      - name: Architecture checks (optional)
        run: |
          if [ -x ./tools/arch_check ]; then ./tools/arch_check; else echo "No arch_check"; fi
      - name: Tests
        run: |
          if [ -f Makefile ]; then make test; else echo "No Makefile"; fi
      - name: Upload coverage artifact (optional)
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: |
            coverage.xml
            coverage/
            lcov.info
            **/coverage.xml
            **/lcov.info
            !**/node_modules/**

  deploy-pages:
    needs: [test, secret-check]
    if: github.ref == 'refs/heads/main' && needs.secret-check.outputs.has_pages == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploy step placeholder (requires GH_PAGES_TOKEN)"
Docs‑only PRs run lint-docs quickly.

Code PRs run pre‑commit, lint, architecture checks (if provided), and tests.

Deploy runs only on main when GH_PAGES_TOKEN exists.

12 · Coding & Documentation Style
4‑space indent (2 for JS/TS if enforced by linter).

≤ 20 logical LOC per function, ≤ 2 nesting levels.

Surround headings/lists/fenced code with a blank line (markdownlint MD022/MD032).

No trailing spaces (git diff --check or pre‑commit).

Wrap identifiers (e.g., __init__) in backticks to avoid MD050.

Every public API has a short doc‑comment with purpose, inputs, outputs, errors.

Add an .editorconfig at repo root to enforce basics consistently.

13 · Security, Secrets & Data Hygiene
Never commit secrets. Provide .env.template with keys and descriptions.

No secrets in logs: redact values; avoid echoing environment variables.

Third‑party code/assets: document license and version in NOTES.md and
pin versions where feasible (lockfiles).

Data: use synthetic/test data by default; never commit PII. If production
samples are essential, encrypt or store outside the repo.

Artifacts: don’t commit build outputs; use dist/** (ignored) or releases.

Optional (human‑enabled): enable GitHub secret scanning, Dependabot/Renovate,
and Code Scanning/CodeQL.

14 · LLM/Coding‑Agent Conduct
Treat this file as the process SSOT; never override rules silently.

With high‑level specs, first write a module map and interface stubs
to /docs/architecture/module-map.md, then implement a minimal vertical
slice.

Read before you code: review /docs/**, TODO.md, NOTES.md, and scan
current code to align with existing patterns. Record a brief plan in NOTES.md.

Provenance: every non‑trivial change adds a short “why” line to NOTES.md
(issue/PR link or reasoning).

No external calls that require secrets unless the secret is present; never
invent credentials; do not paste secrets into code/configs/tests/comments.

Tests required: for each new or modified element, add/adjust tests and
ensure CI runs them (see § 10–§ 11).

15 · How to Update These Rules
Edit only what you need, append a dated bullet to NOTES.md, bump the
version at the top of this file, and open a PR.

When CI/tooling changes (new Action versions, new secrets, new language
runners), update both this guide and the workflow file in the same
PR.

Happy shipping 🚀
