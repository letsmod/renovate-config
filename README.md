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
- **Security updates:** Renovate opens them from the repository's Dependabot alerts, right away instead of on the schedule, with an extra `security` label.
  - Dependabot alerts stay on in every repository, but Dependabot's own security update PRs are off org-wide, so the two bots don't open a PR for the same fix (INFRA-791). Both settings live in `infrastructure/common/github/repositories.tf`.
- **Lock file maintenance:** once a week, on the weekend, Renovate opens one PR per repository that regenerates its lock files (`package-lock.json`, `uv.lock` and others).
  - This is what brings in fixed versions of indirect dependencies (packages pulled in by other packages), which security update PRs don't touch.
  - The weekend schedule is set again under `lockFileMaintenance`, because its built-in default (early Monday) would override the global one.

## Private `@letsmod` npm packages

The `@letsmod/*` packages (dcl-kit: `dcl-core`, `dcl-linter`, `dcl-runtime`, `dcl-transpiler`; `pimcore-link`) are published to GitHub Packages, which Renovate cannot read on its own.

- **Registry:** `npmrc` maps the `@letsmod` scope to `https://npm.pkg.github.com`. `npmrcMerge: true` adds it to a repository's own `.npmrc` instead of replacing it.
- **Credential:** the `npm.pkg.github.com` host rule uses the org-level Mend secret `LETSMOD_GH_PACKAGES_TOKEN`: a classic personal access token with only `read:packages`, owned by the `letsmod-packages-reader` machine account. The Mend app's own token is not accepted by the GitHub Packages npm registry.
- **Package access:** a package inherits access from the repository that publishes it, so the account can read only the repositories granted in `infrastructure/common/github/machine-users.tf` (`dcl-kit`, `pimcore-link`). A new private package needs its repository added there first.
- **Where the secret lives:** [developer.mend.io](https://developer.mend.io) → `letsmod` → Settings → Credentials. Rotate it there before the token expires; no config change is needed.

Without this, Renovate reports `Failed to look up npm package @letsmod/...` on the Dependency Dashboard and cannot regenerate `package-lock.json` for updates that change `package.json` (INFRA-669).
