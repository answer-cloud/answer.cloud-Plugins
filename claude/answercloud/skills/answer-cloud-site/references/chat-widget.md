# Chat widget verification and troubleshooting

Use this reference when a user asks to add, verify, or debug the answer.cloud chat/search widget.
The widget and its API are platform-owned; site authors supply answerable static content.

## Verification flow

1. Run `list_sites` and identify the exact site, kind, live URL, current deploy, and page usage.
2. Run `list_deploys`; confirm the intended deploy is `live`.
3. Open the live URL. Do not use the local Astro preview to judge widget injection.
4. Confirm the page contains substantive HTML and the injected widget launcher.
5. Open the widget and, only when live chat verification is in scope, submit one short,
   non-sensitive question grounded in the page.
6. Confirm the response renders, relevant references use the site's canonical host, and the
   browser reports no failed chat request.

Do not add `https://app.answer.cloud/embed.js` to Astro source. The platform injects it before
`</body>` during publishing.

## Identify the request path

The widget iframe URL contains the platform organization and site identifiers. Treat session
identifiers and chat history as private; do not quote or persist them.

The widget normally makes two independent requests:

- `POST https://app.answer.cloud/api/messages/get_pages` retrieves relevant indexed pages.
- `POST https://app.answer.cloud/api/messages` creates the conversation and generates the answer.

Use the same short, non-sensitive question for both checks. Prefer the live widget first. When a
direct HTTP diagnostic is necessary, send only the minimum JSON fields and do not include API
keys, cookies, personal information, or copied chat history.

## Interpret failures

| Observation | Likely layer | Next action |
| --- | --- | --- |
| No widget launcher on a live page | Platform injection or stale deploy | Confirm the current deploy, valid HTML with `</body>`, and live hostname. Do not add the script manually. |
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
5. Chat references and generated `.txt` mirror URLs.

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
- The evidence comes from the live site, not only a local unit test.
