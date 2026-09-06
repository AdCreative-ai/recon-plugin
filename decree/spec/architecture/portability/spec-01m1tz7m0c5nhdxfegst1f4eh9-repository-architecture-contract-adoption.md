---
date: '2026-09-06'
governs:
- .architecture/contract.json
- .architecture/check.py
- .architecture/lychee.toml
- .github/workflows/architecture.yml
- docs/architecture/README.md
- docs/CLAUDE.md
- AGENTS.md
- CLAUDE.md
- README.md
- lychee.toml
id: SPEC-01M1TZ7M0C5NHDXFEGST1F4EH9
references:
- ADR-01KZ0ZK4WYVWRY0WJM2CZ7ZS8C
status: implemented
---

# SPEC-01M1TZ7M0C5NHDXFEGST1F4EH9 Repository Architecture Contract Adoption

## Overview

Adopt version 1 of the shared repository architecture contract for Recon without
changing the shipped plugin, runtime behavior, release version, host adapters,
Jira approval gates, or optional dossier-storage semantics. The accepted
portable multi-harness architecture remains the owning architectural decision;
this SPEC governs only the repository-local machine contract, concise human
view, read triggers, and continuous verification required by the shared
standard.

## Technical Design

Add `.architecture/contract.json` with the exact version-1 field set. Its stable
identity is `recon-plugin`, its kind is `plugin`, and its evidence paths point
to the canonical plugin manifest, host and pipeline contracts, control CLI,
optional dossier-store adapter, generated-adapter owner, and repository commit
gate. The repository has no package installation manifest, so
`dependencyChecks` is empty. No targeted forbidden-reference boundary is
introduced in this documentation-only rollout, so `boundaryChecks` is empty;
existing behavior tests remain the owners of runtime boundaries.

Add `docs/architecture/README.md` using the shared standard's exact ordered
headings. It distinguishes repository installation metadata, Python/shell
library imports, runtime CLI calls, file exchange, and test-only dependencies.
Detailed command schemas and workflow invariants remain links to their existing
owners in `recon/docs/` and Decree. The task-packet-store relationship is
documented only as an explicit optional `recon-report` delivery path. A future
Jira-to-task-packet producer extraction is identified as future work, not
current Recon behavior and not part of this rollout.

Vendor the shared architecture-standard owner's repository checker
byte-for-byte as `.architecture/check.py`; do not fork or edit its behavior.
Add an isolated GitHub Actions workflow that checks out the repository and runs
`python3 .architecture/check.py` on pushes and pull requests without accessing
the private shared owner. Add short architecture read triggers to `AGENTS.md`
and `README.md`, and register the new documentation directory in
`docs/CLAUDE.md` so the existing coherence rail retains complete role coverage.

Architecture documentation maintenance routes architecture questions and edits
through the local architecture page from both root agent entrypoints and the
repository-docs editor contract. That page progressively discloses exact local
source/decision links and shared architecture diagrams, the standard, and the
maintenance playbook. Existing global editor rules and generated-source guards
remain mandatory.

The architecture workflow additionally installs lychee v0.24.2 through the
pinned lychee action and requires offline local file and named-anchor checking
for README, AGENTS, root/docs CLAUDE, and the architecture page. Its scoped
configuration has no link exclusions. This gate makes no private external URL
reachability claim and preserves the existing repository link check. The
repository's online configuration enumerates only the four exact private
architecture-owner files and the linked maintenance anchor as authentication
exceptions; maintainers validate
these separately with authenticated access. Public links and all other GitHub
URLs keep their existing checks.

## Testing Strategy

Run `python3 .architecture/check.py` directly and compare the vendored checker
to the shared owner with `cmp`. Run generated-adapter drift validation, link and
coherence checks, Decree lint/progress, and the repository's single
`tools/pre-commit-check.sh` gate. Review the diff to confirm plugin manifests,
version mirrors, skills, scripts, runtime docs, and release files are unchanged.
Run the exact documented lychee command with v0.24.2 and demonstrate that a
missing local file or named anchor fails before restoring the valid link.

## Acceptance Criteria

- [x] `.architecture/contract.json` follows schema version 1, resolves every
      evidence path, and declares no nonexistent package-install dependency.
- [x] `docs/architecture/README.md` uses the required ordered headings and
      distinguishes installation, imports, CLI calls, file exchange, and tests.
- [x] The architecture page preserves existing runtime ownership and accurately
      limits task-packet-store to explicit optional dossier delivery.
- [x] `AGENTS.md`, `README.md`, and `docs/CLAUDE.md` contain the required short
      architecture pointers and role registration.
- [x] `.architecture/check.py` is byte-for-byte identical to the shared owner
      checker and `.github/workflows/architecture.yml` runs it on pushes and
      pull requests without central-repository access.
- [x] The vendored architecture check and the complete repository pre-commit
      rail pass with no runtime or release-version changes.

## Completed Outcome

On 2026-09-06, the repository architecture checker returned
`ARCHITECTURE_OK: recon-plugin`, and the vendored bytes matched private shared
owner commit `f9961a34ba3665597e8d1d367e295e88f5aa1fea` at SHA-256
`52524a78978b568a56ca611e0a99feb574185b311283b26790d18fe9c121225d`.
The staged repository pre-commit rail passed links, adapter and generated-view
drift, host and artifact contracts, 139 artifact verifier cases, 11 dossier
store contract groups, repository-only laboratory controls, activation,
registry mirrors, role coverage, invariant citations, and Decree lint. No file
under `recon/`, no plugin manifest, no release version mirror, and no changelog
changed.
