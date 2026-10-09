# Autobuild orientation — paperclip-github-plugin

**What this is:** a Paperclip plugin that synchronizes GitHub issue and pull-request signals into Paperclip projects — two-way issue/PR state sync, remote actions, and a KPI contract. It runs as a plugin worker inside a Paperclip host; it is not a standalone app. This fork tracks upstream `alvarosanchez/paperclip-github-plugin`; repository metadata points at the fork.

**Who uses it:** Paperclip workspace owners who connect GitHub repositories through the plugin UI.

**Highest-priority constraints for automated work:**

- Prove every change with the full local CI-equivalent before reporting done: `pnpm install --frozen-lockfile && pnpm typecheck && pnpm test && pnpm build` (Node >=24.21, pnpm 12). This fork has GitHub Actions disabled, so the local run is the only quality gate — never report green without it.
- `pnpm-workspace.yaml` pins supply-chain policy (`allowBuilds`, `minimumReleaseAgeExclude`). The effective 24h minimum-release-age behavior must stay intact: never work around it by editing policy files or hand-editing lockfile integrity hashes.
- The SDK specifier in `package.json`, `tests/build-script.spec.mjs`, `scripts/e2e` defaults, and `pnpm-workspace.yaml` form one coordinated release triplet — bump them together or not at all.
- `pnpm test:e2e` and `verify:manual` target a live Paperclip host and cannot run here; never claim them as proof.

Details: `AGENTS.md` (commands, layout), `.obvious/orientation.md` (codebase map), `docs/qa.md` (QA process), `.obvious/review/overlay.md` (review guidance).
