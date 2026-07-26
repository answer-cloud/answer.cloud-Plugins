# Official answer.cloud style defaults

Use this reference whenever a new answer.cloud site or CMS template needs visual styling. The
upstream source of truth is
[answer-cloud/answer_cloud_template](https://github.com/answer-cloud/answer_cloud_template).
Inspect its current `AGENTS.md` or `CLAUDE.md` and style files when network access is available;
the summary here is the offline baseline, not permission to ignore newer upstream guidance.

## Choose the right source

| Project context | Visual source |
| --- | --- |
| New standalone site with no supplied brand | The official answer.cloud template, unchanged as the starting system |
| New standalone site with supplied brand direction | The official template's structure and interaction quality, retokenized to the supplied brand |
| Existing site | The existing site's rendered UI and source tokens; use the official template only as a completeness check |
| CMS or AI companion with an existing parent site | The parent site's design system |
| CMS or AI companion without a usable brand source | The official answer.cloud template defaults applied to the CMS template |

Do not ask a user for colors, fonts, radii, or shadows merely because no visual brief was included.
That absence selects the official defaults.

## Scaffold a standalone site

```sh
npm create astro@latest -- --template answer-cloud/answer_cloud_template my-site
cd my-site
npm install
npm run dev
```

`npx degit answer-cloud/answer_cloud_template my-site` or a plain `git clone` is acceptable when
the Astro initializer cannot fetch the template. Do not copy committed `dist/`, `.astro/`, or
`node_modules/` from the upstream repository into the final project; rebuild locally from source.

The template is a style-guide showcase, not finished customer content. Replace the showcase page
with the requested information architecture and copy. Reuse the design system; do not ship the
23-section reference as the customer's home page.

## Upstream files and their roles

- `src/styles/tokens.css` — canonical color, type, spacing, rhythm, layout, radius, shadow, focus,
  status, and motion tokens.
- `src/styles/styleguide.css` — reset, typography, layout primitives, navigation, buttons, forms,
  cards, tables, alerts, metadata, and responsive component behavior.
- `src/layouts/StyleGuide.astro` — page shell, font loading, navigation, and footer patterns.
- `src/pages/index.astro` — component examples and composition reference. Mine it for patterns,
  then replace its demo content.
- `public/assets/answercloud-chat.css` — the matching widget theme. Keep it at the same public
  path and keep all selectors scoped to the supported widget IDs.
- `src/components/Icon.astro` and `src/components/MediaSystem.astro` — reusable icon and media
  examples. Keep only what the actual site uses.
- `AGENTS.md` and `CLAUDE.md` — current upstream constraints. Rewrite the demo-specific project
  description for the generated site, fill real site metadata, and preserve applicable design
  notes.

## Offline baseline

When the upstream repository cannot be fetched, preserve these defaults:

- restrained, technical, editorial visual language;
- Instrument Sans for UI and prose, JetBrains Mono only for code and metadata;
- `#2e7bff` accent and `#0058ef` deep accent, used intentionally rather than everywhere;
- `#1a1b1c` primary ink, `#667085` muted text, white page surfaces, and restrained cream/gray
  alternate surfaces;
- a maximum type size of 32px, reduced to 28px on small screens;
- border- and grid-driven hierarchy, with shadows reserved for floating elements;
- 4px-based spacing, compact radii, visible focus, accessible contrast, responsive stacking, and
  reduced-motion support.

These are defaults, not universal brand rules. Explicit user branding or an existing site's
established system overrides the palette, fonts, and visual tone while accessibility and
interaction completeness remain required.

## Apply the defaults to CMS templates

The CMS template repository owns generation, preview, marker substitution, relocation, and
packaging. Do not replace those mechanics with the standalone style-guide project.

When no parent brand exists:

1. Copy or translate the official tokens into `src/styles/cms.css`.
2. Style the CMS shell, entry template, index template, empty state, tables, code, blockquotes,
   images, `.summary`, `.aeo-related`, and `ul.entries` from the same system.
3. Use the upstream `public/assets/answercloud-chat.css` as the starting chat theme and keep it
   separate from the bundled CMS stylesheet.
4. Keep the CMS preview hub functional and verify the kitchen-sink sample against every styled
   content type.

When a parent site exists, derive the same files from that site's actual design instead.

## Adapt without breaking the system

- Change tokens first; avoid scattered one-off values when a token can express the decision.
- Preserve semantic HTML and real links. Visual composition must not hide answerable content
  behind client-only interactions.
- Keep one clear `h1`, compact readable measure, and the official spacing rhythm.
- Retain component states: default, hover, active, focus, disabled, error, empty, loading, and
  responsive behavior as applicable.
- Keep the chat theme synchronized with the active site tokens. Never import it into the site or
  CMS stylesheet; answer.cloud loads it separately into both the host document and chat iframe.
- Remove unused showcase components and demo media rather than shipping dead reference content.

## Verification

Before handing off a styled site:

1. Confirm the actual pages use the active tokens and components consistently.
2. Confirm no text exceeds the active system's maximum size unless the user explicitly supplied a
   different brand scale.
3. Test desktop and mobile layouts, keyboard focus, contrast, and reduced motion.
4. Build from Astro source and verify the expected HTML and CSS in `dist/`.
5. Confirm `dist/assets/answercloud-chat.css` exists when theming chat and contains no unscoped
   selectors.
6. Confirm the style-guide showcase content, stale placeholder metadata, committed dependencies,
   and upstream build output are absent from the customer artifact.
