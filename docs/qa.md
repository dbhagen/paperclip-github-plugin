# QA guide — paperclip-github-plugin

How to verify this plugin locally, what each automated suite covers, and where those suites stop.

## Toolchain

The package requires Node `>=24.21.0` (`engines` in `package.json`) and pnpm `12.4.2` (`packageManager`). Fresh environments often default to Node 20, which fails the engines check, so pin the toolchain first (verified on Linux x64):

```bash
curl -fsSL https://nodejs.org/dist/v24.21.0/node-v24.21.0-linux-x64.tar.xz -o /tmp/node24.tar.xz
tar -xf /tmp/node24.tar.xz -C /home/user && mv /home/user/node-v24.21.0-linux-x64 /home/user/node24
export PATH=/home/user/node24/bin:$PATH
corepack enable && corepack prepare pnpm@12.4.2 --activate
```

The disposable Paperclip harnesses (e2e and manual verification) run `paperclipai@2026.831.1` under this Node line, avoiding Node 20's missing `node:sqlite` runtime module.

## Local verification pipeline (CI-equivalent)

From the repository root, the same steps `.github/workflows/ci.yml` runs:

```bash
pnpm install --frozen-lockfile
pnpm typecheck   # tsc --noEmit
pnpm test        # node --test tests/build-script.spec.mjs + tsx --test tests/*.spec.ts
pnpm build       # esbuild bundle of manifest, worker, and UI into dist/
```

All four steps are expected to pass on every proposed change. Baseline as of the October 2026 maintenance wave: typecheck clean, **370/370 tests pass** (365 in the `tsx --test` suite + 5 in the `node --test` build-script suite), build completes with `dist/manifest.js`, `dist/worker.js`, and `dist/ui/`.

## What each test file covers

| File | Runner | Coverage |
| --- | --- | --- |
| `tests/plugin.spec.ts` | `tsx --test` | The main behavior suite. Drives worker sync/status-routing logic (executor/reviewer handoffs, PR-linked routing, effective-state fingerprints, wake suppression) through the plugin's exported testing hooks. |
| `tests/git-branch-publisher.spec.ts` | `tsx --test` | Security invariants of branch publication: exact branch-tip SHA enforcement, checked-out branch requirement, base-branch and invalid-input rejection, token redaction on failure, remote readback matching. |
| `tests/issue-interactions.spec.ts` | `tsx --test` | Pure ledger logic: 30-day `[from,to)` window validation, metadata sanitizer allowlist, deterministic summary counts/transitions/repeats/reversals, bounded scan contract, cross-company rejection. |
| `tests/issue-interactions-integration.spec.ts` | `tsx --test` | Ledger behavior through worker tool paths: company-scoped summaries, sanitized attributed failure events, durable-intent persistence failures returning structured tool errors, content-addressed idempotency. |
| `tests/build-script.spec.mjs` | `node --test` | Repo invariants: build script fails clearly without `node_modules`, release harnesses default to the current Paperclip release, SDK dependency targets the current release, workflows let `packageManager` select pnpm, and the 2026.831 adoption boundary stays documented. |

Tests establish behavior through exported hooks and public contracts; they do not mirror internal call graphs. Prefer adding behavior-level cases over implementation-coupled assertions.

## e2e and manual verification

Two heavier harnesses exist for changes the fast suites cannot observe. Both boot a real, disposable Paperclip host and are not part of the CI pipeline.

- `pnpm test:e2e` — builds the plugin, installs Playwright Chromium, boots an isolated Paperclip `2026.831.1` instance, installs the plugin, and verifies the hosted settings page renders. Use when touching manifest contributions, UI mount behavior, the plugin installation flow, or the e2e harness itself.
- `pnpm verify:manual` — builds the plugin, boots a local-trusted Paperclip `2026.831.1` instance, seeds a `Dummy Company` with a mapped review project and a `CEO` agent (Codex local adapter, model `gpt-5.4`), installs the plugin, and opens the company dashboard without seeding KPI history. Use when a task benefits from visual inspection inside a real host.

Boundary: these harnesses install `paperclipai@2026.831.1` from npm and Playwright Chromium from the network, so they need outbound access and take minutes rather than seconds. They are the only checks that exercise a real host; the local CI pipeline does not.

## Fork boundary: no remote CI

This repository is the `dbhagen/paperclip-github-plugin` fork of `alvarosanchez/paperclip-github-plugin`. GitHub Actions are disabled on the fork: it has zero workflow runs, and enabling Actions returned HTTP 403 (the token lacks fork admin rights) — verified by the maintenance-wave lead on 2026-10-09. The badge in `README.md` will therefore not report status until Actions are enabled.

Consequence: **the local pipeline above is the acceptance gate.** Run it on the exact final head of any branch before requesting merge, and record the commands, results, and head SHA in the PR body.

## Cross-references

- `AGENTS.md` → *Verification* section: when to run which check, and the smallest relevant scope.
- `.obvious/orientation.md`: file-by-file codebase map for new agents.
