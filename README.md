# renovate-config

Renovate bot configuration preset @LetsMOD

Please see [official documentation](https://docs.renovatebot.com/config-presets/) for details.

## Defaults

- **Schedule:** weekends (`schedule:weekends`).
- **Rebasing:** `rebaseWhen: conflicted` and `updateNotScheduled: false`.
  - Renovate rebases a PR only when it has a merge conflict, and only touches branches inside the schedule.
  - By default it rebases every open PR whenever the base branch moves, and each rebase re-runs the PR workflows: in agents that cost about 600 billed Actions minutes on 2026-10-02 alone (INFRA-578/579).
  - In repositories whose branch protection requires up-to-date branches, a Renovate PR may need one "Update branch" click before merging.
