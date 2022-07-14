# General Development Setup

These are our recommendations.  If you know what you're doing, feel free to
deviate from them.

1. Install the GitHub CLI:
   1. Outside Google: https://cli.github.com/
   1. Inside Google: http://go/gh-cli

1. **(Maintainers Only)** Install the
   [Squashed Merge Message browser extension](https://github.com/zachwhaley/squashed-merge-message#install).
   For more details, see the [Maintenance docs](../maintenance/#pr-process).

1. Log in:
   ```sh
   gh auth login
   ```

1. *(Optional)*: Add these useful aliases to your GitHub CLI config:
   ```sh
   # Rebase your current branch on the upstream main with "gh sync"
   gh alias set --shell sync 'git pull upstream main --rebase && git push -f'

   # Sync the main branch (local and remote fork) with "gh syncmain"
   # WARNING: This is destructive to any changes in "main", so do your work in
   # a branch!
   gh alias set --shell syncmain 'git fetch upstream main && git push -f origin upstream/main:main && git push -f . upstream/main:main'
   ```

1. Fork and clone the repository you are working on:
   ```sh
   # This will clone the repo with "origin" as your fork, and "upstream" as the
   # upstream remote.
   gh repo fork shaka-project/shaka-player --clone
   ```

> :pencil: **TODO**: Add links to per-project development setup docs
