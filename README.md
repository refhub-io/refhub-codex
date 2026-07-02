# refhub-codex (archived)

> // deprecated — superseded by refhub-skill

This repo is archived. It shipped a bare `agents/openai.yaml` at the root with no `.codex-plugin/plugin.json`, so it was never actually installable as a real Codex plugin per Codex's plugin schema (developers.openai.com/codex/plugins).

Codex packaging now lives directly in [`refhub-skill`](https://github.com/refhub-io/refhub-skill), alongside its Claude Code plugin manifest, using the verified `.codex-plugin/plugin.json` + `.agents/plugins/marketplace.json` format:

```json
{
  "name": "refhub-skill",
  "source": { "source": "github", "repo": "refhub-io/refhub-skill" },
  "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
  "category": "Productivity"
}
```

add that entry to your Codex plugin marketplace configuration, or clone `refhub-skill` locally and point Codex at it (see its README for details).

No further changes will be made to this repository.
