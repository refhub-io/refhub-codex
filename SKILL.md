---
name: refhub-codex
description: Use when Codex needs to operate RefHub through its public API or CLI: search/read vaults, manage vault items, import DOI/BibTeX/URL records, export vaults, audit vault activity, enrich metadata through Semantic Scholar, or upload item PDFs including large API-key resumable Google Drive uploads.
---

# RefHub Codex

Use the `refhub` CLI whenever it is available. It is the execution layer for RefHub API-key workflows and keeps authentication, request formatting, and errors consistent.

## Quick Start

1. Check for the CLI:

```bash
which refhub
refhub --help
```

2. Use `REFHUB_API_KEY` from the environment. A one-off `--api-key` flag may override it.
3. Keep the key out of chat history. Prefer loading `REFHUB_API_KEY` from a local shell env file or wrapper script before starting Codex.
4. Resolve vault IDs with `refhub vaults list` before operating on a vault. Do not infer IDs from names.
5. Prefer JSON output for downstream reasoning. Use `--table` only for human-facing summaries.
6. For unsupported account setup tasks, ask the user to use the RefHub web app.

## Authentication Rules

Normal Codex workflows use RefHub API keys:

```text
Authorization: Bearer rhk_<publicId>_<secret>
```

API keys cover vault, item, tag, relation, import, search, export, vault audit, Semantic Scholar, and item PDF upload routes.

Do not send API keys to browser/session management routes. API-key lifecycle, Google Drive connect/disconnect, legacy publication-level PDF upload, and global audit are session-JWT or web-app workflows.

If credentials are missing, insufficient, or vault-restricted, stop and report the missing key/scope/vault access. Do not try adjacent routes to bypass scope.

## Common Workflows

- Search or read a vault: list vaults, resolve the vault ID, then use vault/item search commands.
- Add or update items: ensure the key has `vaults:write`; tag IDs must already exist.
- Delete a vault or item: warn that deletion is hard delete/no undo, and require explicit user confirmation before proceeding.
- Import records: use DOI, BibTeX, or URL import commands when the vault is known and writable.
- Export a vault: require `vaults:export`; export JSON or BibTeX as requested.
- Audit a vault: use vault-scoped audit with an API key. Global audit is not an API-key workflow.
- Enrich metadata: use `refhub discover` and `refhub enrich`; respect Semantic Scholar rate limits and avoid overwriting user-provided fields.
- Upload item PDFs: use `refhub pdf upload --vault <vaultId> --item <itemId> --file <path.pdf>`.

## PDF Upload Contract

Small item PDFs use the raw API-key route:

```text
POST /api/v1/vaults/:vaultId/items/:itemId/pdf
```

Large vault-item PDFs use the API-key resumable flow:

```text
POST /api/v1/vaults/:vaultId/items/:itemId/pdf/session
PUT <upload_url returned by Google Drive>
POST /api/v1/vaults/:vaultId/items/:itemId/pdf/complete
```

Browser/session upload routes stay under `/api/v1/google-drive/...`. Do not point Codex/API-key workflows at those routes.

## When More Detail Is Needed

Read `references/refhub-api-contract.md` before:

- calling HTTP routes directly instead of the CLI
- handling permission errors or auth-mode confusion
- implementing or reviewing RefHub agent integrations
- reasoning about large PDF upload behavior
- checking whether a workflow is supported by the current public API
