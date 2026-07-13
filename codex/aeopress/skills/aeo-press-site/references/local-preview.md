# The official CMS template (jfals82/aeopress-cms-template)

A complete, tested Astro project hosted at
[github.com/jfals82/aeopress-cms-template](https://github.com/jfals82/aeopress-cms-template).
It is the required starting point for any aeo.press CMS work — scaffold it, don't rebuild it:

```sh
npm create astro@latest -- --template jfals82/aeopress-cms-template my-cms-site
cd my-cms-site
npm install
npm run dev     # → http://localhost:4321/ is the CMS preview hub
```

(`npx degit jfals82/aeopress-cms-template my-cms-site` or a plain `git clone` also work.)

Modeled on Astro's blog example: you edit source files, everything live-reloads, `npm run
build` emits the deployable output.

## How it works

The same layout components are rendered twice:

- **Preview routes** (`src/pages/index.astro` + `src/pages/cms-preview/…`) pass sample content
  from `sample-entries.mjs`, converted with `server-markup.mjs` so the substitution is exactly
  what aeo.press does at publish time (escaped text, `July 12, 2026` dates, markdown→HTML
  content, the literal `<ul class="entries">` markup, `p.empty`, `nav.aeo-related`).
- **Generator pages** (`src/pages/cms-templates/…`) pass literal marker strings
  (`<!-- aeo:title -->` …). After `astro build`, `scripts/relocate-cms-templates.mjs` moves the
  built HTML into `.aeo/cms/{key}/{show,index}.html`, un-escapes the markers Astro escaped,
  prepends a `GENERATED FILE` banner, writes `aeo-cms.json` from `collections.mjs`, and strips
  every dev-only page from `dist/`.

**CSS is bundled, never a public file**: `CmsShell.astro` does `import "../styles/cms.css"`, so
Vite owns the stylesheet and edits hot-reload in every open preview. The relocate script then
concatenates the hashed bundles the generated templates reference into `dist/assets/aeo-cms.css`
and rewrites their `<link>` tags to that stable URL. Moving the CSS to `public/` breaks hot
reload; hand-writing `<link>` tags breaks the rewrite — do neither.

Preview URLs (the hub links to these; keep them exact):

| URL | Renders |
| --- | --- |
| `/` | Preview hub: collection cards with iframe thumbnails, file map, marker cheat sheet |
| `/cms-preview/{key}/show/` | Entry page with the kitchen-sink sample entry |
| `/cms-preview/{key}/` | Collection index listing the sample entries |
| `/cms-preview/{key}/empty/` | Index empty state (`p.empty`) |

There is no `/cms-preview/{key}/index/` route — `[key]/index.astro` builds to
`/cms-preview/{key}/`.

## Prop contract (what keeps generation and preview in sync)

`CmsShowTemplate` / `CmsIndexTemplate` take:

- **Text props** (`title`, `summary`, `publishedAt`, `name`): plain text. Astro escapes them —
  matching how aeo.press escapes those markers. Generator pages pass marker strings here; the
  relocate script un-escapes them (`&lt;!-- aeo:x --&gt;` → `<!-- aeo:x -->`).
- **HTML props** (`contentHtml`, `entriesHtml`): raw HTML rendered with `set:html`.

Keep this contract when editing the layouts; it is why one component serves both purposes.

## Adapting the template to a project

1. `src/consts.ts` — site title/description.
2. `src/cms/collections.mjs` — replace `blog` with the real collections. For a `byo_cms`
   companion site, mirror `list_cms_collections` exactly (keys, `path_pattern`, `index_path`,
   fields); never invent collections.
3. `src/styles/cms.css` — restyle to the user's brand. Keep coverage of everything the
   kitchen-sink sample exercises: `.cms-content` typography (h2–h4, lists, links, `pre>code`,
   blockquote, tables, images), `ul.entries li`, `.summary`, `.empty`, `.aeo-related`.
4. `src/cms/sample-entries.mjs` — make samples read like the real site; keep the first entry a
   kitchen-sink (headings, lists, code, blockquote, table, image) and keep one entry without a
   summary. Update `src/styles/preview.css` only if the hub chrome needs it — it never ships.
5. `README.md` — keep the "Which file do I edit?" table accurate.

## Commands

- `npm run dev` — preview hub at `/`, hot reload for layouts and `cms.css`.
- `npm run build` — `astro build` + relocate: writes `.aeo/cms/**`, `aeo-cms.json`,
  `dist/assets/aeo-cms.css`; strips dev pages from `dist/`.
- `npm run package` — build + assemble `.deploy/` (manifest, `.aeo/`, `assets/aeo-cms.css`
  only) + write `site.zip`, ready for `start_deploy` → upload → `finalize_deploy`. This is the
  `byo_cms` artifact shape; for a `hosting` site, ship all of `dist/` plus `.aeo/` and
  `aeo-cms.json` per SKILL.md's packaging rules instead.

## Verify before handing off (browser tools, not the user)

1. `npm run dev`, open `/`, click every collection's entry / index / empty links — no 404s.
2. Edit `src/styles/cms.css` (e.g. h1 color) — the open preview must update without a manual
   refresh. Revert the edit.
3. Edit `CmsShowTemplate.astro` — the preview must hot-reload.
4. `npm run build` — each `.aeo/cms/{key}/show.html` starts with the banner, contains literal
   `<!-- aeo:content -->` (not `&lt;!--`), links `/assets/aeo-cms.css` (no `/_astro/` hrefs),
   and `dist/` has no `cms-preview/`, `cms-templates/`, or root `index.html`.
5. `npm run package` — `site.zip` contains only `aeo-cms.json`, `.aeo/**`, `assets/aeo-cms.css`.
