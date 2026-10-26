# Chat widget verification and troubleshooting

Use this reference when a user asks to add, verify, or debug the answer.cloud chat/search widget.
The widget and its API are platform-owned; site authors supply answerable static content.

## Verification flow

1. Run `list_sites` and identify the exact site, kind, live URL, current deploy, and page usage.
2. Run `list_deploys`; confirm the intended deploy is `live`.
3. Open the live URL. Do not use the local Astro preview to judge widget injection.
4. Confirm the page contains substantive HTML and the injected widget launcher.
5. Open the widget and submit one short, non-sensitive question grounded in the page
   whenever live chat verification is in scope, including every first launch.
6. Confirm the response renders, relevant references use the site's canonical host, and the
   browser reports no failed chat request.

Do not add `https://app.answer.cloud/embed.js` to Astro source. The platform injects it before
`</body>` during publishing.

## Brand-matched styling workflow

For a site build or visual restyle, create a chat theme by default unless the user explicitly
requests the stock theme or disables chat. For a new unbranded site, begin with the official
answer.cloud style template's `public/assets/answercloud-chat.css`.

1. Read the site's global style sources and inspect representative rendered components. Extract
   the fonts, surfaces, text colors, accent and on-accent colors, border, focus ring, radii,
   shadow, and motion character actually in use. Do not invent a second brand system.
2. Write the theme to `public/assets/answercloud-chat.css`. Astro must emit it unchanged at
   `dist/assets/answercloud-chat.css`.
3. Scope every selector under `#llmo-chat-button`, `#llmo-chat-iframe`, or
   `#llmo-chat-widget`. The same URL loads into the customer's page and the iframe, so unscoped
   rules such as `:root`, `html`, `body`, `*`, `button`, `a`, or generic utility classes can
   damage the host site.
4. Exercise all meaningful states: launcher default/hover/focus, desktop/mobile frame, empty
   chat, user and AI messages, long Markdown, links and references, composer focus, send/close/
   clear controls, loader, errors, and reduced motion.
5. Build first and verify the asset exists. For a `byo_cms` artifact, update the package script
   so `assets/answercloud-chat.css` ships next to `assets/aeo-cms.css`.
6. A new hosting site’s first production deploy automatically attaches this asset when no
   CSS URL is configured. Existing deployed sites and explicit themes are preserved. Once an
   authorized deploy is live, inspect `list_sites`, verify the canonical HTTPS asset, and use
   `configure_widget` only when needed for theme or copy. Never point live configuration at
   localhost, a preview server, or a path that has not deployed yet.

The supported stable hooks are:

- Host page: `#llmo-chat-button`, `.llmo-chat-button-icon`,
  `.llmo-chat-button-input`, and `#llmo-chat-iframe`.
- Chat iframe: `#llmo-chat-widget`, `.llmo-chat-popup`, `.llmo-messages`,
  `.llmo-message.llmo-user`, `.llmo-message.llmo-ai`, `.llmo-markdown`,
  `.llmo-message-references`, `.composer-wrap`, `.composer`, `.chat_textarea`,
  `.chat_send`, `.llmo-close`, `.llmo-clear-chat`, `.llmo-loader`, and `.ai-icon`.
- Legacy theme variables on `#llmo-chat-widget` remain supported: `--bg`, `--ink`, `--ink-mid`, `--muted`,
  `--panel`, `--ring`, `--ring-soft`, `--brand`, `--accent`, `--accent-deep`,
  `--accent-soft`, `--accent-tint`, `--accent-tint-2`, `--accent-line`, `--radius`,
  `--ease-brand`, `--font-sans`, and `--font-mono`.

Use the shared properties below on all three roots. The launcher and iframe are separate
HTML documents, so declaring tokens only on the panel does not style the launcher. Replace
these neutral example values with the site's actual palette; any color is supported.

```css
#llmo-chat-button,
#llmo-chat-iframe,
#llmo-chat-widget {
  --ac-chat-font-family: "Site Sans", system-ui, sans-serif;
  --ac-chat-surface: #ffffff;
  --ac-chat-text: #1a1b1c;
  --ac-chat-secondary-text: #3a3d40;
  --ac-chat-muted: #667085;
  --ac-chat-panel: #f6f5f4;
  --ac-chat-border: #d9d9d9;
  --ac-chat-accent: #1a1b1c;
  --ac-chat-on-accent: #ffffff;
  --ac-chat-focus-ring: rgb(26 27 28 / 24%);
  --ac-chat-radius: 8px;
}

#llmo-chat-iframe {
  --ac-chat-frame-width: 560px;
  --ac-chat-frame-height: 640px;
  --ac-chat-frame-padding: 0px;
  --ac-chat-frame-shadow: 0 16px 48px rgb(26 27 28 / 16%);
}
```

Use `--ac-chat-icon-color` for the supplied Answer Cloud spark, `--ac-chat-link-color` for
answer links, `--ac-chat-user-surface` / `--ac-chat-assistant-surface` for message backgrounds,
and `--ac-chat-send-background` / `--ac-chat-send-color` for the send button. Other optional
properties control `launcher-text`, `launcher-radius`, `launcher-width`, `launcher-shadow`,
`mobile-width`, `mobile-height`, `offset-bottom`, `offset-right`, `message-radius`, `code-surface`,
`input-surface`, `control-surface`, `send-shadow`, and `loader-shine` (all prefixed `--ac-chat-`).
Frame geometry uses these properties directly, so token changes do not need `!important`.

Keep custom fonts available inside the iframe as well as the host page. Theme loading is
nonblocking, with a brief bounded first-paint hold for fast stylesheets. Slow or failed themes
fall back to usable stock chat; a late stylesheet can apply without restarting the conversation.
Existing selectors and legacy variables remain valid; migrate a working older theme only when
requested. Check stock chat and an existing custom theme alongside the new palette.

Configure after the stylesheet is live:

```json
{
  "site_slug": "example",
  "enabled": true,
  "config": {
    "css_path": "https://example.answer.cloud/assets/answercloud-chat.css",
    "chat": {
      "button_text": "Ask Example",
      "intro_text": "What can we help you find?",
      "button_text_color": "#1a1b1c"
    }
  }
}
```

`config` replaces the stored widget configuration when supplied; `{}` resets custom settings.
Omit `config` to toggle `enabled` without losing an existing theme. Text must be concise and
site-specific. `button_text_color` must be a readable hex color.

After configuration, inspect the canonical live page at desktop and mobile widths. Confirm the
CSS URL returns successfully, `list_sites` shows it under `widget.config.css_path`, the launcher
is styled in the parent page, the panel is styled inside the iframe, focus is visible, text meets
contrast expectations, content does not clip, and the host page outside the widget is unchanged.

## Identify the request path

The widget iframe URL contains the platform organization and site identifiers. Treat session
identifiers and chat history as private; do not quote or persist them.

The widget normally makes two independent requests:

- `POST https://app.answer.cloud/api/messages/get_pages` retrieves relevant indexed pages.
- `POST https://app.answer.cloud/api/messages` creates the conversation and generates the answer.

Use the same short, non-sensitive question for both checks. Prefer the live widget first. When a
direct HTTP diagnostic is necessary, send only the minimum JSON fields and do not include API
keys, cookies, personal information, or copied chat history.

## HTTP fallback for a meaningful chat check

Prefer a conversation through the live widget. If browser interaction is unavailable but
HTTP requests are authorized, exercise the public message API with a short, non-sensitive
question whose answer is present on this deployed site:

```text
POST https://app.answer.cloud/api/messages
Content-Type: application/json
```

```json
{
  "organization_id": 123,
  "site_origin": "https://reserved-site.answer.cloud",
  "content": "What services does this organization offer?"
}
```

Replace `123` with the bootstrap's actual numeric `organization_id`, `site_origin` with
its actual reserved `site_url`, and the question with one appropriate to the published
content. Preserve an explicit test application's origin rather than switching environments.
`site_origin` scopes hosted-site retrieval and reference URLs; an organization id alone
must not be used to assume the right site. If the live widget supplies a `site_id`, preserve
that real id as well rather than guessing one.

This endpoint is public and does not use the guest MCP credential. Do not send a build
Bearer token, API key, cookies, account email, personal information, or a custom system
prompt. A successful create returns HTTP 201 with JSON `conversation`, `user_message`,
`ai_message`, and `references`. Check the status and content type before parsing. Verify
that `ai_message.content` is substantive and agrees with the actual site's facts; a generic
greeting, nonempty error text, or HTTP success alone does not prove a useful answer. Check
any cited URLs against the intended live site.

For a follow-up, retain the returned `conversation.id` privately in session memory and
send it as `conversation_id` with the same `organization_id` and `site_origin`, plus the
follow-up `content`. Do not include private conversations, raw responses, or session
identifiers in logs, project files, screenshots, or downloadable artifacts. The test
creates an ordinary stored site conversation, so keep its content non-sensitive.

An optional retrieval diagnostic uses the same `organization_id`, `site_origin`, and
`content` JSON at `POST https://app.answer.cloud/api/messages/get_pages`; it returns
`{ "titles": [...] }`. Relevant titles do not establish that answer generation works.
An API conversation verifies message generation only. Report widget rendering, controls,
and any unavailable visual checks as unverified; do not describe this fallback as a
successful browser test. If no meaningful chat interaction can be completed, retain the
working site URL and state precisely what remains unresolved.

## Interpret failures

| Observation | Likely layer | Next action |
| --- | --- | --- |
| No widget launcher on a live page | Platform injection or stale deploy | Confirm the current deploy, valid HTML with `</body>`, and live hostname. Do not add the script manually. |
| Theme is absent everywhere | Missing/bad `css_path` or asset | Confirm `list_sites`, fetch the absolute CSS URL, and verify the file shipped in the live deploy. |
| Iframe is themed but launcher is not | Stale `embed.js` or unscoped selector mismatch | Confirm the current embed loads the theme in the parent page and that launcher selectors start with `#llmo-chat-button` or `#llmo-chat-iframe`. |
| Host website changes after theming | Leaking custom CSS | Remove unscoped selectors; every rule must begin with a supported widget ID. |
| `get_pages` fails | Ingestion, embeddings, or retrieval | Inspect deploy warnings, page ingestion, and whether published pages have embeddings. |
| `get_pages` succeeds with relevant titles, but `/api/messages` returns 500 | Chat-generation backend | Capture the request ID and response type; investigate the platform application. A site redeploy is not a fix. |
| `/api/messages` succeeds, but the widget shows a generic error | Widget parsing or rendering | Inspect the response shape and browser error. Confirm the endpoint returned JSON. |
| Answer renders but cites the wrong host | Canonical URL construction | Verify hosted pages use `Site#canonical_url`; verify companion pages use the configured `ai.` host. |
| Answer has no useful context | Content or retrieval quality | Confirm the page is ingested, answerable in raw HTML, and retrieved by `get_pages`. |

An HTML 500 page from `/api/messages` is especially important: the widget expects JSON, so
attempting to parse HTML usually collapses to “Sorry, something went wrong.” Record the
`x-request-id`, but do not mistake this client fallback for the root cause.

## Platform-source checks

Only inspect or change the answer.cloud application repository when the user has put that backend in
scope. Otherwise report the cross-repository defect and the evidence needed by the platform owner.

For hosted `hosting` sites:

- Public page and reference URLs must derive from `page.site.canonical_url`.
- A hosted site at `{slug}.answer.cloud` must not require
  `Organization#subdomain` or `Organization#custom_domain`.
- Normalize `source_path` to its pretty public URL. For example, `index.html` becomes `/`, and
  `guides/index.html` becomes `/guides/`.

For `byo_cms` companion sites:

- Canonical URLs should resolve through the site's `ai.{custom_domain}` host.
- Missing organization domain configuration is a real provisioning error; do not silently replace
  it with a new `hosting` site.

When fixing canonical URL handling, add regression coverage for:

1. A standalone hosted site with no custom domain.
2. A standalone hosted site with a custom domain.
3. A `byo_cms` companion site.
4. Root and nested `index.html` source paths.
5. Chat references and generated `.md` mirror URLs.

Also make the widget's message client check `response.ok` and content type before parsing JSON so
future server failures preserve useful diagnostics instead of surfacing only a generic parsing
error.

## Completion gate

Do not call chat fixed until all of these are true:

- Retrieval returns the expected deployed page title.
- Message generation returns a JSON success response.
- The live widget renders the answer.
- References use the canonical live hostname.
- No chat request returns 4xx/5xx.
- The launcher and iframe match the site's visual system at desktop and mobile widths.
- The custom stylesheet causes no style changes outside the widget.
- The evidence comes from the live site, not only a local unit test.
