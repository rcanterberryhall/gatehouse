# Engineering Standards (Day-0)

Standing requirements that apply to **every** project started through
gatehouse, unless a sheet's §7 explicitly overrides them. They are written in
the same shall/should language as a requirements sheet so they can be
verified at Phase 5 like any other requirement — mostly by Inspection.

They are "day-0" standards deliberately: each one is cheap on day 0 and
expensive to bolt on later. The first implementation plan of any new project
establishes all of them before feature work starts.

A second purpose runs through the DOC clauses: **self-documenting structure
makes the codebase efficient for both people and the AI assistant.** An
assistant (or a new engineer) orients from the code map and package READMEs
instead of scanning source files — cheaper, faster, and more accurate — and
because the API maps are generated from docstrings and enforced in CI
(DOC-007/008), that orientation layer cannot silently rot the way
hand-maintained docs do.

## Documentation — DOC

| ID | Requirement | Verify |
|---|---|---|
| DOC-001 | Every project shall have a `README.md` from its first commit covering: what the project is, how to run it, how to test it, and its configuration (env vars / settings table). | I |
| DOC-002 | Every public module, class, and function shall carry a docstring stating what it does, its arguments, and its return value (Google style recommended). | I |
| DOC-003 | Web/UI projects shall embed user-facing help in the UI itself (help page, tooltips, or inline text) — the README is for engineers, not users. | D |
| DOC-004 | Documentation shall be factual and behavior-describing; no aspirational or rhetorical filler. Docs are written to stand alone, not as diffs against prior versions. | I |
| DOC-005 | The README should stay current: any change that alters setup, configuration, or user-visible behavior updates it in the same change set. | I |
| DOC-006 | Every source package (each directory with an `__init__.py` or language equivalent) shall carry a one-line docstring stating its role, and its own `README.md` with a short hand-written header: purpose, key concepts, how it relates to other packages, and common pitfalls. | I |
| DOC-007 | Once the source tree has more than one package, the project shall maintain a **code map**: a top-level `docs/code_map.md` indexing every package and its role, and a per-package public-API section (signatures + docstring summaries) **auto-generated from the docstrings** into marker-delimited blocks of each package README. Docstrings are the single source; the map is derived, never hand-edited. | I |
| DOC-008 | The code-map generator shall be enforced automatically: run on pre-commit to regenerate, and run in CI in check mode so that documentation drift fails the build. | D |
| DOC-009 | Every ongoing project shall keep a changelog (e.g. a `CHANGELOG.md`) recording notable changes — features added, behavior changes, fixes — as they land, grouped under the version or tag they shipped in (VCS-003). It is updated in the same change set that makes the change, so the project's evolution is legible without reconstructing it from git history. | I |

A vendorable reference implementation of DOC-007/008 for Python lives in
[`tools/gen_code_map.py`](tools/gen_code_map.py) (stdlib-only), with adoption
steps and a theoretical C++ mapping in [`tools/README.md`](tools/README.md).

## Code quality — CODE

| ID | Requirement | Verify |
|---|---|---|
| CODE-001 | Code shall carry type hints (or the language's equivalent) from the first commit, with a type checker (e.g., mypy/pyright) running in CI. | I |
| CODE-002 | Errors shall be captured and handled at system boundaries (API routes, file/network I/O, subprocess calls) — no silent excepts, no raw tracebacks to users. | I |
| CODE-003 | Code shall be auto-formatted and linted from the first commit, with the tool **pinned to an exact version** and both checks run in CI so a violation fails the build (Python: ruff covers both roles). An unpinned formatter drifts between environments and buries real changes in reformat noise. | D |
| CODE-004 | Each project shall declare the code-style standard for its language and follow it (Python: **PEP 8** for style, **PEP 257** for docstring conventions). The pinned toolchain (CODE-003) is the arbiter: where the formatter/linter and the written standard disagree, the tool's output wins — style questions are settled by the tool, never by review debate. | I |

## Testing — TST

| ID | Requirement | Verify |
|---|---|---|
| TST-001 | Every project shall have a test runner and CI executing it from the first PR/merge. A failing test fails the build. | D |
| TST-002 | Pure logic (parsers, validation rules, calculations, state machines) should be developed test-first; it is the code that is cheapest to test and most expensive to debug in place. | I |
| TST-003 | New features shall land with unit tests for their logic; bug fixes shall land with a test that reproduces the bug. | I |

## Logging & observability — LOG

| ID | Requirement | Verify |
|---|---|---|
| LOG-001 | Every project shall use structured logging from the first commit — leveled, timestamped, and consistent — never bare `print`. | I |
| LOG-002 | Data-processing projects shall record provenance: what input produced each output, so any result can be traced back to its source. | I |
| LOG-003 | Findings, errors, and warnings produced by processing should be captured as data (queryable records), not only as log lines. | I |

## Containerization & portability — CTR

A container image is built from a checkout, and a checkout is platform-specific.
The most common way a working Linux image fails on someone else's machine is
line-ending corruption: git's Windows default (`core.autocrlf=true`) rewrites
text files to CRLF on checkout, and a shell script that reaches the container's
kernel as `#!/bin/sh\r` cannot exec its interpreter — the ENTRYPOINT dies with
exit code 255. This is cheap to prevent on day 0 (one committed file) and
painful to diagnose after the fact.

| ID | Requirement | Verify |
|---|---|---|
| CTR-001 | Any project that ships a container image shall commit a `.gitattributes` from its first Dockerfile that forces LF on every file executed inside the image — shell scripts (`*.sh`), the `Dockerfile`, and any entrypoint — so a Windows checkout cannot rewrite a script's shebang to `#!/bin/sh\r` and kill the ENTRYPOINT with exit code 255. A blanket `* text=auto eol=lf` default with explicit `eol=lf` on the exec-critical patterns satisfies this. | I |
| CTR-002 | Vendored data, reference libraries, or test fixtures that intentionally ship with CRLF or mixed line endings shall be excluded from normalization (`-text` on their paths) so the line-ending policy never rewrites their bytes — which would otherwise dirty the working tree after a re-checkout and break byte-compared fixtures. | I |
| CTR-003 | Binary assets (images, PDFs, office documents, archives, model files) shall be marked `binary` so no normalization can ever touch them, rather than relying solely on git's text/binary auto-detection. | I |

A canonical `.gitattributes` satisfying CTR-001…003:

```gitattributes
* text=auto eol=lf

*.sh            text eol=lf
Dockerfile      text eol=lf
*.dockerfile    text eol=lf

*.png *.jpg *.pdf *.ico   binary

# vendored/reference data that ships with CRLF on purpose:
path/to/vendored/**       -text
```

Note for existing projects adopting this later: `.gitattributes` only governs
future checkouts, so a working tree already checked out with CRLF will not fix
itself. After committing the file, re-materialize once — `git rm --cached -r .`
then `git reset --hard` — and rebuild the image with `--no-cache` so no layer
cached from the CRLF copy survives.

## Version control & branch hygiene — VCS

History is a verification artifact: Phase 5 traces a behavior back to the
commit that introduced it, so the way branches and merges are kept directly
governs how traceable the project is. These are standing practices, confirmed
by **Inspection of the git history** rather than by a CI gate.

| ID | Requirement | Verify |
|---|---|---|
| VCS-001 | Every feature shall be developed on its own dedicated branch (e.g. `feat/<topic>`) — one branch per feature; nothing is committed directly to the default branch. The default branch only ever advances by merge. | I |
| VCS-002 | A branch shall merge back with `--no-ff`, so the slice stays an identifiable unit in history (a merge commit) rather than being flattened into the mainline. | I |
| VCS-003 | Meaningful checkpoints — a feature merge, a milestone — should be marked with a lightweight `v0.x` tag so any behavior can be traced to when it entered. This is code pedigree, not release management: landmarks, not strict semver. | I |
| VCS-004 | Before any merge or commit intended for the default branch, the currently checked-out branch shall be confirmed (e.g. `git branch --show-current`) — a shared working root may be sitting on another task's branch. After the merge, the result shall be sanity-checked (the default branch's tree matches the merged tip) before it is relied on. | I |
| VCS-005 | Independent or parallel work streams should be isolated in separate branches or worktrees, so unrelated changes never combine in one commit or an accidental merge. | I |
| VCS-006 | When a feature is finished, its integration route — merge into the default branch, or open a pull request for review — shall be an explicit decision put to the engineer, never assumed. The branch is not silently merged. | I |
| VCS-007 | Execution of a plan shall begin from a clean working tree. If uncommitted work is present when execution is about to start, it is surfaced to the engineer for direction (commit, stash, or discard) before any new work is written — new work never silently layers onto unrelated uncommitted changes. | I |
| VCS-008 | Merged feature branches shall be retained as history, not deleted. The branch ref is a permanent, human-readable landmark of exactly which commits made up a feature; together with the `--no-ff` merge (VCS-002) and any milestone tag (VCS-003) it keeps the project's pedigree navigable. | I |

## How these enter a project

1. The intake sheet's **§7 Applicable Standards & Codes** cites this document
   by default. Delete the row only with a reason — that deletion is itself a
   documented *should*-deviation.
2. **Phase 4** (ROADMAP.md): the first implementation plan establishes
   DOC-001, CODE-001, CODE-003, TST-001, and LOG-001 before any feature
   work.
3. **Phase 5**: these IDs are verified alongside the sheet's own
   requirements, by the method listed.
