# TODO – Road‑map  (last updated: YYYY‑MM‑DD)

> *Record only high‑level milestones here; break micro‑tasks out into Issues.*  
> **When you finish a task, tick it and append a short NOTE entry
> (see NOTES.md).**  
> Keep this list ordered by topic and **never reorder past items**.

## 0 · Project bootstrap
- [ ] Commit starter governance files (`AGENTS.md`, `TODO.md`, `NOTES.md`,
      minimal CI)
- [ ] Add `.codex/setup.sh`; ensure it is idempotent and exits 0
- [ ] Configure `make lint` and `make test` (cover every language tool‑chain)
- [ ] Audit repository & docs; identify the single source of truth
      (spec, assignment …) and reference it in README
- [ ] Generate initial dependency manifests (`requirements.txt`,
      `package.json`, `pubspec.yaml`, …) with pinned versions
- [ ] Define ownership of all generated code in `/generated/**` and record the
      regeneration command in `AGENTS.md`
- [ ] Push the first green CI run (docs‑only + full‑tests job)

## 1 · Core functionality  
*Repeat the five‑bullet block below for every MVP feature A, B, C, …*  
- [ ] Analyse source‑of‑truth docs; define acceptance criteria for **feature A**  
- [ ] Document assumptions / edge‑cases for feature A in `/docs` or README  
- [ ] Implement feature A  
- [ ] Add unit / integration tests for feature A  
- [ ] Wire CI quality gate (coverage ≥ 80 %, metric thresholds, etc.) that
      exits 1 on regression

## 2 · Documentation & CI
- [ ] Write README quick‑start (clone → setup → test)
- [ ] Add full doc build (Sphinx / JSDoc / dart‑doc as applicable)
- [ ] Integrate secret‑detection helper step in CI (`has_token` pattern)
- [ ] Extend CI matrix for all runtimes (Python, Node, Dart, Rust, …)
- [ ] Add Actionlint + markdown‑link‑check jobs and pin their versions
- [ ] Publish docs to GitHub Pages when `GH_PAGES_TOKEN` is present

## 3 · Quality & automation
- [ ] Add pre‑commit hooks (formatters, linters, markdownlint, actionlint)
- [ ] Enforce coverage threshold (≥ 80 % branch, exclude `/generated/**`)
- [ ] Add linters for conflict markers, trailing spaces and NOTES ordering
- [ ] Introduce dependabot / Renovate with the version‑pin policy from
      `AGENTS.md`

## 4 · Stretch goals
- [ ] Containerise dev environment (Dockerfile or dev‑container.json)
- [ ] Auto‑deploy docs & storybooks on each tag
- [ ] Publish packages (PyPI, npm, pub.dev) via release workflow
- [ ] Add optional load‑testing / performance CI stage

---

### Add new items below this line  
*(append only; keep earlier history intact)*
