# renovate-config

Renovate bot configuration preset @LetsMOD

Please see [official documentation](https://docs.renovatebot.com/config-presets/) for details.

## Defaults

- **Schedule:** weekends (`schedule:weekends`).
- **Rebasing:** `rebaseWhen: conflicted` and `updateNotScheduled: false`.
  - Renovate rebases a PR only when it has a merge conflict, and only touches branches inside the schedule.
  - By default it rebases every open PR whenever the base branch moves, and each rebase re-runs the PR workflows: in agents that cost about 600 billed Actions minutes on 2026-10-02 alone (INFRA-578/579).
  - In repositories whose branch protection requires up-to-date branches, a Renovate PR may need one "Update branch" click before merging.
- **Python minor versions:** an update to a new Python minor version (for example 3.14 → 3.15) waits under "Pending Approval" on the repository's Dependency Dashboard.
  - Tick its box there when the repository is ready, and Renovate opens the PR.
  - Patch updates (3.14.5 → 3.14.6) arrive as usual, in their own PR (`separateMinorPatch`). Without that split, Renovate would offer only the newest version as one minor update, and holding it would hold the patches too.

## Private `@letsmod` npm packages

The `@letsmod/*` packages (dcl-kit: `dcl-core`, `dcl-linter`, `dcl-runtime`, `dcl-transpiler`) are published to GitHub Packages, which Renovate cannot read on its own.

- **Registry:** `npmrc` maps the `@letsmod` scope to `https://npm.pkg.github.com`. `npmrcMerge: true` adds it to a repository's own `.npmrc` instead of replacing it.
- **Credential:** the `npm.pkg.github.com` host rule uses the org-level Mend secret `LETSMOD_GH_PACKAGES_TOKEN`: a classic personal access token with only `read:packages`, owned by a machine account that has read access to `letsmod/dcl-kit` only. The Mend app's own token is not accepted by the GitHub Packages npm registry.
- **Where the secret lives:** [developer.mend.io](https://developer.mend.io) → `letsmod` → Settings → Credentials. Rotate it there before the token expires; no config change is needed.

Without this, Renovate reports `Failed to look up npm package @letsmod/...` on the Dependency Dashboard and cannot regenerate `package-lock.json` for updates that change `package.json` (INFRA-669).
