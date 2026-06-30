# RefHub API Contract for Codex

Base URL: `https://refhub-api.netlify.app/api/v1`

## Execution Layer

Prefer `@refhub/cli`:

- `refhub --help`
- `refhub vaults --help`
- `refhub items --help`
- `refhub import --help`
- `refhub discover --help`
- `refhub enrich --help`
- `refhub pdf --help`

The CLI reads `REFHUB_API_KEY`. A `--api-key` flag may override it for one command.

CLI exit codes:

- `0`: success
- `1`: API error
- `2`: bad arguments
- `3`: auth error

## Auth Modes

Never mix API-key and session-JWT route families.

### API Key Routes

Use RefHub API keys for normal agent/Codex work:

```text
Authorization: Bearer rhk_<publicId>_<secret>
X-API-Key: rhk_<publicId>_<secret>
```

Scopes:

- `vaults:read`: list/read vaults, search, stats, changes, vault audit, Semantic Scholar
- `vaults:write`: add/update/delete items, tags, relations, import, item PDF upload
- `vaults:export`: export vault JSON or BibTeX
- `vaults:admin`: create/update/delete vaults, visibility, shares

Keys may be vault-restricted. Check the target vault is in scope before operating.

### Session-JWT / Web-App Routes

Use the RefHub web app, or ask the user for an explicit session-JWT workflow, for:

- API-key lifecycle under `/keys`
- Google Drive connect/disconnect under `/google-drive`
- global audit under `/audit`
- legacy publication PDF upload under `/publications/:publicationId/pdf`
- browser item PDF upload under `/google-drive/vaults/...`

If an API key is sent to management routes, the API may return `401 refhub_api_key_not_supported`. Do not retry with the same key.

## Data Routes

### Vaults

- `GET /vaults`
- `POST /vaults`
- `GET /vaults/:vaultId`
- `PATCH /vaults/:vaultId`
- `DELETE /vaults/:vaultId`
- `PATCH /vaults/:vaultId/visibility`
- `GET /vaults/:vaultId/shares`
- `POST /vaults/:vaultId/shares`
- `PATCH /vaults/:vaultId/shares/:shareId`
- `DELETE /vaults/:vaultId/shares/:shareId`

Delete is hard delete/no undo. Ask for explicit confirmation before using it.

### Items

- `GET /vaults/:vaultId/items`
- `POST /vaults/:vaultId/items`
- `PATCH /vaults/:vaultId/items/:itemId`
- `DELETE /vaults/:vaultId/items/:itemId`
- `POST /vaults/:vaultId/items/upsert`
- `POST /vaults/:vaultId/items/import-preview`

Item delete is hard delete/no undo. `tag_ids` on update replaces the full tag set. Tag IDs must already exist.

### Tags

- `GET /vaults/:vaultId/tags`
- `POST /vaults/:vaultId/tags`
- `PATCH /vaults/:vaultId/tags/:tagId`
- `DELETE /vaults/:vaultId/tags/:tagId`
- `POST /vaults/:vaultId/tags/attach`
- `POST /vaults/:vaultId/tags/detach`

### Relations

- `GET /vaults/:vaultId/relations`
- `POST /vaults/:vaultId/relations`
- `PATCH /vaults/:vaultId/relations/:relationId`
- `DELETE /vaults/:vaultId/relations/:relationId`

Relation creation is not idempotent. Check existing relations before creating duplicates.

### Import

- `POST /vaults/:vaultId/import/doi`
- `POST /vaults/:vaultId/import/bibtex`
- `POST /vaults/:vaultId/import/url`

DOI import depends on server-side Semantic Scholar configuration. Surface existing item IDs on conflicts.

### Search, Stats, Sync, Export, Audit

- `GET /vaults/:vaultId/search`
- `GET /vaults/:vaultId/stats`
- `GET /vaults/:vaultId/changes`
- `GET /vaults/:vaultId/export?format=json|bibtex`
- `GET /vaults/:vaultId/audit`

Search/list use canonical `per_page` and `tag`; compatibility aliases `limit` and `tag_id` may also work. DOI filtering is supported.

### Semantic Scholar

API-key routes:

- `POST /semantic-scholar/lookup`
- `POST /semantic-scholar/doi-metadata`
- `POST /semantic-scholar/search`
- `POST /semantic-scholar/recommendations`
- `POST /semantic-scholar/related`
- `POST /semantic-scholar/references`
- `POST /semantic-scholar/citations`
- `POST /semantic-scholar/cited-by`

Require `vaults:read`. Respect the 1 request/second per-user rate limit. Enrichment should patch only missing/blank fields unless the user explicitly asks to overwrite.

## PDF Uploads

Item PDF upload requires:

- API key with `vaults:write`
- target vault editor permission
- user's Google Drive already linked through the RefHub web UI

Small PDFs:

```text
POST /vaults/:vaultId/items/:itemId/pdf
Content-Type: application/pdf
```

Raw API uploads are capped at the smallest of:

- `REFHUB_API_MAX_BODY_BYTES`
- `GOOGLE_DRIVE_MAX_UPLOAD_BYTES`
- Netlify synchronous Function body ceiling, 6 MiB

Large vault-item PDFs:

```text
POST /vaults/:vaultId/items/:itemId/pdf/session
PUT <upload_url> with PDF bytes directly to Google Drive
POST /vaults/:vaultId/items/:itemId/pdf/complete
```

The complete call sends Drive `file_id` and optional `web_view_link`.

Browser/session JWT routes are separate:

```text
/api/v1/google-drive/vaults/:vaultId/items/:itemId/pdf
/api/v1/google-drive/vaults/:vaultId/items/:itemId/pdf/session
/api/v1/google-drive/vaults/:vaultId/items/:itemId/pdf/complete
```

Do not use those `/google-drive/...` routes for Codex/API-key workflows.

Publication-level PDF upload remains session-JWT/raw only; the large resumable API-key contract is vault-item scoped.

## Unsupported Workflows

Do not approximate these unless the API adds routes:

- vault archiving/unarchiving
- vault duplication/clone
- item soft-delete/restore
- item revision history
- item move/copy between vaults
- webhooks/event delivery
- bulk relation import
