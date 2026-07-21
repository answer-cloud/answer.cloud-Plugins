# answercloud MCP tool reference

Server: `https://answer.cloud/mcp`

Auth: send `Authorization: Bearer aeo_host_...`

Treat tool errors as structured next steps. Common machine codes include `card_required`,
`not_found`, `too_large`, and `rate_limited`. Read the human message before retrying.

Rate limits: 60 requests/minute per key. Deploy tools also consume a daily deploy quota and
return `deploys_remaining_today`.

## list_sites

- Params: none
- Returns: `sites[]` with slug, name, url, status, page usage, and current deploy info

## create_site

- Params: `slug` required, `name` optional
- Returns: the site record and live URL
- Notes: Use lowercase letters, digits, and hyphens. If the org has no card on file, expect
  `card_required` and direct the user to complete signup at `https://answer.cloud/site`.

## start_deploy

- Params: `site_slug`
- Returns: `deploy_id`, `sequence`, `upload_url`, `expires_at`, `max_bytes`,
  `upload_command`
- Notes: Run the returned upload command after fixing the local build directory path. It
  uploads the zip directly to storage.

## finalize_deploy

- Params: `deploy_id`, optional `artifact_sha256`
- Returns: queued deploy status
- Notes: Call this only after the artifact upload finishes.

## deploy_files

- Params: `site_slug`, `files: [{ path, content_base64 }]`, optional `message`
- Returns: `deploy_id`, `status`
- Notes: Use only for very small edits, 500 KB total or less. Do not inline full site
  deploys through the tool call.

## get_deploy

- Params: `deploy_id`
- Returns: `sequence`, `status`, `warnings[]`, `pages_ingested`, `pages_skipped`, and
  `live_url` when complete
- Status flow: `uploading -> queued -> extracting -> embedding -> generating_aeo ->
  publishing_cms -> live`
- Notes: Honor `retry_after_seconds` when present.

Warnings to act on:

- `spa_shell`: the HTML is an empty client-rendered shell; switch to static generation
- skipped pages: the site exceeds the page limit for AEO indexing
- CMS collision: an artifact path conflicts with a dashboard-authored page

## list_deploys

- Params: `site_slug`
- Returns: up to 7 recent deploys with sequence, status, and timestamps

## rollback

- Params: `site_slug`, `to_sequence`
- Returns: the new current deploy and live URL
- Notes: Roll back only to a deploy that previously went live.

## CMS tools

CMS tools are org- and site-scoped like deploy tools, but they do **not** consume the daily
deploy quota. They require the site to have been deployed at least once before publishing
(`cms_no_live_deploy` otherwise). CMS error codes: `cms_collection_not_found`,
`cms_entry_not_found`, `cms_template_missing`, `cms_template_invalid`, `cms_path_collision`,
`cms_no_live_deploy`, `cms_invalid_field`, `cms_publish_failed`. Each error message includes a
concrete next step.

### list_cms_collections

- Params: `site_slug`
- Returns: `collections[]` with `key`, `name`, `description`, `index_path`, `path_pattern`,
  `item_type`, `template_path`, `index_template_path`, `template_health: { show, index }`
  (`healthy`/`missing`), and `entries_count`/`published_count`/`draft_count`.

### upsert_cms_collection

- Params: `site_slug`, `key` (required); optional `name`, `description`, `index_path`,
  `path_pattern`, `template_path`/`index_template_path` or `templates: { show, index }`,
  `item_type`, `fields[]`, `sort: { field, direction }`
- Returns: the collection view. Invalid paths/fields return `cms_invalid_field`.

### list_cms_entries

- Params: `site_slug`, `collection_key`; optional `status` (`all`/`draft`/`published`/`archived`), `query`
- Returns: `entries[]` with `id`, `title`, `status`, `has_unpublished_changes`, `source_path`,
  `live_url`, `preview_url`, `updated_at`, `published_at`.

### get_cms_entry

- Params: `site_slug`, `collection_key`, `entry_id`
- Returns: entry summary plus `published_payload`, `draft_payload` (if any), and `aeo` status.

### upsert_cms_entry

- Params: `site_slug`, `collection_key`; optional `entry_id` (omit to create); content fields
  `title`, `slug`, `summary`, `content`, `published_at`, `metadata`; `save_mode` `draft` (default)
  or `publish`.
- Returns: entry detail (+ `next` guidance) for drafts, or the publish result for `publish`.
- Notes: only author content when the user asks; only `publish` after they approve.

### preview_cms_entry

- Params: `site_slug`, `collection_key`, `entry_id`
- Returns: `preview_url` (private in-answer.cloud preview of staged draft) and `template_status`.
  Does not return full HTML.

### publish_cms_entry

- Params: `site_slug`, `collection_key`, `entry_id`
- Returns: `entry`, `live_url`, `index_url`, `written_paths[]` (entry HTML/txt, index HTML/txt,
  `llms.txt`, `sitemap.xml`). Path collisions return `cms_path_collision`.

### archive_cms_entry

- Params: `site_slug`, `collection_key`, `entry_id`
- Returns: updated `entry` status and `changed_paths[]`. Re-renders the collection index.

### republish_cms

- Params: `site_slug`; optional `collection_key`
- Returns: refreshed `collections[]`. Re-renders all published entries and indexes into the
  current deploy — use after template changes.

## Optional site-ops tools

Some environments also expose tools such as `set_custom_domain`, `check_domain`,
`get_analytics`, `search_site`, or `configure_widget`. Use them for verification and site
operations when available, but do not block the core build-and-deploy loop on them.

## Core loop

list_sites -> create_site -> build locally -> start_deploy -> run upload command -> finalize_deploy -> poll get_deploy to live -> fix warnings -> redeploy until clean
