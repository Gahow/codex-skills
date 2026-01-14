---
name: repo-context
description: Build accurate context for large external dependencies by identifying the exact repo/version, preferring local installed sources, and summarizing key APIs before coding.
metadata:
  short-description: Verify external repo context
---

# Repo Context (GitHub & Large Dependencies)

## Trigger
Use when the user asks to read GitHub source or docs, or when a task depends on the exact behavior of large dependencies (e.g., vLLM).

## Local-First Rule
Always look for local, already-downloaded dependencies used by the project before cloning or fetching anything.

## Workflow
1. Identify dependency and version
   - Use `rg` to find dependency pins (for example: `pyproject.toml`, `requirements.txt`, `uv.lock`, `Cargo.lock`, `go.mod`).
   - Check for vendored copies or git submodules.
   - If version or repo URL is unclear, ask the user for the exact source and version/commit.
2. Acquire source (local-first)
   - Prefer local sources already used by the project: `.venv`, `vendor/`, `third_party/`, `deps/`, or submodules.
   - For Python deps, run `source .venv/bin/activate`, then locate the package path with:
     - `python -c "import <pkg>; print(<pkg>.__file__)"`
   - If a local wheel or source tree exists, treat it as authoritative for that version.
   - If absent, clone into a local `deps/` directory or a user-provided path.
   - Checkout the exact tag/commit and record `git rev-parse HEAD`.
   - If network access is restricted, request approval before cloning or fetching.
3. Load docs for that version
   - Read `README*`, `docs/`, `examples/`, `CHANGELOG*`, and release notes.
   - Avoid using website docs if they do not match the pinned version; if only web docs exist, note the mismatch.
   - For vLLM, confirm scheduler, block manager, and engine APIs from source rather than assumptions.
4. Build a short context pack
   - Keep a few bullets: version/commit, key APIs, and any differences from common assumptions.
   - Include file paths with line references for critical behavior.
5. Proceed with verified context
   - Use the context pack to guide implementation.
   - If anything remains unclear, ask before coding.

## Commands
- Search: `rg --files`, `rg "pattern" path`
- Repo metadata: `git status -sb`, `git log -1`, `git rev-parse HEAD`
- Structure: `ls` (use `tree` only if available)

## Context Pack Template
```
repo: <url>
version: <tag> / <commit>
local path: <path>
key APIs: <names + file paths>
notes/risks: <brief bullets>
```

## Constraints
- Do not modify external repos unless the user asks.
- Avoid heavy builds/tests without explicit request.
- Keep the context pack minimal; do not paste large docs into the conversation.
