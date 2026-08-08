# LEGAS-C — The Fan Cooperative

Marketing and member site for **LEGAS-C**, a member-owned soccer fan cooperative
built on four pillars: Information, Education, Participation, and Promotion.

- **Live:** https://legas-c.netlify.app
- **Netlify project:** https://app.netlify.com/projects/legas-c
- **Repo:** https://github.com/raoul-graham/LegaS-C

---

## Repository layout

```
index.html      the entire site — markup, CSS, JS, and logo, in one file
netlify.toml    publish config, SPA fallback redirect, security headers
README.md       this file
```

There is no build step, no package manager, and no dependencies to install.

## Running locally

Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

A plain `file://` open works for everything except the Google Translate view,
which expects an http(s) origin.

## Deployment

The Netlify project is linked to this repo, so **every push to `main`
auto-deploys**. No manual deploy step.

`netlify.toml` sets:

| Setting | Value | Why |
| --- | --- | --- |
| `publish` | `.` | the site root *is* the deployable output |
| build command | *(none)* | nothing to compile |
| redirect | `/*` → `/index.html` (200) | client-side routing serves any path |
| headers | `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy` | baseline hardening |

---

## Architecture

**Single-file SPA.** All six views live in `index.html` as sibling `.view`
sections; `go(view)` toggles the `.active` class and syncs the nav state. Views:
`home`, `scores`, `news`, `pillars`, `register`, `translate`.

**Persisted state** (`localStorage`):

- `lc-theme` — `light` / `dark`
- `lc-favs` — favorited teams shown in the scores sidebar

### The logo

One SVG lives in `HEART_LOGO_SRC` as a data URI and is injected into every
`img.heart-logo` on load. **All six placements share that one asset**, so the
mark cannot drift out of sync — change the SVG and everywhere updates.

The mark is a two-tone heart-shaped tree canopy whose trunk reads as **"1L"**.
The trunk is drawn *before* the canopy so the legs join the base cleanly at any
size, rather than depending on per-instance text offsets.

Two ink variants, swapped by `refreshHeartLogos()` on every theme change:

| Variant | Ink | Used by |
| --- | --- | --- |
| `HEART_LOGO_SRC` | black | light theme, **and** anything tagged `.heart-logo-fixed` |
| `HEART_LOGO_SRC_DARK` | white | dark theme |

`.heart-logo-fixed` marks logos sitting on chrome that is hardcoded white in
*both* themes (the chat bubble and its avatar). They must keep black ink or they
would vanish. Any new logo on a permanently-light surface needs this class.

### The wordmark

Colors are defined once as `.wm-*` classes — never inline — so the three
lockups (hero, nav, chat header) cannot drift apart again:

| Class | Light | Dark surfaces |
| --- | --- | --- |
| `.wm-lega`, `.wm-s` — "LEGAS" | `#1B5E3A` green | `#3FAF7D` |
| `.wm-hyphen` — "-" | `#0D0D0D` ink | `#FFFFFF` |
| `.wm-c` — "C" | `#1B3358` navy | `#6D9FE0` |

Dark surfaces use lightened equivalents because an ink-black hyphen is invisible
on the dark hero and on the always-dark chat header.

### Palette

| Role | Hex |
| --- | --- |
| Brand green | `#1B5E3A` |
| Brand navy | `#1B3358` |
| Brand rose (accent, logo dashes) | `#B5305A` |
| UI teal | `#007A6E` |
| UI pink | `#C8274A` |

### Responsive

Breakpoints at **1024 / 900 / 768 / 600 / 400 px**.

The desktop nav links hide at ≤1024px, so the fixed bottom nav bar takes over
across that whole range — **not** only at ≤768px. If you change one of those
thresholds, change the other to match, or tablets lose all navigation.

Because `.mob-nav` is a `<nav>` element, it also has to explicitly reset
`top`/`height`/`border-bottom`/`box-shadow` from the generic `nav {}` rule, or
it renders pinned to the top of the screen.

---

## External services

| Service | Used for | Notes |
| --- | --- | --- |
| Google Fonts | Barlow, Barlow Condensed | |
| ESPN site API | live scores and news | via public CORS proxies |
| Wikipedia | club badges | hotlinked |
| flagcdn.com | country flags | |
| Unsplash | photography | hotlinked |
| Google Translate | the translate view | needs an http(s) origin |
| Anthropic API | AI assistant | **not wired up — see below** |

Score/news requests try four public CORS proxies in sequence
(`corsproxy.io`, `allorigins`, `codetabs`, `thingproxy`) with a 7s timeout and
in-memory caching. Static match data in the HTML is what renders until a fetch
resolves — and what remains if all four proxies fail.

---

## Known limitations

**The AI assistant does not work.** Both the news-page "Ask LEGAS-C AI" panel
and the chat widget `POST` to `api.anthropic.com` with no credentials, so every
request fails and users always see "Connection error."

Do **not** fix this by putting an API key in `index.html` — the file is public,
and the key would be readable by anyone. The correct fix is a Netlify Function
that holds the key server-side and proxies the request, with the front end
calling that function instead. The pinned model id (`claude-sonnet-4-6`) should
be refreshed at the same time.

**Other things worth knowing:**

- `legas-c.com` is referenced as the canonical domain in `<head>` and by the
  translate view, but is not configured yet — add it under the Netlify
  project's Domain management.
- Content is hardcoded markup. Anything genuinely live (scores, news, ticker)
  is fine to leave to the ESPN fetch; anything editorial requires editing HTML
  and pushing. If non-developers will be maintaining copy, a headless CMS
  (Decap CMS integrates cleanly with Netlify) is the natural next step.
- Third-party images are hotlinked and can break without warning.
