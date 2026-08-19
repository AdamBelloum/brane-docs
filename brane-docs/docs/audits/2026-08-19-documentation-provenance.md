# Documentation provenance classification — 2026-08-19

**Deployment baseline:** `brane-deployment` `main@369392b991e0c3290739077d0ad071b5ce3f76bb`
**Deployment image baseline:** `3.0.0-nightly_fdbbd6c2`

## Current operational documentation

The role guides, architecture material, and frontend documentation are retained as
documentation targets for the current baseline, but their commands and topology
claims require the verification and correction work scheduled in later phases.

## Historical or reference material

- The Brane specification pages are locally maintained conceptual/reference
  material. No authoritative upstream implementation revision is recorded.
- Tutorials referring to Brane `2.0.0` are historical/reference material and
  must not be presented as current command guides.
- The UMC Utrecht tutorial refers to generic Brane `3.0.0`, not the pinned
  `3.0.0-nightly_fdbbd6c2` baseline, and requires verification.
- Introductory, quick-start, FAQ, and BraneScript-reference pages require
  command/source verification before being treated as current operations guides.

## Content defects to resolve

- Add or correct H1 titles in User Guide pages 01 and 02.
- Correct the duplicated/misnamed title in `brane-spec/08-extensibility.md`.
- Review `brane-tutorials/03-hello-world-part2.md` as likely misnamed or
  duplicated historical content.
- Resolve navigation status for the two unlinked tutorial pages and
  `brane-user-guide/08-brane-tools.md`.
