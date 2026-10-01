# PrincipiaTemplate

[![CI](https://github.com/PrincipiaMathOrg/PrincipiaTemplate/actions/workflows/ci.yml/badge.svg)](https://github.com/PrincipiaMathOrg/PrincipiaTemplate/actions/workflows/ci.yml)

The starting point for the Lean 4 formalization repository of a
[Principia](https://principia-math.com/) project. Principia creates a
repository from this template when a project's owner asks for one, and fills in
the statement, the proof and the metadata from the project. It can also be used
by hand: replace the toy theorem and all `TEMPLATE` metadata, and keep the
separation between the human-auditable statement and the proof.

The layout follows the Palomar template.

## Repository map

- `Challenge.lean` is the small statement surface a reader audits.
- `Solution.lean` connects the same declaration to the completed proof.
- `PrincipiaTemplate/` contains the full proof development.
- `comparator.json` tells `lake comparator` which declarations must match.
- `formalization.yaml` records the public result description, provenance,
  authorship, automation, fidelity, and review information.
- `LICENSE` contains the Apache License 2.0 terms declared by
  `project.license`.
- `docbuild/` is the recommended nested doc-gen4 project.
- `scripts/verify-comparator.sh` runs the `lake comparator` that ships in
  this project's own toolchain over the checked-in `comparator.json`,
  registering the toolchain's bundled independent kernels (NanoDa and con-ron);
  the optional `enable_nanoda` field is ignored, and `external_kernels` is not
  a submitter field.

The root uses `lakefile.toml`, a Lean toolchain that ships `lake comparator`
(`leanprover/lean4:v4.35.0-rc2` or later), and committed Lake manifests.
Everything that judges the proof comes from `lean-toolchain`; there is no
separate verifier pin to keep in step with it. GitHub Actions builds the Lean
project with `lean-action`, generates API documentation with doc-gen4, and
independently checks the advertised statement with `lake comparator`. Actions
are pinned to immutable commits.

## Start a real project

1. Click **Use this template** on GitHub and clone the new repository.
2. Rename `PrincipiaTemplate` in the Lake package, module directory, namespace,
   Comparator declaration, and metadata.
3. Replace the example library, `Challenge.lean`, and `Solution.lean`.
   Keep `Challenge.lean` as the small statement-only surface, with one `sorry`
   for each advertised declaration; put the proofs in `Solution.lean`, where
   Comparator checks them against those statements. The proof-status counts in
   `formalization.yaml` exclude the deliberate Challenge `sorry`s.
4. Replace every `TEMPLATE` value in `formalization.yaml`. Values that might
   otherwise look like plausible defaults, including repository role,
   classifications, proof counts, automation method, and review status, are
   deliberately invalid until you choose them. Replace a placeholder list with
   an empty list only where its adjacent comment permits that; lists described
   as required must remain nonempty.

   Write `project.description` as the concise public abstract for the
   formalization as a whole. It should let a mathematical reader identify the
   subject and principal result families; it is not an inventory of Comparator
   declarations. `status.main_results` is optional: add it only when a short
   curated project-level list is useful.

   The `sources` list must remain nonempty. Every source needs a title and a
   relationship, exactly one of `formalizes`, `adapts`, `independently-proves`,
   `background`, or `other`. Choose one result origin: for a result first
   presented by the formalization, include a descriptive source with
   `type: original-proof` and `relationship: other`; every additional source
   must use `background` or `other`. Otherwise, omit `type: original-proof`,
   and give at least one cited mathematical source a `formalizes`, `adapts`, or
   `independently-proves` relationship. A new proof of a known published result
   is source-based and uses `independently-proves`, not `original-proof`.

   Keep the repository's Apache-2.0 `LICENSE` file and the matching
   `project.license: "Apache-2.0"` metadata. This template supports only that
   root licence. Cited sources and dependencies retain their own licences.
5. Update and commit dependency pins:

   Before fetching and building the dependency closure, budget several GiB of
   free space. Both `.lake/` directories are generated and must not be
   committed.

   ```text
   lake update
   (cd docbuild && MATHLIB_NO_CACHE_ON_UPDATE=1 lake update)
   ```

6. Run the project checks:

   ```text
   lake exe cache get
   lake build
   (cd docbuild && lake build PrincipiaTemplate:docs)
   ruby scripts/validate-formalization.rb
   ./scripts/verify-comparator.sh
   ```

   The metadata command parses the YAML, requires the Apache-2.0 root-licence
   declaration, and reports the path of every retained template sentinel. CI
   also detects the checked-in `LICENSE` file independently and runs an
   explicit `--expect-template` check only in the canonical
   `PrincipiaMathOrg/PrincipiaTemplate` repository, proving that the shipped
   toy metadata still has exactly the intended sentinel surface. Every other
   repository, including repositories made with **Use this template**, runs the
   ordinary command and requires every sentinel to be replaced. CI also runs
   the corresponding build, documentation, cache, and `lake comparator` checks.
   The full check set requires Linux, Git, Ruby, Python 3, and `bwrap`
   (bubblewrap), which `lake comparator` uses to sandbox the build it judges.

   A successful run of the unmodified template deliberately includes the
   statement-surface `sorry` warning. It demonstrates the wiring, not a result.

## Important boundaries

A green build or Comparator check establishes only that Lean accepts the
project and that the recorded solution proves the recorded statement using the
permitted axioms. It does not establish mathematical significance, fidelity to
a source, novelty, or peer review.

Keep `Challenge.lean` ordinary and readable. Definitions needed by the
statement must have precise mathematical meanings and docstrings. Keep its
imports to Lean core and Mathlib wherever possible, so that a reader audits the
statement against definitions they already know. Dependencies used only by the
proof may be arbitrary pinned Git dependencies. The root licence covers this
repository snapshot only; cited papers, reused formalizations, and dependencies
retain their own licences.

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
