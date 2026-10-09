# Automated review overlay — paperclip-github-plugin

Guidance for automated reviews of PRs in this repository.

- **Verify doc claims against code.** README, AGENTS.md, docs/, and .obvious/ statements must match the actual sources: versions against `package.json` (`engines`, `packageManager`), CI steps against `.github/workflows/ci.yml`, tool counts and file maps against `src/` and `tests/`. Reject stale version claims and phantom files.
- **Require test evidence for `src/` or `tests/` changes.** A PR that touches behavior must show the local CI-equivalent results on its final head (`pnpm install --frozen-lockfile && pnpm typecheck && pnpm test && pnpm build`) with test counts and the head SHA. There is no remote CI on this fork — an unverified claim of passing checks is a blocker, not a formality.
- **Flag scope creep beyond the unit.** Docs/orientation/QA-config PRs must stay inside their declared file set. A docs PR editing `src/`, `package.json`, lockfiles, or workflows is out of scope and should be split.
- **Respect intentional upstream references.** `package.json` `repository`/`homepage`/`bugs` and the manifest `author` deliberately point at the upstream origin (`alvarosanchez/paperclip-github-plugin`); do not flag them as stale identity metadata.
