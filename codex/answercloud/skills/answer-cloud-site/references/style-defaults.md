# Official answer.cloud style defaults

Use this reference whenever a new answer.cloud site or CMS template needs visual styling. The
upstream source of truth is
[answer-cloud/answer_cloud_template](https://github.com/answer-cloud/answer_cloud_template).
Inspect its current `AGENTS.md` or `CLAUDE.md` and style files when network access is available;
the summary here is the offline baseline, not permission to ignore newer upstream guidance.
The primary standalone template is **SaaaaS**. The former style-guide template is retained
under `archive/classic/` for reference; do not scaffold new sites from that archive by default.

## Choose the right source

| Project context | Visual source |
| --- | --- |
| New standalone site with no supplied brand | The template's structure and semantic tokens, with a palette chosen from the brief or a neutral fallback |
| New standalone site with supplied brand direction | The official template's structure and interaction quality, retokenized to the supplied brand |
| Existing site | The existing site's rendered UI and source tokens; use the official template only as a completeness check |
| CMS or AI companion with an existing parent site | The parent site's design system |
| CMS or AI companion without a usable brand source | The official answer.cloud template defaults applied to the CMS template |

Do not ask a user for colors, fonts, radii, or shadows merely because no visual brief was included.
Choose a fitting palette from the brief or use a neutral fallback. The guide does not require an
Answer Cloud brand accent. Retokenize older template colors before authoring the site.

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

The template contains a SaaS home page and a separate `/style-guide/` page. Its product, plans,
workspace data, and copy are illustrative. Replace the home page with the requested information
architecture and factual copy. Keep the style guide as a development reference; it defaults to
search `noindex,follow` and `answercloud:index=exclude`. Publish it as indexed documentation only
when that is intentional.

An unassigned template builds with `npm run build` and needs no `SITE_URL`. For a derived public
site, set `SITE_URL` to its actual public HTTPS domain origin and run `npm run build:site`. This
validates the origin and emits absolute canonical and social image URLs; never invent a domain.

## Upstream files and their roles

- `src/styles/tokens.css` — canonical palette, typography, spacing, radii, shadows, and motion.
- `src/styles/site.css` and `src/layouts/SiteLayout.astro` — shared page shell, metadata,
  navigation, footer, buttons, forms, focus treatment, and product tour.
- `src/pages/index.astro` and `src/styles/home.css` — the illustrative home page and its
  composition. Replace its sample product information with the site's actual content.
- `src/pages/style-guide.astro` and `src/styles/guide.css` — the separate design-system reference.
- `src/components/Workspace.astro` and `src/styles/workspace.css` — the static hero product
  illustration, exposed as one accessible image. It contains no interactive tabs or task inputs.
- `src/components/SiteMotion.astro` and `src/scripts/motion.ts` — deferred GSAP/ScrollTrigger
  loading, scroll reveals, small workspace drift, and motion lifecycle cleanup.
- `src/components/GradientBackdrop.astro` — three-layer Aurora flow, pause/resume control,
  offscreen and hidden-tab pausing, and reduced-motion behavior.
- `src/components/Icon.astro`, `BrandMark.astro`, and `IntegrationLogo.astro` — reusable line
  icons and precise identity/integration artwork. Keep only what the actual site uses.
- `public/assets/answercloud-chat.css` — the independently loaded widget theme. Keep this path
  and scope all selectors to supported widget IDs. Apply the
  [chat design requirements](chat-widget.md#chat-design-requirements) before reusing it.
- `scripts/build-site.mjs` and `scripts/site-url.mjs` — public-site build and origin validation.
- `AGENTS.md` and `CLAUDE.md` — upstream constraints. Rewrite the demo-specific project identity
  for the generated site, fill real site metadata, and retain applicable design guidance.

## Offline baseline

When the upstream repository cannot be fetched, preserve these defaults:

- a clear SaaS composition with a quiet white canvas, generous space, and aligned product UI;
- self-hosted Inter variable for prose and UI, monospace for token/code labels, and a restrained
  serif wordmark where appropriate to the site's identity;
- charcoal `#171717` primary ink, `#666666` muted text, `#6b6b6b` subtle text, fine `#e8e8e8`
  borders, and black primary actions with white text;
- mint, blue, lilac, and peach surfaces; Aurora, Dawn, Sky, and Meadow gradients for product
  backdrops and color moments while body copy remains on readable surfaces;
- fluid 40–56px display text, 30–36px section headings, 16px body text, and 12–14px interface
  labels, adapted to the viewport rather than a fixed 32px maximum;
- an 1180px content maximum, a 4px spacing rhythm, 6/10/16/24px radii, and restrained shadows
  for product frames and floating UI;
- semantic HTML, real links and controls, visible keyboard focus, accessible contrast, responsive
  composition, and reduced-motion support.

Color and identity are site choices. Replace the illustrative SaaaaS name and answer.cloud mark
when appropriate to the customer's brand. Supplied branding or an existing site's established
system overrides the palette, fonts, visual tone, and motion defaults. Keep accessibility and
usable interactions intact.

## Motion and product illustrations

- Use lightweight CSS hero entrances. Keep the headline visible from first paint; supporting
  text, actions, and product UI may arrive in a short stagger. Keyboard focus completes entrance
  animations without making the focused control move or replaying the entrance on blur.
- Keep the hero workspace a static, polished product illustration, separate from functional
  navigation and product-tour controls. Do not add fake clickable tabs, task inputs, decorative
  oversized logos, handwritten notes, or tilted slogan cards to it.
- Use three overlapping mint/blue/lilac Aurora layers moving through continuous transform loops.
  Retain pause/resume, pause while offscreen or in a hidden tab, and keep the white upper edge.
- Keep GSAP/ScrollTrigger deferred until after page load and idle time or visitor intent. Use
  restrained reveals and a small workspace drift with native scrolling; do not hide substantive
  content in initial HTML or compete with hero entrance transforms.
- Reduced motion disables the CSS entrances, gradient flow, and GSAP download. Preserve cleanup
  and restoration on preference changes, pagehide/pageshow, and back/forward-cache restoration.
  Static content stays readable without JavaScript. Do not restore the archived Three.js hero
  unless the user explicitly asks for that visual direction.

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

- Change tokens first, including accent surface/border/focus tokens; avoid scattered one-off
  colors. Check that hover, focus, links, forms, figures, and chat follow the chosen palette.
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
2. Confirm display and body type use the active responsive scale, with readable line lengths
   and no clipped or overflowing text.
3. Test desktop and mobile layouts, keyboard focus, contrast, reduced motion, and gradient
   pause/resume. Audit the production build with `npm run preview`, not the development server.
4. Build from Astro source and verify the expected HTML and CSS in `dist/`. Include a
   site-specific 1200 × 630 PNG/JPEG share image, Open Graph/Twitter metadata in each static
   page head, and absolute HTTPS image URLs on the actual site origin. Apply the skill's social
   preview checks to CMS templates and companion-only packages too.
5. Confirm `dist/assets/answercloud-chat.css` exists when theming chat and contains no unscoped
   selectors. Apply the chat reference's visual check: readable launcher label, compact messages,
   and a visible composer and controls at desktop and mobile sizes. Verify the injected widget
   after an authorized deploy; report live verification as pending for local-only work.
6. Confirm illustrative product claims, stale placeholder metadata, dependencies, archived
   templates, and upstream build output are absent from the customer artifact. Keep development
   references outside the indexed answer corpus.
