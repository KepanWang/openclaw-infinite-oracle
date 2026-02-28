# AGENTS.md

This file is for coding agents operating in this repository.
It is intentionally concrete and evidence-based.

## 1) Repository Snapshot

- Language: Python
- Main runtime: `peco_loop.py`
- Docs: `README.md`, `README_EN.md`, `SKILL.md`
- Current scope: single executable module + documentation
- No detected package manager metadata (`pyproject.toml`, `requirements.txt`, `setup.py`)
- No detected CI workflows (`.github/workflows/*`)
- No detected test suite files (`test_*.py`, `*_test.py`)
- No detected lint/format config (`ruff.toml`, `.flake8`, `mypy.ini`, `black` config)

## 2) Build / Lint / Test Commands

Because this repo is minimal, "build/lint/test" are mostly runtime and syntax checks.

### 2.1 Core Commands (known to work in this repo)

- Show CLI and options:
  - `python3 peco_loop.py --help`
- Syntax-only compile check:
  - `python3 -m py_compile peco_loop.py`
- Start loop daemon (default behavior):
  - `python3 peco_loop.py`

### 2.2 Controlled Runtime Check (safe local smoke)

Use this when you need one bounded iteration while developing:

- `python3 peco_loop.py --max-iterations 1 --sleep-seconds 1 --log-level DEBUG`

Notes:
- This may still require OpenClaw gateway/token setup depending on environment.
- If gateway is unavailable, treat failure as environment/config issue, not syntax issue.

### 2.3 "Single Test" Guidance

There is no test framework configured right now.

- Current reality: no canonical "run one test" command exists.
- If/when pytest is added, use this pattern for a single test:
  - `pytest path/to/test_file.py::test_name`
- If/when unittest is added, use this pattern for a single test:
  - `python3 -m unittest path.to.module.TestClass.test_method`

### 2.4 Suggested Local Quality Sequence

When making changes, run in this order:

1. `python3 -m py_compile peco_loop.py`
2. `python3 peco_loop.py --help`
3. Optional bounded runtime check (Section 2.2)

## 3) Code Style Guidelines (from existing code)

These conventions are inferred from `peco_loop.py` and should be preserved.

### 3.1 Imports and Module Layout

- Keep shebang + module docstring at file top.
- Keep `from __future__ import annotations` directly after docstring.
- Group imports by source (stdlib first; local/third-party if introduced later).
- Prefer explicit imports over wildcard imports.
- Keep constants near top-level after imports.

### 3.2 Naming Conventions

- Constants: `UPPER_SNAKE_CASE` (example: `ALLOWED_TRANSITIONS`).
- Classes/exceptions/dataclasses: `PascalCase`.
- Functions/methods/variables: `snake_case`.
- Internal helpers/attributes: leading underscore (example: `_ensure_token`).
- CLI flags: kebab-case in argparse options (example: `--max-iterations`).

### 3.3 Types and Data Modeling

- Use type hints broadly on parameters and returns.
- Keep `from __future__ import annotations` for forward-compatible typing.
- Existing file uses `typing` aliases (`Dict`, `List`, `Optional`, `Any`); match local style.
- Use `@dataclass` for structured state/parsed payload objects.
- Keep state payloads JSON-serializable where possible.

### 3.4 Error Handling

- Define domain exceptions (`LoopError` family) for meaningful failure classes.
- Catch specific exceptions first (`HTTPError`, `URLError`, parse errors).
- For broad catches, always log or re-raise with context; do not swallow silently.
- Preserve exception chains via `raise ... from exc` when translating errors.
- In loop execution, degrade gracefully and continue when safe; halt only on thresholds.

### 3.5 Logging and Observability

- Use `logging` (not prints) for runtime diagnostics.
- Include iteration/session/phase in key logs.
- Keep log messages operationally actionable.
- Write logs to both file and stdout through configured handlers.
- Keep notifier side effects non-fatal (warnings over hard crashes where appropriate).

### 3.6 File and I/O Conventions

- Use `pathlib.Path` for filesystem work.
- Ensure parent directories exist before writing files.
- Prefer UTF-8 explicitly for reads/writes.
- Use atomic replacement for JSON state writes (`*.tmp` then `replace`).
- Keep override/backlog/state files line-oriented and append-friendly.

### 3.7 CLI and Runtime Behavior

- Add new runtime settings through `argparse` with help text.
- Maintain backward-compatible flags when practical (legacy alias pattern exists).
- Keep default values conservative for long-running loops.
- Bound risky behavior via explicit thresholds (failures/repeat signatures/timeouts).
- Keep PECO phase transitions validated and normalized.

### 3.8 JSON/Protocol Discipline

- Keep machine-parseable contracts strict and validated.
- Validate required keys and value ranges before state transitions.
- Never trust remote payload shape without checks.
- Keep structured output parsers deterministic and explicit.

## 4) Editing Guidance for Future Agents

- Make minimal, surgical changes; avoid broad refactors unless requested.
- Preserve current behavior for gateway auth/session headers unless required.
- When touching Feishu integration, keep failure path non-blocking.
- Maintain bilingual safety where already present in user-facing strings.
- Do not introduce irreversible/destructive shell operations in runtime flows.

## 5) Rule Files Check (Cursor / Copilot)

The following were checked and not found in this repository:

- `.cursorrules`
- `.cursor/rules/`
- `.github/copilot-instructions.md`

If any of these files are added later, update this AGENTS.md immediately so agents follow them.

## 6) Practical "Done" Checklist for Code Changes

Before finishing a task in this repo:

1. Run `python3 -m py_compile peco_loop.py` (or all changed `.py` files).
2. Run `python3 peco_loop.py --help` if CLI surface changed.
3. If runtime logic changed, run one bounded iteration locally when environment permits.
4. Confirm logs/errors remain informative and non-silent.
5. Update README/SKILL docs when behavior or flags materially change.

## 7) Git Commit Workflow (for agents)

Use this repository's commit style and keep commits focused.

1. Check current changes:
   - `git status --short`
   - `git diff`
2. Stage only relevant files:
   - `git add peco_loop.py AGENTS.md`
   - Or stage selectively per task scope.
3. Commit with concise prefix-style message (examples in history):
   - `git commit -m "fix: <why this change is needed>"`
   - `git commit -m "docs: <why docs changed>"`
4. Verify commit created cleanly:
   - `git status --short`
   - `git log -1 --oneline`
5. Push to tracked remote branch when required:
   - `git push`

ClawHub note (if release is needed in your workflow):
- This repo is already registered with ClawHub.
- After Git push, run your team-standard ClawHub publish command (for example `clawhub publish`) only when you intend to release.

This repository is small; correctness and operational safety matter more than framework ceremony.
