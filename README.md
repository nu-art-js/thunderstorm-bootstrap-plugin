# Thunderstorm Bootstrap — Cursor Plugin

A [Cursor](https://cursor.com) plugin that scaffolds a new [Thunderstorm](https://github.com/nu-art-js/thunderstorm) monorepo by **cloning the official sample boilerplate**, then customizing it.

## What it does

The skill guides an AI agent to:

1. **Clone** [`nu-art-js/thunderstorm-sample`](https://github.com/nu-art-js/thunderstorm-sample) (maintained Thunderstorm 0.500.x template with BAI, Vite + Webpack apps, sample `core/` library).
2. **Re-point** `git remote` to your new repository.
3. **Customize** workspace name, Firebase project IDs, local ports, and optional extra capability packages.
4. **Drop** the unused frontend (`app/frontend` vs `app/frontend-vite`) and optionally the sample `core/` tree.

It does **not** recreate the monorepo file-by-file — that stays in sync with the template repo.

## Installation

Install from the [Cursor Marketplace](https://cursor.com/marketplace), or clone this repo and install the plugin from disk.

## Usage

- "new thunderstorm project"
- "bootstrap thunderstorm from sample"
- `/bootstrap-thunderstorm-project`

## License

Apache-2.0
