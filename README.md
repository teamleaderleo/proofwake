# Proofwake

**The evidence trail behind every revision.**

Proofwake is a local, content-minimised evidence index for software projects. It collects bounded observations from local commands, Git, GitHub, CI, browser reviews, deployments, domain tools, and optional AI-usage estimates, then organises them by repository and revision.

It answers practical evidence questions:

- What changed recently?
- Which revisions have convincing evidence?
- What is failing, stale, sparse, partial, or still unobserved?
- What recovered after failure?
- Which repository needs attention next, and which source observation supports that conclusion?

Proofwake observes and projects evidence. Source systems retain their own authority; coordination, assignment, approval, runner operation, deployment, and remediation live elsewhere. A passing receipt says what one declared producer observed under its declared conditions.

```text
SmolRunner runs it.
Renderprove sees and verifies it.
Domain tools measure it.
Proofwake remembers the evidence trail.
Stensibly coordinates what happens next.
```

For the detailed product and trust model, see [product direction](docs/product-direction.md) and [architecture](docs/architecture.md). [Roadmap](docs/roadmap.md) owns implementation chronology and evolving milestone status.

## Quick start

Requires Node.js 22 or newer.

```bash
npm install
node src/cli.js status
node src/cli.js status --json
npm run serve
```

The package exposes `proofwake` as the primary command and `shadowbill` as a compatibility alias.

A clean installation uses `~/.proofwake`. Existing `~/.shadowbill` storage remains supported under the current compatibility contract. `PROOFWAKE_*` configuration takes precedence over matching `SHADOWBILL_*` aliases, and an ambiguous pair of implicit old/new ledgers fails closed instead of choosing or combining them. `proofwake status` reports the active identity and paths without exposing secret values. See [naming migration](docs/naming-migration.md) for exact precedence, storage selection, legacy interfaces, and migration safety.

## Current evidence surfaces

Current main includes:

- strict, content-minimised local Git observations and a bounded compatibility fallback for repositories without canonical GitHub identity;
- signed GitHub webhook observations for supported push, merged pull-request, workflow-run, deployment-status, and published-release deliveries;
- an append-oriented local observation ledger with deterministic replay/idempotency behavior and rebuildable projections;
- deterministic repository, revision, activity, freshness, failure, and recovery views;
- bounded work-evaluation and review-finding observations with attributable runs, confidence, uncertainty, coverage, and immutable history;
- `proofwake evaluation` for one repository/task-class evidence view;
- read-only Proofwake fleet/repository/revision/evaluation projections over MCP;
- the original optional Shadowbill AI-usage estimate reports and dashboard.

Missing, stale, sparse, partial, unavailable, and excluded evidence stays visible in projections. Proofwake does not turn missing evidence into a score or passing evidence into universal correctness, approval, causal attribution, routing, or a developer/repository productivity ranking.

Detailed contracts live in [Git ingestion](docs/git-ingestion.md), [signed GitHub webhook ingestion](docs/github-webhook-ingestion.md), [evaluation projection](docs/evaluation-projection.md), [MCP reporting](docs/mcp-reporting.md), and [observation v1](docs/observation-v1.md). Legacy activity-report coexistence and duplicate-representation rules live in [activity report compatibility](docs/activity-report-compatibility.md).

## Local collection and reports

Start the loopback collector:

```bash
npm run serve
```

Load the unpacked browser collector from [`extension/`](extension/). `proofwake status` shows the active collector-token path; keep the token local and paste it into the extension popup.

Install local commit collection in a repository:

```bash
node src/cli.js hook install /path/to/repository
```

The Git path records bounded metadata and retained-code estimates while discarding source text after collection. Hook/repository-identity details live in [Git ingestion](docs/git-ingestion.md) and [Git collection security](docs/git-collection-security.md).

Read aggregate reports with:

```bash
node src/cli.js report
node src/cli.js report --days 30 --by-repository
node src/cli.js report --days 30 --json
```

Repository allocation uses the versioned `same-day-added-code-tokens` heuristic. Unallocated cost and coverage remain explicit; the model is correlation-based rather than causal attribution. See [repository allocation](docs/repository-allocation.md).

## Evaluation evidence

```bash
proofwake evaluation \
  --repo teamleaderleo/stensibly \
  --task-class oauth-client-lifecycle \
  --output json
```

The projection reports current marks/findings, immutable histories, coverage, exclusions, confidence, uncertainty, open findings, and limitations for the selected repository/task class. Conservative evidence sufficiency requires distinct target runs; multiple receipts for one target cannot create sample breadth. It remains a read-only evidence view with no assignment, routing, approval, merge, deployment, or global worker/reviewer/model score authority. See [evaluation projection](docs/evaluation-projection.md).

## Dashboard and diagnostics

With the collector running:

```text
http://127.0.0.1:7337/dashboard/
```

The local dashboard is served from the collector origin with its browser-facing privacy controls. Current behavior belongs to [dashboard](docs/dashboard.md); Host, CORS, loopback, and reverse-proxy rules belong to [HTTP security](docs/http-security.md).

Inspect an installation read-only with:

```bash
node src/cli.js doctor
node src/cli.js doctor --json
```

[Doctor](docs/doctor.md) owns ledger/readiness, lock/recovery, permission, token-configuration, pricing, timezone, and report-generation diagnostics.

## MCP

```bash
npm run mcp
```

Proofwake-native projection tools are read-only and use the same deterministic projection functions as the CLI. The disclosure boundary returns bounded evidence/projection metadata while excluding content-bearing paths, commands/output, logs, receipt bytes, prompts/responses, credentials, and environment values. Existing Shadowbill report tools remain compatibility interfaces; aggregate chat writes remain separately opt-in. See [MCP reporting](docs/mcp-reporting.md).

## GitHub webhooks and HTTP boundary

A configured webhook secret enables signed GitHub delivery ingestion. Signature verification is the provider-authority boundary; accepted observations keep reviewed scalar facts, repository/revision relationships, bounded provider identities, coverage, and digest-backed evidence references while discarding raw payload content and sensitive prose.

The collector is loopback-first, validates HTTP `Host` authority, and grants cross-origin access only to the authenticated browser-collector routes. Hosted webhook ingress requires an explicitly configured TLS reverse proxy and narrow allowed authority. See [GitHub webhook ingestion](docs/github-webhook-ingestion.md) and [HTTP security](docs/http-security.md).

## Durable local ledger

The append-oriented JSONL ledger is the durable observation source; repository, revision, activity, and evaluation views are derived and rebuildable. Local writers coordinate through a filesystem lock, preserve idempotency, sync accepted writes, and retain crash-truncated tail evidence in a recovery sidecar. Detailed locking, recovery, and file-permission behavior belongs to [ledger durability](docs/ledger-durability.md).

Historical Shadowbill rows and newer observation-v1 records can coexist under explicit compatibility readers without rewriting history or double-counting recognized duplicate representations. See [activity report compatibility](docs/activity-report-compatibility.md).

## Privacy boundary

Proofwake stores bounded metadata, hashes, timestamps, labels, counts, durations, statuses, tool/evidence identities, coverage, and reviewed evaluation facts.

Default exclusions include:

- prompts and responses;
- source patches, added source text, and arbitrary source content;
- arbitrary command output and logs;
- secrets and environment dumps;
- pull-request descriptions/comments and other unreviewed prose;
- raw provider payloads;
- raw review prose, raw evaluation receipt bytes, and content-bearing wrapper extensions where the narrower contracts exclude them.

Every new adapter should define exact schemas, maximum sizes, source/trust identity, disclosure classes, redaction/truncation, idempotency/conflict behavior, and degraded-mode behavior before broadening collection.

## Accuracy and authority

Proofwake records observations and evidence. Evidence keeps its producer identity, trust class, coverage, freshness, and limitations. Relationships come from source evidence; timestamp proximity alone does not create causality.

The optional Shadowbill module reports API-equivalent estimates under versioned assumptions. Provider-internal cache behavior, hidden reasoning tokens, internal tool traffic, context handling, routing, and exact provider cost remain unknown unless a source exposes them. The estimate module can be disabled without reducing Proofwake's revision-evidence role.

Use language such as “observed passing,” “evidence present,” “source coverage incomplete,” and “recovery observed.” [Product direction](docs/product-direction.md) owns the broader language and authority discipline.

## Compatibility and current direction

Proofwake is the primary product identity. Shadowbill command/environment/storage compatibility remains active under [naming migration](docs/naming-migration.md), and historical schemas/records keep their existing identities. Current SmolRunner/Glaeda presentation and compatibility work is tracked separately; this README keeps the current landed identities until that owner migration changes them.

For detailed milestone history and active/future lanes, use [roadmap](docs/roadmap.md). README stays focused on the current public contract.

## Development

```bash
npm test
```

The current project uses zero runtime dependencies.

## Licence

Apache-2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
