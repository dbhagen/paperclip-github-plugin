# Orientation — paperclip-github-plugin

A codebase map for agents new to this repository. Pair this with `AGENTS.md` (working rules) and `docs/qa.md` (verification guide).

## Purpose

`paperclip-github-plugin` ("GitHub Sync") is a Paperclip plugin that connects GitHub repositories to Paperclip projects: it imports open GitHub issues as top-level Paperclip issues, keeps them in sync (status, labels, descriptions, assignees), exposes GitHub workflow tools to Paperclip agents, and adds hosted UI surfaces (settings page, dashboard widgets, a project Pull Requests queue, an issue detail tab). This checkout is the `dbhagen/` fork of `alvarosanchez/paperclip-github-plugin`; the upstream author attribution in `src/manifest.ts` is intentional.

## Source map (`src/`)

Read order below follows dependency: contracts and helpers first, then the two large entry points.

| File | Role |
| --- | --- |
| `src/kpi-contract.ts` | Shared constants: plugin id, the company-metric (KPI) API route key/path, and the `pull_request_created` metric type. Imported by manifest, worker, and UI. |
| `src/github-repo.ts` | `parseRepositoryReference`: normalizes `owner/repo` slugs or GitHub URLs into a validated `{owner, repo, url}`. |
| `src/paperclip-health.ts` | Normalizes Paperclip host health responses and derives board-access policy (authenticated deployments require board access before sync). |
| `src/issue-interactions.ts` | Contract and pure logic for the issue interaction ledger: event shape/schema version, metadata sanitizer allowlist, 30-day window and scan-row caps, deterministic summary builder. |
| `src/github-agent-tools.ts` | `GITHUB_AGENT_TOOLS`: the 21 agent-facing `PluginToolDeclaration` schemas (issues, PRs, review threads, org projects, linking, asset upload, KPI summary) plus a declaration lookup. |
| `src/git-branch-publisher.ts` | `publishLocalBranchForPullRequest`: publishes an exact local branch tip to GitHub through an isolated temporary bare repo (no hooks, no arbitrary refspecs), verifying checked-out branch, branch tip, base ancestry, and remote SHA readback; redacts the token from git failures. |
| `src/manifest.ts` | The `PaperclipPluginManifestV1`: plugin id/display name, capabilities, per-minute scheduled sync job, tool declarations, and UI slots (project PR page + sidebar item, sync and KPI dashboard widgets, issue detail tab, toolbar buttons). Version resolves from `PLUGIN_VERSION` / `package.json` / `npm_package_version`. |
| `src/worker.ts` | The backend (~25k lines — search, do not read linearly). `definePlugin(...)` wiring for the full sync engine: GitHub REST via Octokit, secret-ref resolution, import/dedup, status routing with durable fingerprints and action journals, agent tool execution, KPI API route. Exports `plugin` (default), `__testing`, and `shouldStartWorkerHost`. |
| `src/ui/index.tsx` | The hosted UI (~15k lines — search, do not read linearly). All React surfaces: settings page, dashboard sync widget, KPI widget, project Pull Requests page and sidebar item, issue detail tab, toolbar buttons. Also exports pure resolvers (toolbar state, saved-token UI state, detail-tab state) that tests exercise directly. |

### `src/ui/` helper modules

| File | Role |
| --- | --- |
| `src/ui/http.ts` | Browser fetch helpers: URL building, JSON response parsing that detects HTML-instead-of-JSON auth pages, host health fetch, CLI auth poll URL resolution. |
| `src/ui/assignees.ts` | Normalizes company agent/user lists into assignee dropdown options (drops terminated agents). |
| `src/ui/host-secrets.ts` | Company secret helpers: list/create/rotate request builders, resolve-or-create, and the optional opt-in exposure of the GitHub token to host features as `GITHUB_TOKEN`. |
| `src/ui/plugin-config.ts` | Plugin config types and normalizers: `secret_ref` bindings (upgrades legacy bare secret-id rows), worker API base URL normalization, config merge. |
| `src/ui/plugin-installation.ts` | Resolves the installed plugin's id and settings-page href from host plugin list responses. |
| `src/ui/project-bindings.ts` | Discovers and filters existing Paperclip projects as sync candidates for a repository mapping (deduping by project/repo). |

## Test map

| File | Covers |
| --- | --- |
| `tests/plugin.spec.ts` | Main behavior suite (~32k lines): sync status routing, executor/reviewer/return handoffs, effective-state fingerprints and wake suppression, direct-PR and issue-linked-PR routing. Uses the worker's `__testing` hooks. |
| `tests/git-branch-publisher.spec.ts` | Branch publisher security invariants (exact SHA, checkout, ancestry, readback, token redaction). |
| `tests/issue-interactions.spec.ts` | Pure ledger logic: ranges, sanitizer, summary determinism and bounds. |
| `tests/issue-interactions-integration.spec.ts` | Ledger through worker tool paths: scoping, failure attribution, durable intent, idempotent content-addressed writes. |
| `tests/build-script.spec.mjs` | Repo invariants via `node --test`: build script behavior, release pinning, workflow/packageManager consistency, documented adoption boundary. |

Harnesses under `scripts/`: `build.mjs` (esbuild bundling into `dist/`), `e2e/run-paperclip-smoke.mjs` (hosted smoke test), `e2e/manual-paperclip-verify.mjs` (seeded local host for manual inspection). See `docs/qa.md` for boundaries.

## Verification commands

```bash
pnpm install --frozen-lockfile
pnpm typecheck
pnpm test
pnpm build
```

Scope-selective checks (`pnpm test:e2e`, `pnpm verify:manual`) are described in `docs/qa.md`; when to use each is in `AGENTS.md` → *Verification*.

## Gotchas

- **Node `>=24.21.0` and pnpm `12.4.2` are required.** Fresh sandboxes default to Node 20, which fails the engines check and lacks `node:sqlite`. Bootstrap steps are in `docs/qa.md`.
- **`dist/` is generated.** Never hand-edit; rebuild with `pnpm build`.
- **The fork has no remote CI.** GitHub Actions are disabled on `dbhagen/paperclip-github-plugin` (enable attempt returned HTTP 403); the local pipeline above is the acceptance gate. Record commands, results, and head SHA in PRs.
- **`worker.ts` and `ui/index.tsx` are huge** (~25k and ~15k lines). Use symbol search and the exported `__testing` hooks instead of reading top-to-bottom.
- **Source imports use explicit `.ts` extensions** (e.g. `./github-agent-tools.ts`); follow that convention.
- **Upstream references are intentional.** `package.json` `repository`/`homepage`/`bugs` and the manifest `author` point at the upstream origin; do not "fix" these as if they were stale identity metadata.
