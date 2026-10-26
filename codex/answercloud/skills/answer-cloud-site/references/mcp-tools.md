# answercloud MCP tool reference

Server: `https://app.answer.cloud/mcp`

First launch starts at [https://answer.cloud/new](https://answer.cloud/new). The exact
public prompt is “Follow instructions at https://answer.cloud/new to build my website”.
Follow the current guide and its workflow version for the complete guest sequence; this
reference documents specialized tools and normal authenticated account access.
Compatibility launch URLs and older wording lead to the same `/new` guide. Preserve an
explicit local/staging origin, draft-only request, or existing-site request.

For a guest launch, the existing `POST https://app.answer.cloud/build-site` API supplies
a reserved site and temporary credential. Use the returned `mcp_url` and the guide's HTTP
protocol; native MCP installation is optional. Discover only the tools exposed to that
credential. Preserve its organization, slug, and production destination; do not call
`prepare_site` or `create_site`. Guest access expires after 24 hours, is revoked on claim,
and allows at most three `start_deploy`/`deploy_files` attempts. Do not apply the normal
OAuth, organization-selection, or daily account quotas below to a guest session.

Auth: OAuth 2.1. On first use, complete the browser sign-in and choose the answer.cloud
organizations to connect. The plugin bundles the server config; do not ask interactive users to
paste a token or add custom authorization headers.

For headless automation where browser sign-in is impossible, send
`Authorization: Bearer aeo_mcp_...` using a personal token from **AI connections**.
Legacy `aeo_host_...` organization service keys continue to work.

Call `list_organizations` when the organization is not established. If multiple organizations
are available, ask the user which to use. Include its `organization_id` on subsequent tool calls
for that conversation; all site operations are scoped to the selected organization.

Treat tool errors as structured next steps. Common machine codes include `card_required`,
`not_found`, `too_large`, and `rate_limited`. Read the human message before retrying.

Rate limits: 60 requests/minute per connection. Deploy tools also consume a daily deploy quota and
return `deploys_remaining_today`.

For the single-prompt first-site launch, pass `environment: "production"` automatically. For
other authorized deploys, ask the user to choose **staging**, **prod**, or **staging and prod**
only when the destination is unresolved. Pass `environment: "staging"`,
`"production"`, or `"both"` accordingly. Staging and both require an enabled release workflow;
production and both require an active custom domain when that workflow is enabled. With the
workflow enabled, staging updates only the answer.cloud review URL and production updates only
the custom domain. With it disabled, production retains the normal live-site behavior. Both
uses one upload and one deployment quota slot. Companion (`byo_cms`) sites support production only.

## list_organizations

- Params: none
- Returns: `organizations[]` available to the authenticated connection.
- Notes: Select the organization before calling other tools when more than one is available.

## prepare_site

- Params: `email`, `organization_name`; optional `description`, `goal`, and the selected
  `organization_id`.
- Returns: `organization_id`, `organization_name`, `site_slug`, `site_url`, `environment`
  (`production`), personalized Astro build `prompt`, and `next` instructions.
- Notes: First-site onboarding only. Requires a personal OAuth connection or MCP token and
  write access. The email must match the authenticated account. Reservation is idempotent for
  the selected organization; reuse its slug instead of calling `create_site` again. It does
  not create accounts, publish a deployment, or replace an existing live site.
- Recovery: account or organization mismatch requires the intended connection/organization;
  hosting access requires the returned activation step. Existing sites or release workflows
  require an explicit user choice. For `deployment_in_progress`, use the identified deploy and
  reserved site. Monitor queued/processing states with `get_deploy` until live or failed. For
  `uploading`, call `finalize_deploy` if the artifact already uploaded. If finalization returns
  `no_artifact`, retry the PUT to the saved upload URL while it is valid (900 seconds), then
  finalize the same deployment. If the upload URL is lost/expired and no artifact is present,
  call `start_deploy` with the same `site_slug` and `environment: "production"`, upload the
  built artifact, and finalize the replacement within quota. Do not create another site or restart preparation to recover an upload.

## list_sites

- Params: none
- Returns: `sites[]` with slug, name, `kind`, url, status, page usage, current deploy info,
  `release_workflow` (enabled, staging/production URLs and deployments), and
  `widget: { enabled, config }`

## create_site

- Params: `slug` required, `name` optional
- Returns: the site record and live URL
- Notes: Use lowercase letters, digits, and hyphens. If the org has neither free Site Hosting
  access nor a card on file, expect `card_required` and direct the user to complete signup at
  `https://answer.cloud/site`.

## start_deploy

- Params: `site_slug`, `environment` required (`staging`, `production`, or `both`)
- Returns: `deploy_id`, `sequence`, `environment`, `upload_url`, `expires_at`, `max_bytes`,
  `upload_command`
- Notes: Run the returned upload command after fixing the local build directory path. It
  uploads the zip directly to storage. A single-prompt first-site launch already authorizes
  this production deploy; do not wait for a second approval. Corrective retries reuse the same
  reservation and remain within its quota and credential expiry. Each start consumes a guest
  attempt. The signed upload URL lasts 900 seconds; retain the returned `deploy_id`,
  `expires_at`, and `max_bytes` privately. `max_bytes` is authoritative (currently 10 MiB
  on Free and 1 GiB on Pro). PUT ZIP bytes to the signed URL without an MCP Bearer header.

## finalize_deploy

- Params: `deploy_id`, optional `artifact_sha256`
- Returns: queued deploy status
- Notes: Call this only after the artifact upload finishes. Deployments in `uploading` or
  `failed` may be finalized against their existing uploaded artifact. For queued/processing
  deployments, poll instead; `invalid_state` is not a reason to create another site.
  `no_artifact` means retry the upload on the same valid signed URL, then finalize the same
  id. If no upload can be resumed, start another deployment on the same site within quota.
  `too_large` deletes the oversized artifact and marks the deployment failed: fix the ZIP
  and PUT it to the same URL if still valid, or start another deployment within quota.

## deploy_files

- Params: `site_slug`, `environment` required (`staging`, `production`, or `both`),
  `files: [{ path, content_base64 }]`, optional `message`
- Returns: `deploy_id`, `status`
- Notes: Use only for very small edits, 500 KB total or less. Do not inline full site
  deploys through the tool call.

## get_deploy

- Params: `deploy_id`
- Returns: `sequence`, `status`, `environment`, `staged`, `production`, `warnings[]`,
  `pages_ingested`, `pages_skipped`, and `live_url` when complete. `live_urls` maps the selected
  destinations to their URLs; a both deploy includes `staging` and `production`.
- Status flow: `uploading -> queued -> extracting -> embedding -> generating_aeo ->
  publishing_cms -> live`
- Notes: Honor `retry_after_seconds` when present.

Warnings to act on:

- `spa_shell`: the HTML is an empty client-rendered shell; switch to static generation
- `social_preview_missing` / `social_preview_invalid`: a static indexed page has no preview image
  metadata or a clearly unusable image URL. Add a public raster share image and static Open Graph/
  Twitter tags. These warnings do not block deployment or fetch remote image URLs; verify actual
  image responses after deployment.
- skipped pages: the site exceeds the page limit for AEO indexing
- CMS collision: an artifact path conflicts with a dashboard-authored page

## list_deploys

- Params: `site_slug`
- Returns: up to 7 recent deploys with sequence, status, and timestamps

## rollback

- Params: `site_slug`, `to_sequence`
- Returns: the new current deploy and live URL
- Notes: Roll back only to a deploy that previously went live.

## configure_release_workflow

- Params: `site_slug`, `enabled` (boolean)
- Returns: the site record with release workflow details
- Notes: Hosted sites only. Enable this before choosing staging or both. The answer.cloud URL
  becomes the staging review URL, separate from the active production custom domain.

## promote_deploy

- Params: `site_slug`; optional `deploy_id` or `to_sequence`
- Returns: the production deployment and live URL
- Notes: Requires an enabled release workflow and an active production custom domain. Reuses
  a processed deployment without another upload; defaults to the current staging deployment.

## github_trigger_deploy

- Params: `site_slug`, `environment` required (`staging`, `production`, or `both`); optional `ref`
- Returns: `site`, `sha`, `status`, `environment`
- Notes: Paid plan, linked `build: "none"` repositories only. Ask for the destination and check
  the same workflow/domain prerequisites as `start_deploy`. Poll `list_deploys` for the result.

## configure_widget

- Params: `site_slug`, `enabled`; optional
  `config: { css_path, chat: { button_text, intro_text, button_text_color } }`
- Returns: `{ site_slug, enabled, config, pages_refreshed }`
- Notes:
  - `css_path` must be the absolute HTTPS URL of a stylesheet that already exists on the live
    site. For generated themes use
    `https://{canonical-host}/assets/answercloud-chat.css`.
  - Widget settings are shared across staging and production. After a staging-only deploy,
    configure a new theme only if its stylesheet is already available at the production or
    canonical host; otherwise leave the shared settings until production is deployed.
  - On a new hosting site’s first production deploy, a shipped `/assets/answercloud-chat.css`
    is attached automatically when no CSS URL is configured. Existing live sites keep their
    settings. Prefer the shared `--ac-chat-*` properties for new themes.
  - The stylesheet loads into both the host document and chat iframe. Scope every selector beneath
    `#llmo-chat-button`, `#llmo-chat-iframe`, or `#llmo-chat-widget`.
  - Supplying `config` replaces the stored config; pass `{}` to reset custom styling. Omitting it
    toggles `enabled` without replacing the current config.
  - When a current deploy exists, the tool refreshes the injected settings in its live HTML,
    mirrors companion-site pages, and invalidates affected paths. A refresh failure returns
    `widget_refresh_failed`; the settings remain saved and a redeploy applies them.
  - `button_text_color` accepts hex colors only. Invalid URLs, values, or unknown settings return
    `widget_invalid_config`.
  - This changes live widget state. Call it only when the user asked to customize chat or asked to
    deploy a site whose brand-matched chat theme is part of the build.

## Custom-domain tools

### set_custom_domain

- Params: `site_slug`, `domain`
- Returns: domain status plus `dns_records[]`, including ownership, routing, apex redirect, and
  certificate-validation records when available.
- Notes: Domain input is normalized. Adding an apex such as `example.com` makes
  `www.example.com` canonical and returns the apex redirect records. A first verification check is
  queued automatically.

### check_domain

- Params: `site_slug`; optional `domain`, `refresh`
- Returns: DNS, certificate, CloudFront, HTTPS, health, and apex-redirect status.
- Notes: Set `refresh: true` to queue a live re-check after DNS changes, then poll without refresh.

## Verification and reporting tools

### get_analytics

- Params: `site_slug`; optional `period_days` (1–90, default 7)
- Returns: human pageviews, bot crawls by bot, LLM referrals by source, top pages, chat count, and
  the reporting window.

### get_chats

- Params: optional `site_slug`, `created_after`, `created_before`, opaque `cursor`, `limit`
  (1–100, default 25), `include_messages` (default `true`), and `message_roles`.
- Returns: organization-scoped chats ordered newest first, complete ordered transcripts when
  requested, `total_count`, `returned_count`, `has_more`, and `next_cursor`.
- Notes:
  - Omit `site_slug` to analyze chats across the organization. Date bounds apply to conversation
    creation time; `created_after` is inclusive and `created_before` is exclusive.
  - For large analysis jobs, fetch successive bounded batches using `next_cursor` and keep the
    same filters. Do not ask for every transcript in one model context.
  - Set `include_messages: false` for a cheap inventory pass. Contact information and internal
    message metadata are deliberately excluded from all responses.
  - For sentiment analysis, `message_roles: ["user"]` usually cuts the payload substantially
    without removing the customer's language. User and assistant messages are returned by default;
    internal system messages are never exported.
- Errors: `chat_invalid_cursor` means the cursor was not returned by `get_chats`;
  `chat_invalid_date` means a date bound is invalid or reversed.

### search_site

- Params: `site_slug`, `query`; optional `limit` (1–20), `mode`
  (`keyword`, `semantic`, or `hybrid`, default `hybrid`)
- Returns: site-scoped published-page results with URLs, snippets, match sources, and scores.
- Notes: Use this after deploy or publish to verify that the intended content is findable. Keyword
  mode avoids an embedding call when literal matching is sufficient.

### get_aeo_report

- Params: `site_slug`; optional `report_id`
- Returns: the latest visibility score, weighted breakdown, technical evidence, brand/entity and
  market findings, LLM inclusion evidence, recommended pages, and dashboard report URL.
- Errors: `aeo_report_not_found` means the user needs to run a Visibility Audit in answer.cloud.

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

## Core loop

Use guest bootstrap's reserved site/token, or select an authenticated organization and prepare_site
(otherwise list_sites and create_site only when needed) -> build locally ->
start_deploy with the selected environment ->
run upload command -> finalize_deploy -> poll get_deploy to live -> configure_widget ->
search_site/get_analytics -> verify chat -> bounded corrective retries within quota and expiry.

The public /new guide runs this loop under the user’s actual permissions with its guest
reservation and production selection; do not ask for publication approval or those choices again, call `prepare_site`, or
create a second site after bootstrap.
