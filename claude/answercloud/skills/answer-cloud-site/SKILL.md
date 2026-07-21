---
name: answer-cloud-site
description: >-
  Scaffold, author, build, and deploy an Astro site to answer.cloud with AEO-friendly HTML, editable CMS
  collection templates, the answercloud MCP workflow, and verification of the generated chat/search
  widget. Use when the user wants to build,
  scaffold, or deploy a site for answer.cloud or answercloud, ship a site to {slug}.answer.cloud,
  make a site answer-engine optimized / AEO / LLM-friendly, create a CMS template wizard
  setup for an existing site, build an AI companion site on the ai. subdomain, deploy
  through the answercloud MCP server, verify the injected widget, or diagnose chat failures and
  `/api/messages` 500 errors. Astro source is mandatory; never substitute a hand-written
  plain-HTML project. Before scaffolding or deploying a new or unknown target, first ask whether
  the user wants an AI companion site (CMS-only, served at ai.{their-domain}) or a standalone
  website ({slug}.answer.cloud).
---

# Author and deploy an Astro site for answer.cloud

**The deliverable is an Astro source project, not a folder of hand-written HTML.** Author pages,
layouts, and components as `.astro` source files, then run `astro build`. The resulting HTML in
`dist/` is the deploy artifact; it does not make plain HTML an acceptable source stack.

Treat answer.cloud as a static host that also serves a site to answer engines. On deploy, the
platform ingests your HTML and generates an answer-engine layer: `llms.txt`, plain-text page
mirrors, `sitemap.xml`, `robots.txt` with AI-crawler allows, JSON-LD, analytics, and the
first-party chat/search widget. Sites are served at `https://{slug}.answer.cloud`.

Use Astro to generate clean, content-rich static HTML, then deploy Astro's build output through
the answercloud MCP server.
Do not hand-write `llms.txt`, sitemaps, analytics snippets, or answercloud widget scripts.
The platform adds that layer at deploy time.

**Deploying is always a separate, explicit step.** Scaffold, style, preview, and build locally
by default; call the deploy tools only when the user explicitly asks to deploy (see "Run the
deploy loop through answercloud MCP").

## First, decide which kind of site you are building

Before scaffolding or deploying a new or unknown target, **ask the user** which of these two they
want. The answer changes the deploy target, what files ship, and the served hostname. Do not
guess. Do not repeat this question when diagnosing or updating an existing site whose kind is
already established by `list_sites`, its hostname, or the current task context.

Ask, in plain language, something like:

> Are you building an **AI companion site** for an existing website (a CMS-driven site served
> at the `ai.` subdomain of your own domain, e.g. `ai.example.com`), or a **standalone
> website** hosted on answer.cloud (served at `{slug}.answer.cloud`)?

Make the distinction explicit so the user understands what they are creating:

- **AI companion site (`byo_cms`)** — a CMS-only site that lives *alongside* the user's real
  website, on the `ai.` subdomain of the user's own domain (`{org.subdomain||'ai'}.{org.custom_domain}`,
  e.g. `ai.example.com`). It publishes **only** pages generated from the organization's CMS
  collections. It is not a general website; it is the answer-engine/CMS companion to one.
- **Standalone website (`hosting`)** — a full site the user authors, hosted on answer.cloud and
  served at `{slug}.answer.cloud` (plus any custom domain they attach). It can contain arbitrary
  pages *and* optionally CMS collections.

Route by the answer:

| Answer | Site `kind` | Served at | What ships in the artifact |
| --- | --- | --- | --- |
| AI companion site | `byo_cms` | `ai.{org.custom_domain}` | **Only** CMS templates + manifest + referenced assets — see "AI companion (CMS-only) deploys" |
| Standalone website | `hosting` | `{slug}.answer.cloud` | Full built site (all of `dist/`), optionally plus CMS templates |

If the user has not confirmed which one, ask before doing anything else.

## AI companion (CMS-only) deploys

Follow this section **only** when the user chose the AI companion site. It overrides the
general packaging rules below.

The served hostname is derived from the org, not from the deploy — a `byo_cms` site
canonically serves at `{org.subdomain||'ai'}.{org.custom_domain}` (e.g. `ai.example.com`).
To hit it you must deploy to the org's **`byo_cms` site** (deterministic slug `{org.slug}-cms`),
using that **organization's** MCP key. A `hosting`-kind site can never serve at the `ai.`
subdomain — it only serves at `{slug}.answer.cloud`.

Two hard rules for these deploys:

1. **Deploy to the `byo_cms` site of the correct org.** Run `list_sites` with the org's key and
   confirm the `{org.slug}-cms` site is present; deploy to that slug. If it is missing, the org
   likely does not have BYO CMS enabled / a custom domain set — tell the user; do not create a
   `hosting` site as a substitute.
2. **Ship only files generated from the org's CMS collections.** The deploy pipeline serves
   *every* static file in the artifact, and on a path collision the static file wins and the CMS
   render is skipped (with a `cms_path_collision` warning). So do **not** ship Astro's built
   public pages. Use Astro purely as the local generator for the CMS templates.

**Enumerate the org's existing collections first — never invent, infer from the user's words,
or create new ones for a companion site.** The set of collections is whatever already exists in
the organization; the AI's job is to mirror it, not extend it.

1. Run `list_cms_collections` on the `{org.slug}-cms` site to get the real collection keys,
   their `path_pattern`/`index_path`, and field schemas. This list is the single source of truth.
2. **If it returns zero collections, stop and tell the user** — there is nothing to build a
   companion site from. Do not scaffold speculative collections and do not call
   `upsert_cms_collection` to invent them. The user must create the collections first (in the
   dashboard) or explicitly ask you to create specific ones.
3. Generate `.aeo/cms/{key}/show.html` + `index.html` for **exactly** those keys — no more, no
   fewer — and make `aeo-cms.json`'s `collections` match that set one-to-one, reusing each
   collection's existing `path_pattern`/`index_path`/fields rather than choosing your own.
4. Ship nothing at a collection-owned path, and no stray pages. You do not need an `index.html`:
   for `byo_cms` sites the publisher writes the AI chat page at `/` automatically, and any
   collection whose `index_path` is `/` is rendered from CMS.

Package **only** the CMS pieces. The official template repo does this with one command:

```sh
npm run package    # build + assemble .deploy/ + write site.zip
```

which is equivalent to (note: no `cp -R dist/. .deploy/`):

```sh
npm run build                                 # .aeo/cms/**, aeo-cms.json, dist/assets/aeo-cms.css
rm -rf .deploy && mkdir -p .deploy/assets
cp aeo-cms.json .deploy/                       # manifest at artifact root
cp -R .aeo       .deploy/.aeo                  # private templates → stashed, never served
cp dist/assets/aeo-cms.css .deploy/assets/     # only assets the templates reference
(cd .deploy && zip -r /tmp/site.zip .)
```

Then — only once the user has explicitly asked to deploy — run the normal `start_deploy` →
upload → `finalize_deploy` → `get_deploy` loop against the
`{org.slug}-cms` slug. Verify the result at `ai.{org.custom_domain}`, and confirm there are no
`cms_path_collision` warnings (a collision means a stray static file shadowed a CMS page — remove
it and redeploy).

## Require real HTML, not an empty shell

Serve static files only. Do not rely on SSR, serverless functions, or a server-side build
step on answer.cloud. Deploy the static artifact Astro emits (`dist/`). Every answercloud site —
standalone or AI companion, large or single-page — must be built with Astro; never hand-write
or deploy a folder of plain `.html` files.

Put substantive content directly into the served HTML. The ingestion pipeline and answer
engines read raw markup without executing client JavaScript. A client-only SPA that renders
into an empty `<div id="root">` will produce a `spa_shell` warning and poor retrieval.

Generate the site with Astro, which statically renders content into the HTML. If the user
insists on React or Vue, wrap it in Astro's static output so page content lands in the HTML.

## Required stack: Astro (mandatory for every site)

Always scaffold the site with [Astro](https://astro.build) — no exceptions. This applies to
both site types (standalone `hosting` and AI companion `byo_cms`) and to any size, including
small or single-page sites. Astro outputs static HTML, keeps content in markup, and maps well
to the answercloud ingest pipeline. Never hand-write a folder of plain `.html`/`.css` files; build
them as Astro pages and let `astro build` emit the static HTML. For AI companion sites, Astro is
still the required local generator for the CMS templates even though the built `dist/` pages are
not shipped.

```bash
npm create astro@latest -- --template minimal my-site
cd my-site
npm install
npm run build
```

For CMS work (an AI companion site, or adding CMS collections to a hosting site), do not start
from the minimal template — scaffold from the official CMS template repo instead (see "Run the
CMS template wizard" below): `npm create astro@latest -- --template
jfals82/answercloud-cms-template my-cms-site`. It ships the CMS layouts, live preview, and build
pipeline pre-wired.

Avoid client-only SPA setups; Astro's default static output is what you want.

### Astro source-project completion gate

Before authoring page content, scaffold or verify the Astro source project. Do not treat a folder
containing only `.html`, `.css`, and `.js` files as a valid implementation, even when the site is
small or has only one page.

A site-generation task is complete only when all of these are true:

- `package.json` declares `astro` and provides `dev` and `build` scripts that run Astro.
- `astro.config.mjs` (or an equivalent supported Astro config) exists and keeps static output.
- Page source lives under `src/pages/**/*.astro`; shared chrome belongs in an Astro layout or
  component rather than being duplicated across hand-written HTML pages.
- `npm run build` succeeds and produces `dist/**/*.html` from the `.astro` source.
- Hand-written `.html` is limited to generated build output and the generated `.aeo/cms/**`
  templates required by answer.cloud. It is never the site's primary editable source.
- `CLAUDE.md` and `AGENTS.md` context markers exist at the project root (see "Write the
  project context marker" below).

If `npm create astro` cannot run, create the minimal Astro project files directly and install the
dependencies when possible. Do not silently fall back to a plain-HTML site; report the blocked
dependency or build instead.

### Write the project context marker (CLAUDE.md + AGENTS.md) — mandatory

Every answer.cloud project must carry a context marker so any future session — Claude Code, Codex,
or another agent — immediately knows this is an answercloud site and re-loads this skill instead of
treating it as a generic Astro project. Claude Code auto-loads `CLAUDE.md`; Codex auto-loads
`AGENTS.md`. Ship both, kept identical via symlink.

When scaffolding a site (and whenever you touch an existing answer.cloud project that lacks the
marker), write `CLAUDE.md` at the project root:

```markdown
# answer.cloud site

- Site kind: hosting            <!-- or byo_cms (AI companion) -->
- Slug: {slug}                  <!-- fill in once known -->
- Served at: https://{slug}.answer.cloud   <!-- or ai.{custom_domain} for byo_cms -->
- Organization: {org name/slug}
- Source: Astro. Never hand-write plain HTML, llms.txt, sitemaps, analytics, or
  chat-widget scripts — the platform generates that layer at deploy time.
- Deploy: through the answercloud MCP server only, and only when explicitly requested.

For ANY work on this site — authoring, styling, CMS templates, building, deploying,
or verifying/debugging the chat widget — invoke the `answer-cloud-site` skill first
and follow it.
```

Then link and commit:

```sh
ln -s CLAUDE.md AGENTS.md
git add CLAUDE.md AGENTS.md   # if symlinks are unsupported, copy instead and keep both in sync
```

Fill in `Slug`, `Served at`, and `Organization` as soon as they are known — at the latest,
update the marker right after the first successful deploy so the values reflect reality. When
updating an existing marker, preserve any user-added notes.

## Follow AEO-friendly authoring rules

1. Use one `<h1>` per page and keep a semantic heading hierarchy.
2. Write self-contained, question-shaped sections that fully answer real user questions.
3. Add an FAQ section with clear Q/A pairs when it fits; the platform can surface FAQ
   structure directly.
4. Set a unique `<title>` and `<meta name="description">` on every page.
5. Use clean directory-style URLs such as `/pricing/` or `/docs/getting-started/`.
6. Cross-link pages with real `<a href>` links.
7. Keep content in the markup, not hidden behind client-rendered tabs, accordions, or lazy
   loaders.
8. For editable content, ship CMS templates under `.aeo/cms/{collection_key}/show.html`
   and `index.html` (see "Define CMS collections" below) — not in the public `dist/` tree.

Do not add the chat widget manually. Write content that the widget can answer from.

## Treat chat as a generated platform feature

answer.cloud injects the first-party chat/search widget at serve time. Do not add `embed.js`, copy
widget markup, create a local chat endpoint, or add an AI SDK to the Astro project to make the
widget appear. Local Astro previews do not prove whether the production widget was injected.

When the user asks to add, verify, or debug the widget:

1. Inspect the live answer.cloud URL and the current live deploy before changing local source.
2. Confirm the widget is absent or broken; distinguish a missing launcher from a failed answer.
3. If the widget is missing, investigate deploy status and platform injection. Do not add the
   script manually.
4. If an answer fails, test retrieval and message generation separately. A healthy retrieval
   response does not prove that chat generation works.
5. Capture the failing route, HTTP status, response content type, and request ID. Never expose
   API keys, cookies, or private chat history.
6. Treat a platform API 500 as a platform/backend defect. Do not redeploy unchanged Astro output
   or claim the site content caused it without evidence.

Read [references/chat-widget.md](references/chat-widget.md) completely for the live verification
and troubleshooting procedure whenever chat behavior is in scope.

## Run the CMS template wizard for editable content

When the user wants an existing site to publish editable CMS entries through answer.cloud, run a
wizard instead of pasting a one-off prompt. Treat Astro as the local authoring, preview, and
generation environment; the deployable output is still static HTML plus public assets.

**Scope:** this wizard (defining brand-new collections from the user's requirements) is for
**standalone `hosting` sites**. For an **AI companion (`byo_cms`) site**, do not define
collections from the user's words — mirror the org's existing collections exactly, per "AI
companion (CMS-only) deploys" above.

### Wizard flow

1. Inspect the existing project structure, styling entrypoints, build command, and public asset
   folder before deciding where files should live.
2. Identify requested CMS collections from the user's words. Do not assume `blog`. If details are
   missing, ask only for the missing collection key, display name, URL pattern, fields, and the
   existing site style source to match.
3. **Start from the official template repo.** Scaffold from
   [jfals82/answercloud-cms-template](https://github.com/jfals82/answercloud-cms-template) instead of
   assembling the layout by hand:

   ```sh
   npm create astro@latest -- --template jfals82/answercloud-cms-template my-cms-site
   cd my-cms-site && npm install
   ```

   (`npx degit jfals82/answercloud-cms-template my-cms-site` works too.) See
   [references/local-preview.md](references/local-preview.md) for the walkthrough. The layout:
   - `src/cms/CmsShell.astro`, `CmsShowTemplate.astro`, `CmsIndexTemplate.astro` — the
     editable shell/entry/listing layouts, shared by generation and preview.
   - `src/cms/collections.mjs` — collection definitions; single source for the manifest,
     generator pages, and preview routes.
   - `src/cms/sample-entries.mjs` + `src/cms/server-markup.mjs` — preview-only sample
     content and the exact server-injected markup.
   - `src/styles/cms.css` — THE stylesheet. It is `import`ed by `CmsShell.astro`, so Vite
     bundles it and CSS edits hot-reload in the preview; the build rewrites the bundled
     links to one stable `/assets/aeo-cms.css`.
   - `src/pages/index.astro` — the CMS preview hub; the project's entry point in dev.
   - `src/pages/cms-preview/…` — dev-only preview routes; `src/pages/cms-templates/…` —
     build-only marker pages. Both are stripped from `dist/` on every build.
   - `scripts/relocate-cms-templates.mjs` — post-build: relocates templates into
     `.aeo/cms/**`, un-escapes markers, bundles CSS, writes `aeo-cms.json`, prepends
     generated-file banners, strips dev pages. `scripts/package.mjs` — zips the `byo_cms`
     deploy artifact (`npm run package`).
4. Adapt the template to the project: set `src/consts.ts`, replace the `blog` collection in
   `collections.mjs` with the real collections (for `byo_cms`, mirror `list_cms_collections`
   exactly), restyle `src/styles/cms.css` to the user's brand, and refresh
   `sample-entries.mjs` so previews look like the real site.
5. Keep the README's "Which file do I edit?" section accurate — it lists the editable files
   and the generated ones (`.aeo/cms/**`, `aeo-cms.json`, `dist/**`). Every generated
   template gets a `GENERATED FILE` banner from the relocate script; the preview hub repeats
   the file map. This is the answer to "which file do I edit?" — never weaken it.

### Local preview and live reload (mandatory)

CMS templates are rendered server-side by answer.cloud, so without help the developer cannot see a
template until after a deploy, and CSS edits are invisible locally. The official template repo
(`jfals82/answercloud-cms-template`) implements all of this; keep every property below intact when
adapting it (walkthrough: [references/local-preview.md](references/local-preview.md)).

1. **The preview hub is the entry point.** `npm run dev` → `http://localhost:4321/` IS the CMS
   preview hub (`src/pages/index.astro`): collection cards with live iframe thumbnails and
   links to each preview, the "Your files — edit freely" vs "Generated output — never edit"
   split, the marker cheat sheet, and the ship-it steps. Tell the user to open it first.
2. **Per-collection preview routes** — `/cms-preview/{key}/show/` (entry),
   `/cms-preview/{key}/` (index), and `/cms-preview/{key}/empty/` (empty state) render the
   *same* `CmsShowTemplate` / `CmsIndexTemplate` components used for generation, filled with
   `sample-entries.mjs` data. Note the index preview lives at `/cms-preview/{key}/` — never
   link `/cms-preview/{key}/index/`, which does not exist.
3. **Faithful substitution** — previews inject exactly what the server injects (see "What
   replaces each marker" below) via `server-markup.mjs`: escaped text for title/summary/dates,
   markdown-rendered HTML for content, and the server's exact `<ul class="entries">` markup.
   Sample markdown must exercise headings, lists, links, code blocks, blockquotes, tables, and
   images so the stylesheet gets styled against everything real entries can contain.
4. **Never shipped** — the relocate script strips `dist/cms-preview/`, `dist/cms-templates/`,
   and the hub's `dist/index.html` on every build, so dev pages cannot leak into a deploy.
5. **CSS is bundled, not a public file.** `CmsShell.astro` does `import "../styles/cms.css"`,
   so Vite owns the stylesheet and edits hot-reload instantly in every preview — no plugins,
   no manual refresh. At build time the relocate script concatenates the hashed bundles the
   templates reference into `dist/assets/aeo-cms.css` and rewrites the template `<link>`s to
   that stable URL. Do not move the CSS to `public/` (kills hot reload) and do not hand-write
   `<link rel="stylesheet">` tags in the shell.

### What replaces each marker (style against this)

answer.cloud substitutes markers server-side with fixed markup. The template's preview hub and
`server-markup.mjs` encode this; keep them consistent with this table:

| Marker | Replaced with |
| --- | --- |
| `<!-- aeo:title -->`, `<!-- aeo:summary -->` | HTML-escaped plain text (safe in attributes, e.g. meta description) |
| `<!-- aeo:published_at -->` | escaped date formatted like `July 12, 2026` |
| `<!-- aeo:content -->` | markdown rendered to HTML (`h2`–`h4`, `p`, `ul`/`ol`, `a`, `img`, `pre>code`, `blockquote`, tables); may end with a related-links block: `<nav class="aeo-related" aria-label="Related"><h2>Related</h2><ul><li><a …>…</a></li></ul></nav>` |
| `<!-- aeo:entries -->` | `<ul class="entries"><li><a href="…">Title</a> <span class="summary">…</span></li>…</ul>`, or `<p class="empty">No entries yet.</p>` when the collection is empty |
| `<!-- aeo:collection_name -->`, `<!-- aeo:collection_summary -->` | HTML-escaped plain text |

At serve time answer.cloud also injects JSON-LD into `<head>` and the chat widget before
`</body>` — never add those to templates or previews. Stylesheets should therefore cover at
minimum: `.cms-content` typography, `ul.entries li`, `.summary`, `.empty`, and `.aeo-related`.

### CSS and assets

One canonical stylesheet: `src/styles/cms.css`, imported by `CmsShell.astro` so Vite bundles it
(hot reload in dev). The build rewrites the bundled links in the generated templates to a single
stable public asset:

```html
<link rel="stylesheet" href="/assets/aeo-cms.css">
```

The relocate script writes that file to `dist/assets/aeo-cms.css`; ship it with the artifact
(the `byo_cms` packaging copies it explicitly; `hosting` deploys ship all of `dist/`). Inline
CSS only when the user explicitly asks for a fully self-contained template.

### Packaging rules (important)

These rules are for **standalone `hosting` sites**. For an **AI companion (`byo_cms`) site**,
use "AI companion (CMS-only) deploys" above instead — ship only CMS templates + manifest +
referenced assets, never the built `dist/` pages.

- Put public build output (HTML/CSS/JS/images) at the **artifact root**.
- Put CMS templates under private `.aeo/cms/{collection_key}/show.html` and
  `.aeo/cms/{collection_key}/index.html`; answer.cloud stores them privately and never serves them
  publicly.
- Put `aeo-cms.json` at the artifact root.
- Do not put CMS templates in the public `dist/` tree.
- Do not ship the dev helpers: `dist/` must contain no `cms-preview/` or `cms-templates/`
  pages (the relocate script removes them; verify before zipping).
- Do not create a static file where answer.cloud owns the CMS path. If a collection owns `/resources/`,
  do not ship `dist/resources/index.html`; answer.cloud generates that index from the CMS template.
- For Astro: build to `dist/`, then assemble a temporary deploy dir containing the contents of
  `dist/` at the root plus `.aeo/` and `aeo-cms.json` beside them:

  ```sh
  npm run build
  rm -rf .deploy && mkdir .deploy
  cp -R dist/. .deploy/
  cp -R .aeo .deploy/.aeo
  cp aeo-cms.json .deploy/aeo-cms.json
  (cd .deploy && zip -r /tmp/site.zip .)
  ```

### Manifest (`aeo-cms.json` at the artifact root)

Use modern keys: `index_path`, `path_pattern`, `templates.show`, `templates.index`, object-style
`fields`, and optional `sort`.

```json
{
  "collections": {
    "resources": {
      "name": "Resources",
      "description": "Helpful articles, guides, and explainers.",
      "type": "Article",
      "index_path": "/resources/",
      "path_pattern": "/resources/{slug}/",
      "templates": {
        "show": ".aeo/cms/resources/show.html",
        "index": ".aeo/cms/resources/index.html"
      },
      "fields": [
        { "key": "title", "label": "Title", "type": "string", "required": true },
        { "key": "summary", "label": "Summary", "type": "text" },
        { "key": "content", "label": "Content", "type": "markdown", "required": true },
        { "key": "published_at", "label": "Published At", "type": "datetime" },
        { "key": "author", "label": "Author", "type": "string" },
        { "key": "tags", "label": "Tags", "type": "tags" }
      ],
      "sort": { "field": "published_at", "direction": "desc" }
    }
  }
}
```

### Required template markers

`templates.show` must contain `<!-- aeo:content -->`; `templates.index` must contain
`<!-- aeo:entries -->`. Show templates may also use `<!-- aeo:title -->`,
`<!-- aeo:summary -->`, and `<!-- aeo:published_at -->`. Index templates may also use
`<!-- aeo:collection_name -->` and `<!-- aeo:collection_summary -->`. If a template is missing or
lacks its marker, answer.cloud renders with a built-in fallback theme and emits a deploy warning.

Example generated show template:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title><!-- aeo:title --></title>
  <meta name="description" content="<!-- aeo:summary -->">
  <link rel="stylesheet" href="/assets/aeo-cms.css">
</head>
<body>
  <header><a href="/">Site name</a><nav><a href="/resources/">Resources</a></nav></header>
  <main class="cms-show">
    <article>
      <h1><!-- aeo:title --></h1>
      <p class="cms-summary"><!-- aeo:summary --></p>
      <time class="cms-published"><!-- aeo:published_at --></time>
      <div class="cms-content"><!-- aeo:content --></div>
    </article>
  </main>
  <footer><a href="/">Home</a></footer>
</body>
</html>
```

Example generated index template:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title><!-- aeo:collection_name --></title>
  <meta name="description" content="<!-- aeo:collection_summary -->">
  <link rel="stylesheet" href="/assets/aeo-cms.css">
</head>
<body>
  <header><a href="/">Site name</a></header>
  <main class="cms-index">
    <h1><!-- aeo:collection_name --></h1>
    <p class="cms-summary"><!-- aeo:collection_summary --></p>
    <section class="cms-entries" aria-label="Entries">
      <!-- aeo:entries -->
    </section>
  </main>
  <footer><a href="/">Home</a></footer>
</body>
</html>
```

Backward compatibility: older manifests using `path`, `template`, `template_path`,
`index_template_path`, and `type` still work, but prefer `path_pattern`, `index_path`,
`templates.show`, `templates.index`, and object-style `fields`.

After deploy, collections appear in the `/site/{slug}/cms` dashboard and via the CMS MCP tools.
Publishing an entry renders the show page and collection index, writes `.txt` mirrors, refreshes
`llms.txt`/`sitemap.xml`, injects JSON-LD and the widget layer, and includes both in search/chat.
Avoid CMS paths that collide with artifact files; the artifact path wins during deploy and a direct
publish should choose another slug/path.

### Wizard verification checklist

Before finishing, verify:

- Every show template contains `<!-- aeo:content -->` (and no marker arrives HTML-escaped as
  `&lt;!-- aeo:… --&gt;` — the relocate script must unescape markers Astro escaped).
- Every index template contains `<!-- aeo:entries -->`.
- Every template links to the shared public CSS file.
- The CSS file exists in the public artifact.
- `aeo-cms.json` paths match generated files.
- No static public file collides with a CMS-owned path.
- Every generated template starts with the generated-file banner comment, and its stylesheet
  link points at `/assets/aeo-cms.css` (not a hashed `/_astro/*.css` path).
- `npm run dev` serves the preview hub at `/` with working entry (`/cms-preview/{key}/show/`),
  index (`/cms-preview/{key}/`), and empty (`/cms-preview/{key}/empty/`) previews for every
  collection — no hub link may 404 — and editing `src/styles/cms.css` or a `src/cms/*.astro`
  file updates the open preview without a manual refresh. Verify this yourself with the
  browser tools; do not hand it to the user unchecked.
- `dist/` contains no `cms-preview/`, `cms-templates/`, or hub `index.html` after build.
- The README's "Which file do I edit?" table matches the actual project layout.
- The build succeeds, and `npm run package` produces an artifact containing only
  `aeo-cms.json`, `.aeo/**`, and `assets/aeo-cms.css` for `byo_cms` sites.

## Run the deploy loop through answercloud MCP

**Deploy only when the user explicitly asks.** Building, previewing, and packaging are local
work; never call `create_site`, `deploy_files`, `start_deploy`, `finalize_deploy`, `rollback`,
or `republish_cms` just because a build is ready, as the tail end of a wizard run, or to
"verify" your work. When the local work is done, report it and stop — deploy only after the
user says to deploy/ship/publish/go live in their own words, and a single approval covers a
single deploy, not future ones. Read-only tools (`list_sites`, `list_deploys`, `get_deploy`,
`list_cms_collections`, `list_cms_entries`) are fine at any time.

Drive deploys through the answercloud MCP server at `https://answer.cloud/mcp`.

Use this core loop:

1. Run `list_sites` to inspect existing sites.
2. Run `create_site` with a DNS-safe slug if the site does not exist yet.
3. Build the static artifact locally.
4. Run `start_deploy` with `site_slug`.
5. Run the returned upload command after fixing the build directory path.
6. Run `finalize_deploy` with the `deploy_id`.
7. Poll `get_deploy` until the status reaches `live`.
8. Read `warnings`, fix the site, and redeploy until the warnings are clean.

Use `deploy_files` only for very small changesets of 500 KB or less. For full sites, use
`start_deploy` so the artifact uploads directly to storage instead of flowing through model
context.

### CMS authoring loop (only when the user asks)

After a CMS-ready site is live, use the CMS tools:

1. `list_cms_collections` to confirm collections and template health synced from the manifest.
2. If a collection is missing, fix `aeo-cms.json`/`.aeo/cms/**` and redeploy, or call
   `upsert_cms_collection`.
3. `upsert_cms_entry` with `save_mode: "draft"` to create content **only when asked** to author.
4. `preview_cms_entry` to give the user a private in-answer.cloud preview URL.
5. `publish_cms_entry` **only after the user approves** — it returns live/index URLs and written paths.
6. `archive_cms_entry` to unpublish; `republish_cms` to re-render after template changes.

Read [references/mcp-tools.md](references/mcp-tools.md) for the tool shapes, warnings, and
polling expectations.

### GitHub push-to-deploy (only when the user asks)

When the user asks to connect GitHub, set up push-to-deploy, or "deploy on push", use the
GitHub tools instead of manual deploys:

1. `github_get_connection` — if not connected, give the user the returned `install_url` to
   open in a browser (they pick the repos), then call it again once they confirm.
2. `github_link_repo` with `site_slug`, `repo` (`owner/name`), and `build`:
   - `actions` (default, right for Astro): the response includes the workflow file
     (`.github/workflows/answercloud.yml`). Write it into the repo, commit, and push. From
     then on every push builds in GitHub Actions and deploys via OIDC — no secrets.
   - `none`: the repo already is the built output; pushes deploy directly with no CI.
3. One repo powers one site. After a push, check status with `list_deploys`/`get_deploy`
   (deploys carry the commit `git` metadata).
4. `github_trigger_deploy` redeploys the branch head of a `build: none` site without a push;
   `github_unlink_repo` turns push-to-deploy off.

Linking a repo is configuration, not a deploy — it still requires the user's explicit ask,
and the first push that deploys is theirs to make.

## MCP connection and auth

The answercloud MCP server lives at `https://answer.cloud/mcp` (Streamable HTTP, Bearer auth). If
MCP auth fails, tell the user to create a hosting API key in the answer.cloud dashboard
(Settings -> API & MCP). The key starts with `aeo_host_`.

If this skill came from the **answercloud plugin** (Claude Code or Codex), the server config is
already bundled — the user only needs to export the key and restart the session:

```sh
export AEOPRESS_API_KEY=aeo_host_YOUR_KEY
```

Manual setup without the plugin:

- Claude Code:

  ```sh
  claude mcp add --transport http answercloud https://answer.cloud/mcp \
    --header "Authorization: Bearer aeo_host_YOUR_KEY"
  ```

- Codex — add to `~/.codex/config.toml`:

  ```toml
  [mcp_servers.answercloud]
  url = "https://answer.cloud/mcp"
  bearer_token_env_var = "AEOPRESS_API_KEY"
  ```

## Verify the live result

After a deploy goes live, verify the public site and the answer-engine layer:

1. Open `https://{slug}.answer.cloud`.
2. Check that key pages contain full HTML content, not empty client shells.
3. Verify generated artifacts such as `/llms.txt`, `sitemap.xml`, and page `.txt` mirrors.
4. Read deploy warnings before declaring success.
5. Confirm the chat launcher is present. If the user asked to verify chat or reported it broken,
   run one short, non-sensitive question grounded in the deployed content and follow
   [references/chat-widget.md](references/chat-widget.md).

If the MCP server exposes post-deploy helpers such as analytics or search tools, use them
to confirm the site is discoverable and answerable.

## Apply practical limits

- Treat the platform as static hosting only.
- Keep artifacts within the plan limits: 10 MB on free, 100 MB on paid.
- Design within indexed page limits: 12 pages on free, 500 on paid.
- Respect deploy quotas: 30 deploys/day on free, 200 on paid.
- Expect custom domains to require a paid plan.

## Use this end-to-end recipe

1. Scaffold an Astro source project, write the `CLAUDE.md`/`AGENTS.md` context marker, and
   pass the Astro source-project completion gate.
2. Author semantic, question-oriented pages; for editable content run the CMS template
   wizard, generate `aeo-cms.json` and `.aeo/cms/{key}/show.html` + `index.html`, and keep
   the shared CMS CSS in the public build.
3. Build to a static output directory.
4. Stop here and report the local result. Continue only if the user has explicitly asked to
   deploy.
5. Create the site if needed, then run `start_deploy`, upload, and `finalize_deploy`.
6. Poll `get_deploy` to `live`.
7. Fix warnings and redeploy until clean (fixing warnings on a deploy the user just requested
   does not need re-approval).
8. Update the context marker with the final slug, served hostname, and organization.
9. Report the live `https://{slug}.answer.cloud` URL and note that the answer-engine layer is
   generated automatically.
