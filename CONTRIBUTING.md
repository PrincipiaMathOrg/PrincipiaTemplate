# Adapting this template

1. Rename the package and namespace throughout the repository.
2. Put the proof development in the library and import it from `Solution.lean`.
3. Rewrite `Challenge.lean` as a small, independently auditable statement
   surface. Keep its advertised declarations statement-only with `sorry`; the
   corresponding proofs belong in `Solution.lean`. Keep its imports to Lean core
   and Mathlib wherever possible.
4. Update `comparator.json` with every advertised theorem and any definition
   holes. Definition holes require special scrutiny.
5. Replace every `TEMPLATE` value in `formalization.yaml` with honest,
   independently checkable metadata. Run
   `ruby scripts/validate-formalization.rb`; it parses the file and lists every
   retained sentinel, including deliberately invalid classification, proof,
   automation, and review defaults. Replace a placeholder list with `[]` only
   where its adjacent comment permits that; lists described as required must
   remain nonempty. In particular, follow the result-origin instructions beside
   `sources` rather than replacing that list with `[]`. Keep the Apache-2.0
   `LICENSE` file and the matching `project.license: "Apache-2.0"` metadata.
   This template supports only that root licence. Leave the `repository`
   example commented out unless this repository is only a wrapper around a
   separately pinned substantive formalization.
6. Run `lake update` and `cd docbuild && lake update` after changing dependencies,
   then commit both manifest files.
7. Run `lake build`, build the docs, and run Comparator before relying on the
   result.

Do not keep the toy theorem.

## Module system and file sizes

Every regular `.lean` source file in the repository must use Lean's module
system and contain at most **10,000 physical lines**. This includes Challenge,
Solution, unused source files, generated certificates, contained projects, and
local path dependencies. Ordinary comments may precede the `module` header;
module documentation belongs after it. Blank and comment lines count. LF and
CRLF each delimit one line; an unterminated final line counts, and a final
newline does not add an empty line.

Lake configuration files named `lakefile.lean` are exempt from the module
header requirement, but still have the 10,000-line cap. Files below `.git` or
`.lake` are excluded; `.lean` symbolic links are rejected so a link cannot hide
an oversized source file.

Porting requires more than adding `module`: make the declarations needed by
other modules public, use `public import` where the public interface needs an
import, and expose definitions whose bodies clients need. See
[Lean's modules and visibility reference](https://lean-lang.org/doc/reference/latest/Source-Files-and-Modules/#modules-and-visibility).
Rebuild and rerun Comparator after porting. Split oversized files into smaller
modules or reduce generated certificates; do not hide them in excluded paths.

Run `python3 scripts/check-lean-sources.py` before `lake build`. CI repeats this
non-executing check before installing dependencies or building. The supplied
modules use public imports and declarations; keep this structure as you add
files.
