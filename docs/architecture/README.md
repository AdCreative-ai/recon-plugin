# Recon repository architecture

This page is the concise repository boundary map. The shipped workflow and its
invariants remain owned by the [pipeline contract](../../recon/docs/pipeline.md),
and architectural decisions remain owned by Decree.

## Responsibility

This repository owns one evidence-first Jira reconnaissance plugin for Claude
Code and local Codex. It packages shared Agent Skills, deterministic shell and
Python rails, and generated host-native metadata. Recon decides whether a Jira
task is actionable, discovers the affected code and behavior, routes an
implementation brief through an optional governance adapter, and stops at
human gates; it does not implement the task.

The accepted
[portable multi-harness ADR](../../decree/adr/architecture/adr-01kz0zk4wyvwry0wjm2cz7zs8c-portable-multi-harness-recon-plugin-architecture.md)
owns the canonical-source and generated-adapter design. The
[pipeline contract](../../recon/docs/pipeline.md) owns stage behavior, artifact
schemas, and Jira delivery gates.

## Interfaces

| Relationship | Current interface | Source owner |
| --- | --- | --- |
| Installation | Claude and Codex marketplace metadata install the same `recon` plugin tree; Codex metadata is generated from the Claude manifest and skill frontmatter. | [Claude manifest](../../recon/.claude-plugin/plugin.json), [host contract](../../recon/docs/hosts.md) |
| Plugin entry | Eight `SKILL.md` entrypoints are registered by the canonical manifest; `recon-decree` is an internal routing adapter and the other skills expose their documented triggers. | [Claude manifest](../../recon/.claude-plugin/plugin.json), [`recon-triage`](../../recon/skills/recon-triage/SKILL.md), [`recon-report`](../../recon/skills/recon-report/SKILL.md) |
| CLI | Skills call repository shell rails. `reconctl.sh` is the host-neutral entrypoint for root resolution, host/surface detection, capability snapshots, preflight, and invocation rendering. | [`reconctl.sh`](../../recon/scripts/reconctl.sh), [host contract](../../recon/docs/hosts.md) |
| API | Triage reads Jira REST API v2 and uses authenticated `gh` queries for conflict evidence. Jira comment and attachment writes remain behind the separate explicit human delivery gate. | [`recon-triage`](../../recon/skills/recon-triage/SKILL.md), [pipeline invariants](../../recon/docs/pipeline.md#invariants-must-hold-in-every-run) |
| File | Stages exchange registered artifacts inside one current per-ticket workspace under `$RECON_ROOT`; archived `runs/` are never input. | [artifact registry](../../recon/docs/registry.yaml), [pipeline contract](../../recon/docs/pipeline.md) |
| CLI and file | Only an explicit store config makes `recon-report` call `store-dossier.sh`, which stages the current run and invokes task-packet-store. | [`recon-report`](../../recon/skills/recon-report/SKILL.md), [storage contract](../../recon/docs/storage.md) |

## Dependencies

Repository installation and runtime invocation are separate relationships:

- Repository installation declares no npm, Python, `@doruksahin/*`, or
  `@adcreative/*` package dependency; there is no package manifest in this
  repository. Native host marketplaces install the plugin files directly, so
  `.architecture/contract.json` has no `dependencyChecks` entry.
- The base local runtime preflight requires `bash`, `python3`, and `git`.
  Shipped Python rails import only Python standard-library modules. Triage adds
  `curl`, authenticated `gh`, Jira environment credentials, and network access.
  Recorded UI repro adds the `proofshot` and `agent-browser` CLIs. The Decree
  CLI is optional and loaded only when governance resolves to `decree`.
- The optional dossier-store adapter requires Node.js 20+, npm, `find`, `cp`,
  and a caller-supplied absolute store config. It invokes the exact
  `@doruksahin/task-packet-store` package pin with `npm exec`; that one-off CLI
  invocation is not a repository installation dependency.
- Repository-only verification uses Python, shell, `uv run decree`, and
  PyYAML-backed tooling. `lychee` is optional for external link checking.
  These do not become shipped plugin imports.

The executable host and storage prerequisites remain owned by the
[host contract](../../recon/docs/hosts.md) and
[storage contract](../../recon/docs/storage.md); this page does not duplicate
their command or credential schemas.

## Execution and storage

Recon executes in a local Claude Code or Codex shell. Shared skills orchestrate
the rails, while host-specific differences are limited to the capability
contract. The current release does not run on hosted ChatGPT, Codex Cloud, or a
central service.

Runtime state lives outside the target repository in
`$RECON_ROOT/<TICKET>/`. The current stage artifacts are file exchange between
skills and rails; prior runs are archived locally and are forbidden as inputs.
Repository-only replay and review laboratories do not ship with the plugin.

Task-packet-store is an optional output adapter for an already-rendered current
Recon run. A normal triage, discovery, repro, state, or report flow does not
consume a task packet and does not probe for a store. Filesystem and Google
Drive transport behavior remains owned by task-packet-store; Recon owns only
the explicit delivery adapter and its truthful receipt.

## Failure behavior

Required preflight failure stops before workspace mutation. Stage verifiers and
the artifact registry fail closed on malformed, missing, undeclared, or
incoherent evidence. Unsupported host capabilities are reported or rendered
locally rather than silently emulated.

Human approval remains mandatory before Jira mutation, and package approval is
not reused as Jira delivery approval. A requested task-packet save that fails
does not emit a persistence receipt and cannot be reported as render-only
success. The detailed stop and replacement rules are the
[pipeline invariants](../../recon/docs/pipeline.md#invariants-must-hold-in-every-run).

## Current implementation

The central architecture model should follow these source entrypoints:

- Packaging source: [`recon/.claude-plugin/plugin.json`](../../recon/.claude-plugin/plugin.json)
  registers the canonical skill directories; [`recon-triage`](../../recon/skills/recon-triage/SKILL.md)
  and [`recon-report`](../../recon/skills/recon-report/SKILL.md) expose the main
  input and optional-storage boundaries represented in the central model.
- Generated host metadata: [`tools/generate-adapters.py`](../../tools/generate-adapters.py)
  owns the Codex manifest, marketplace entry, and skill UI metadata.
- Runtime control: [`recon/scripts/reconctl.sh`](../../recon/scripts/reconctl.sh)
  and the [host contract](../../recon/docs/hosts.md).
- Workflow and file exchange: [`recon/docs/pipeline.md`](../../recon/docs/pipeline.md)
  and [`recon/docs/registry.yaml`](../../recon/docs/registry.yaml).
- Optional task-packet output: [`recon/scripts/store-dossier.sh`](../../recon/scripts/store-dossier.sh)
  and the [storage contract](../../recon/docs/storage.md).
- Repository verification: [`tools/pre-commit-check.sh`](../../tools/pre-commit-check.sh)
  composes links, coherence, universal controls, and Decree lint.

## Planned changes

A separate Jira producer that creates Jira task packets is planned at the
ecosystem level. It does not exist in this repository and is not current Recon
behavior: today triage reads Jira directly. When an operator explicitly
requests dossier storage after rendering, Recon saves the current dossier and
workspace as a `10-recon` stage run in the selected task-packet store; it does
not create the Jira task packet.

No Recon runtime, adapter, release, Jira-gate, or storage behavior change is
planned as part of this architecture-contract rollout.

## Decisions

- [ADR-01KZ0ZK4WYVWRY0WJM2CZ7ZS8C](../../decree/adr/architecture/adr-01kz0zk4wyvwry0wjm2cz7zs8c-portable-multi-harness-recon-plugin-architecture.md)
  owns the accepted portable multi-harness architecture and generated-adapter
  boundary.
- [SPEC-01M1SMA0CR7BCGE0821Z46YWYW](../../decree/spec/architecture/portability/spec-01m1sma0cr7bcge0821z46ywyw-recon-task-packet-store-delivery.md)
  owns the implemented optional task-packet-store delivery adapter.
- [SPEC-01M1TZ7M0C5NHDXFEGST1F4EH9](../../decree/spec/architecture/portability/spec-01m1tz7m0c5nhdxfegst1f4eh9-repository-architecture-contract-adoption.md)
  governs this repository's architecture-contract adoption and CI wiring.

## Verification

Pull requests and pushes run `python3 .architecture/check.py` from the vendored
repository copy. Maintainer verification also compares that file byte-for-byte
with the shared owner, runs `python3 tools/generate-adapters.py --check`, and
runs `bash tools/pre-commit-check.sh`.

The vendored checker is synchronized from private architecture-owner commit
`f9961a34ba3665597e8d1d367e295e88f5aa1fea`; its SHA-256 is
`52524a78978b568a56ca611e0a99feb574185b311283b26790d18fe9c121225d`.
