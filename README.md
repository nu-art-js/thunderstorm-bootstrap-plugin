# Thunderstorm Bootstrap — Cursor Plugin

A [Cursor](https://cursor.com) plugin that scaffolds a new [Thunderstorm](https://github.com/nu-art-js/thunderstorm) monorepo project from scratch.

## What it does

Provides a skill that walks an AI agent through creating a complete Thunderstorm (BAI) workspace:

- Collects project parameters (name, scope, ports, firebase projects, initial packages)
- Initializes a git repo with `_thunderstorm` as a submodule
- Generates all root config files (`.nvmrc`, `__package.json`, `bai-config.json`, tsconfig, eslint, firebase rules)
- Scaffolds app packages (vite frontend + firebase function backend)
- Scaffolds library packages (shared/frontend/backend per capability)
- Sets up `.gitignore`, `CLAUDE.md`, and `_docs/` structure

Based on Thunderstorm 0.500.x conventions.

## Installation

Install from the [Cursor Marketplace](https://cursor.com/marketplace), or clone this repo and point Cursor at it.

## Usage

In Cursor chat, ask the agent to create a new Thunderstorm project. The skill activates on phrases like:

- "new thunderstorm project"
- "bootstrap project"
- "create TS project from scratch"

Or invoke directly: `/bootstrap-thunderstorm-project`

## License

Apache-2.0
