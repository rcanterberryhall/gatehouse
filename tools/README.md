# tools

Reference implementations for the self-documenting-structure standards
(`STANDARDS.md` DOC-006..008).

## `gen_code_map.py` — Python (working, tested)

Maintains, from docstrings alone:

- a **Public API** section in each subpackage's `README.md` (signatures +
  docstring summaries, inside marker-delimited blocks — the hand-written
  header above the markers is never touched);
- a top-level `docs/code_map.md` indexing every package and its role (the
  first line of its `__init__.py` docstring).

Zero dependencies beyond the standard library (`ast` does the parsing).

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
