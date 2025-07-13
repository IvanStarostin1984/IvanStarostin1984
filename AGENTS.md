# Contributor & CI Guide  <!-- AGENTS.md v1.1 -->

> **Read this file first** before opening a pull‑request.  
> It defines the ground rules that keep humans, autonomous agents and CI in‑sync.  
> If you change *any* rule below, **bump the version number in this heading**.

---
Always follow single source of truth.
Always do as specified in signle source of truth.
If something is not specified in single source of truth - choose simplest safest options.
Implement project as specified in TODO.md. Reflect on progress in NOTES.md.
When any issue in codex environment happens, always suggest additions/modifications to this AGENTS.md to prevent such issues in future.
Maintain and develop the project so that after each new feature user will be able to download github repo and run in local IDE to test manually.

## 1 · File‑ownership & merge‑conflict safety

| Rule | Detail |
|------|--------|
| **Distinct‑files rule** | Every concurrent task **must** edit a unique list of non‑markdown files.<br>_Shared exceptions:_ anyone may **append** (never rewrite) `AGENTS.md`, `TODO.md`, `NOTES.md`. |
| **Append‑only logs** | `TODO.md` & `NOTES.md` are linear logs—**never delete or reorder entries**.<br>Add new items **at the end of the file**. |
| **Generated‑files rule** | Anything under `generated/**` or `openapi/**` is **code‑generated** – never hand‑edit; instead rerun the generator. |
| **Search for conflict markers before every commit** | `git grep -n '<<<<<<<\\|=======\\|>>>>>>>'` must return nothing. |

---

## 2 · Bootstrap (first‑run) checklist

1. Run `.codex/setup.sh` (or `./setup.sh`) once after cloning & whenever dependencies change.  
   *The script installs language tool‑chains, pins versions and injects secrets.*  
2. Export **required secrets** (`GIT_TOKEN`, `GH_PAGES_TOKEN`, …) in the repository/organisation **Secrets** console.  
3. Verify the **secret‑detection helper step** in `.github/workflows/ci.yml` (see § 4) so forks without secrets still pass.  
4. On the first PR, update README badges to point at your fork (owner/repo).

---

## 3 · What every contributor must know up‑front

1. **Branch & PR flow** – fork → `feat/<topic>` → PR into `main` (one reviewer required).  
2. **Pre‑commit commands** (also run by CI):  
   ```bash
   make lint                  # all format / static‑analysis steps
   make test                  # project’s unit‑/integration tests
   ```
3. **Style rules** – keep code formatted (`black`, `prettier`, `dart format`, etc.) and Markdown lines ≤ 80 chars; exactly **one blank line** separates log entries.  
4. **Exit‑code conventions** – scripts must exit ≠ 0 on failure so CI catches regressions (e.g. fail fast when quality gates or metric thresholds aren’t met).  
5. **Version‑pin policy** – pin *major*/*minor* versions for critical runtimes & actions (e.g. `actions/checkout@v4`, `node@20`, `python~=3.11`).  
6. **When docs change, update them everywhere** – if ambiguity arises, `/docs` overrides this file.
7. **Log discipline** – when a TODO item is ticked you **must** add the matching
   section in `NOTES.md` *in the same PR*; this keeps roadmap and log in‑sync.  

---

## 4 · Lean but “fail‑fast” CI skeleton

`.github/workflows/ci.yml` — copy → adjust tool commands as needed.

```yaml
name: CI
on:
  pull_request:
  push:
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

  # --- helper step: detect secrets without using them in `if:` ---
  secret-check:
    runs-on: ubuntu-latest
    outputs:
      has_pages_token: ${{ steps.echo.outputs.has_pages }}
    steps:
      - id: echo         # returns 'true' / 'false'
        run: echo "has_pages=${{ secrets.GH_PAGES_TOKEN != '' }}" >> $GITHUB_OUTPUT

  lint-docs:
    needs: [changes]
    if: needs.changes.outputs.md_only == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          npx --yes markdownlint-cli '**/*.md'
          grep -R --line-number -E '<<<<<<<|=======|>>>>>>>' . && exit 1 || echo "No conflict markers"

  test:
    needs: [changes]
    if: needs.changes.outputs.md_only != 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Bootstrap
        run: ./.codex/setup.sh   # idempotent; safe when absent
      - run: make lint
      - run: make test
```

* **Docs‑only changes** run in seconds (`lint-docs`).  
* **Code changes** run full lint + tests (`test`).  
* Add job matrices (multi‑language), action‑lint, or deployment later—guardrails above already catch the 90 % most common issues.  

---

## 5 · Coding & documentation style

* 4‑space indent (or 2‑spaces for JS/TS when enforced by the linter).  
* ≤ 20 logical LOC per function, ≤ 2 nesting levels.  
* Surround headings / lists / fenced code with a blank line (markdownlint MD022, MD032).  
* **No trailing spaces.** Run `git diff --check` or `make lint-docs`.  
* Wrap identifiers like `__init__` in back‑ticks to avoid MD050.  
* Each public API carries a short doc‑comment.  
* Keep Markdown lines ≤ 80 chars to improve diff readability (tables may exceed if unavoidable).  

---

## 6 · How to update these rules

* Edit **only what you need**, append a dated bullet in `NOTES.md`, **bump the version number** at the top of this file, and open a PR.  
* When CI tooling changes (new Action versions, new secrets, extra language runners) **update both** this guide **and** the workflow file in the **same PR**.  

Happy shipping 🚀
