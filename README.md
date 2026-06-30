# refhub-codex

Codex-native RefHub skill packaging.

This repo keeps RefHub's Codex integration separate from the Claude Code plugin marketplace. The skill uses `@refhub/cli` as its execution layer and describes the current RefHub public API contract for agent workflows.

## Install

Install the RefHub CLI first:

```sh
npm install -g @refhub/cli
```

Then install or copy this skill into Codex's skills directory:

```sh
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R refhub-codex "${CODEX_HOME:-$HOME/.codex}/skills/refhub-codex"
```

Set an API key for normal data workflows:

```sh
export REFHUB_API_KEY="rhk_..."
```

Do not paste live API keys into Codex chat. Load them from the shell environment instead.

## Secret Injection

Recommended pattern: keep the key in a local env file and launch Codex through a wrapper.

```sh
mkdir -p ~/.config/refhub
cat > ~/.config/refhub/env <<'EOF'
export REFHUB_API_KEY='rhk_REPLACE_ME'
EOF
chmod 600 ~/.config/refhub/env
```

Optional wrapper:

```sh
mkdir -p ~/.local/bin
cat > ~/.local/bin/codex-refhub <<'EOF'
#!/usr/bin/env bash
set -euo pipefail
source "$HOME/.config/refhub/env"
exec codex "$@"
EOF
chmod +x ~/.local/bin/codex-refhub
```

Then start Codex with `codex-refhub`. The skill and `@refhub/cli` pick up `REFHUB_API_KEY` from the environment, not from chat.

## Use

Invoke the skill from Codex:

```text
Use $refhub-codex to search my RefHub vault for papers about graph layout and summarize the top matches.
```

The skill covers API-key workflows for vaults, items, tags, relations, import, search, export, audit, Semantic Scholar discovery/enrichment, and item-scoped PDF uploads. Small PDFs use the raw item upload route. Larger vault-item PDFs use the API-key resumable Google Drive flow.

Account setup/admin workflows such as API-key creation, Google Drive connect/disconnect, and global audit remain RefHub web app or session-JWT workflows.

## Layout

```text
SKILL.md
agents/openai.yaml
references/refhub-api-contract.md
```

## Related

- [refhub-skill](https://github.com/refhub-io/refhub-skill): Claude Code skill/plugin packaging
- [refhub-claude](https://github.com/refhub-io/refhub-claude): Claude Code marketplace
- [@refhub/cli](https://www.npmjs.com/package/@refhub/cli): CLI execution layer
