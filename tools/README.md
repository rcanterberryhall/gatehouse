# tools

Reference implementations for the self-documenting-structure standards
(`STANDARDS.md` DOC-006..008).

## `gen_code_map.py` — Python (working, tested)

### Why it exists

STANDARDS.md DOC-007/008 require every project to keep an orientation
layer — per-package API maps and a top-level package index — that is
**derived from docstrings, never hand-edited, and enforced in CI** so it
cannot drift from the code. This script is that mechanism as one vendorable
file: docstrings are the single source of documentation truth; everything
this script writes is a projection of them.

It maintains:

- a **Public API** section in each subpackage's `README.md` (signatures +
  docstring summaries, inside marker-delimited blocks — the hand-written
  header above the markers is never touched);
- a top-level `docs/code_map.md` indexing every package and its role (the
  first line of its `__init__.py` docstring).

Zero dependencies beyond the standard library (`ast` does the parsing).
`test_gen_code_map.py` is its fixture test — also stdlib-only, run with
`python tools/test_gen_code_map.py` — and gatehouse's own CI runs it along
with a version-pinned ruff gate (practicing CODE-003).

### How it works

Two phases: it reads and renders everything in memory first, and touches no
file until that is complete.

1. **Discover.** List the immediate subdirectories of `--src-root` that
   contain an `__init__.py`, skipping `_`-prefixed names. Each is one
   package; each gets one README.
2. **Extract.** Statically parse every public module in each package with
   the stdlib `ast` module — the code is **never imported or executed**, so
   the script is safe to run on any tree and needs no project environment.
   From the parse tree it collects public symbols (functions, classes and
   their public members, annotated constants) with their signatures and
   docstrings. `__all__`, when present, overrides the leading-underscore
   rule and pulls in re-exports from private modules.
3. **Render.** Compute the complete desired content of every target file
   in memory: each package README with only its marker-delimited block
   replaced (all hand-written text preserved byte-for-byte), and
   `docs/code_map.md` with only its index table replaced.
4. **Compare, then act** — the only step that depends on the mode:
   - **default:** write each file whose current content differs from the
     rendered content; identical files are left untouched (idempotent — a
     second run is always a no-op).
   - **`--check`:** write nothing; print a unified diff of every file that
     differs and exit 1. Because check and write share the same rendered
     desired-state from step 3, a clean `--check` guarantees regeneration
     would change nothing — the two modes cannot disagree.
   - **`--init`:** additionally create any missing README or code map from
     the bootstrap template (`_TODO_` narrative header + filled API block).

Exit codes: `0` clean/regenerated, `1` drift detected by `--check`, `2`
configuration error (missing README without `--init`, or a damaged/missing
marker pair — the script refuses to guess where a block belongs).

### Adopting it in a new project

1. Copy `gen_code_map.py` into your repo (conventionally `tools/` or
   `scripts/`).
2. Bootstrap: `python tools/gen_code_map.py --src-root src/<yourpkg> --init`
   — creates each package README with `_TODO_` placeholders for the
   narrative header and a filled-in Public API block, plus
   `docs/code_map.md`.
3. Fill in the `_TODO_`s by hand (purpose, key concepts, relations,
   pitfalls). This is the half only a human/designer knows.
4. Wire the enforcement (DOC-008):

   `.pre-commit-config.yaml`:

   ```yaml
   repos:
     - repo: local
       hooks:
         - id: gen-code-map
           name: regenerate code map
           entry: python tools/gen_code_map.py --src-root src/yourpkg
           language: system
           pass_filenames: false
   ```

   CI step (any runner):

   ```yaml
   - name: Code map up to date
     run: python tools/gen_code_map.py --src-root src/yourpkg --check
   ```

   `--check` exits 1 and prints a diff when any README or the code map
   would change — documentation drift fails the build.

### Conventions the script assumes

- Layout `src/<package>/<subpackage>/` — each subpackage has `__init__.py`
  with a one-line docstring (its role in the code map) and gets a README.
- Public = not underscore-prefixed, or listed in the package's `__all__`
  (when `__all__` is set it wins, including re-exports from private
  modules).
- Never edit between the `<!-- BEGIN/END: AUTO-GENERATED ... -->` markers;
  everything outside them is yours.

## C++ — design sketch (theoretical, no tooling shipped)

The mechanism transfers to C++ conceptually, but **no C++ generator is
provided here and none of this has been exercised** — treat this section as
a starting point, not a tool.

The mapping:

| Python mechanism | C++ equivalent |
|---|---|
| Package (`__init__.py` directory) | Source/include subdirectory (module) |
| Docstrings | Doxygen comments (`///`, `/** ... */`) |
| Package one-liner (`__init__` docstring) | `\file` + `\brief` block in the module's primary header |
| Public API boundary (no `_` prefix / `__all__`) | The **header files** — what's declared in `include/` IS the public API; skip `detail`/`internal` namespaces |
| `ast` (stdlib parser) | **No stdlib parser exists** — extract with libclang (Python bindings over the C API) or Doxygen XML output |

The extraction backend is the only hard part. Two viable routes:

1. **libclang** (`pip install clang==<major matching your libclang.so>`):
   parse each header, walk cursors for public declarations (free functions,
   classes/structs, public methods, enums, type aliases), read
   `cursor.brief_comment` / `raw_comment` for the doc text. Robust against
   real C++ (templates, overloads, macros), but adds a
   version-sensitive dependency — the bindings' major version must match
   the installed `libclang.so`.
2. **Doxygen XML**: run `doxygen` with `GENERATE_XML = YES`, transform the
   XML into the marker-block markdown. Heavier, but Doxygen is the
   de-facto C++ documentation standard and handles everything.

Everything *around* extraction — the marker-delimited blocks, the
hand-written narrative header, `--init` bootstrapping, `--check` drift
detection, pre-commit + CI enforcement — is language-agnostic and can be
lifted from `gen_code_map.py` unchanged.

Illustrative target output for a header like:

```cpp
/// \file
/// \brief Alpha module — parses widgets into records.

namespace acme::alpha {

/// Parse one raw widget line into a record.
/// \param raw    the input line
/// \param strict throw on malformed input when true
WidgetRecord parse_widget(std::string_view raw, bool strict = true);

}  // namespace acme::alpha
```

would be:

```markdown
### `acme/alpha/parser.hpp`
*Alpha module — parses widgets into records.*

- `WidgetRecord parse_widget(std::string_view raw, bool strict = true)` — Parse one raw widget line into a record.
```

## C# / .NET — design sketch (theoretical, no tooling shipped)

The best-positioned language for this mechanism — the toolchain half-builds
it for you. Untested here; treat as a starting point.

| Python mechanism | .NET equivalent |
|---|---|
| Package (`__init__.py` directory) | Project (`.csproj`) or namespace within a solution |
| Docstrings | XML doc comments (`/// <summary>...`) — parsed by the compiler itself |
| Package one-liner (`__init__` docstring) | The project's `<Description>` property, or the `<summary>` on the namespace's anchor type |
| Public API boundary (no `_` prefix / `__all__`) | The `public` keyword — explicit in the language |
| `ast` (stdlib parser) | **The build output**: `<GenerateDocumentationFile>true</GenerateDocumentationFile>` makes every build emit an XML inventory of all documented public members with full signatures. Roslyn (Microsoft.CodeAnalysis) is the full parser if you outgrow it. |

Two properties make .NET the easy case:

1. **No parsing needed.** The compiler-generated XML doc file already *is*
   the extracted API — a generator is a small transform of that XML into
   the marker-block markdown, run after build.
2. **Docstring enforcement is native.** Compiler warning **CS1591** fires
   on any undocumented public member; with `<TreatWarningsAsErrors>`, the
   equivalent of DOC-002 is enforced by compilation itself — no custom
   check required.

Illustrative source:

```csharp
namespace Acme.Alpha;

/// <summary>Parse one raw widget line into a record.</summary>
/// <param name="raw">The input line.</param>
/// <param name="strict">Throw on malformed input when true.</param>
public WidgetRecord ParseWidget(string raw, bool strict = true) { ... }
```

target output:

```markdown
### `Acme.Alpha`

- `WidgetRecord ParseWidget(string raw, bool strict = true)` — Parse one raw widget line into a record.
```

## TypeScript — design sketch (theoretical, no tooling shipped)

Strong fit for TypeScript; plain JavaScript is the weak case (no types,
JSDoc optional and unverifiable — it works only as well as comment
discipline holds). Untested here; treat as a starting point.

| Python mechanism | TypeScript equivalent |
|---|---|
| Package (`__init__.py` directory) | Directory module with an `index.ts` barrel |
| Docstrings | TSDoc comments (`/** ... */`) |
| Package one-liner (`__init__` docstring) | `@packageDocumentation` comment block in `index.ts` |
| Public API boundary (no `_` prefix / `__all__`) | `export` — and the barrel's re-exports play exactly the role of `__all__` |
| `ast` (stdlib parser) | TypeScript compiler API (`ts.createProgram` + symbol walk) |
| `--check` drift detection | **api-extractor** (Microsoft) does this natively: it emits an API report file you commit, and CI fails when the exported API drifts from the report — DOC-008 as an off-the-shelf tool |

For **Angular/React** the unit shifts: the public API isn't a module of
functions but a **component's contract** — `@Input()`/`@Output()` pairs in
Angular, props in React, plus routes and injectable services. The map
becomes a component index rather than a function map; Compodoc (Angular)
and react-docgen (React) are the extractors. Same per-directory READMEs,
same marker blocks, different extraction backend.

Illustrative source:

```typescript
/**
 * Parse one raw widget line into a record.
 * @param raw - the input line
 * @param strict - throw on malformed input when true
 */
export function parseWidget(raw: string, strict = true): WidgetRecord { ... }
```

target output:

```markdown
### `acme/alpha`

- `parseWidget(raw: string, strict = true): WidgetRecord` — Parse one raw widget line into a record.
```
