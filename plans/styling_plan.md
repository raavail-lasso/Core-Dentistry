# Styling Plan: Tailwind CSS Setup + Design System

## Context
The Core Dentistry project is a fresh Astro 5.x site with no styling infrastructure. The goal is to install Tailwind CSS and configure it with the typography and brand colors extracted from the existing Webflow site at coredentistrysc.com.

## Extracted Typography from coredentistrysc.com

**Fonts:**
- **Heading**: `Bricolage Grotesque` (self-hosted variable font)
- **Body**: `Inter Variable` (self-hosted variable font)

**Type Scale (desktop):**
| Tag | Size     | Weight | Line Height |
|-----|----------|--------|-------------|
| h1  | 5rem     | 500    | 1.2         |
| h2  | 3.25rem  | 500    | 1.2         |
| h3  | 2.75rem  | 500    | 1.2         |
| h4  | 2rem     | 500    | 1.3         |
| h5  | 1.5rem   | 700    | 1.4         |
| h6  | 1.375rem | 500    | 1.4         |

**Body text sizes:**
- Large: `1.25rem`
- Medium: `1.125rem`
- Regular: `1rem` / line-height `1.5`

**Font weights used:** 400, 500, 600, 700

---

## Approach: Tailwind CSS v4 (CSS-first)

Tailwind v4 uses CSS-first configuration via `@theme` blocks in `src/styles/global.css` instead of a `tailwind.config.js` file. All design tokens — typography and colors — live in one place and automatically become Tailwind utility classes.

**Responsive typography** is handled via CSS `clamp()` on the type scale. Values were derived directly from the Webflow site's mobile and desktop breakpoint sizes, so headings scale smoothly across all viewport widths with no breakpoint classes needed in markup.

---

## Font Hosting

Fonts are self-hosted in `public/fonts/` rather than loaded from Google Fonts, eliminating the external DNS lookup and network round-trip on every page load.

**Files:**
| File | Size | Covers |
|------|------|--------|
| `BricolageGrotesque-Variable.woff2` | 75 KB | wght 200–800, opsz 12–96, latin subset |
| `Inter-Variable.woff2` | 71 KB | wght 100–900, latin subset |

Both are variable fonts — one file per family covers all weights. `@font-face` declarations live at the top of `global.css`, above the `@theme` block. The Google Fonts `<link>` tags have been removed from `GlobalLayout.astro`.

> **Note on naming:** The original Google Fonts `<link>` loaded `Inter` (family name `"Inter"`), but `global.css` referenced it as `"Inter Variable"` — meaning the body font was silently falling back to Arial. The self-hosted `@font-face` declares the family as `'Inter Variable'` to match, fixing the mismatch.

---

## File Structure

```
src/
  styles/
    global.css        ← @font-face declarations + Tailwind entry + all design tokens (@theme)
  layouts/
    GlobalLayout.astro ← No external font links; fonts load from /public/fonts/
public/
  fonts/
    BricolageGrotesque-Variable.woff2
    Inter-Variable.woff2
astro.config.mjs      ← Tailwind Vite plugin registered
netlify.toml          ← Build config for Netlify deployment
```

---

## Design Tokens (`src/styles/global.css`)

### Typography

| Token | Value | Usage |
|-------|-------|-------|
| `--font-heading` | Bricolage Grotesque | `font-heading` |
| `--font-body` | Inter Variable | `font-body` |
| `--text-7xl` | clamp(3.25rem → 5rem) | h1 |
| `--text-5xl` | clamp(2.75rem → 3.25rem) | h2 |
| `--text-4xl` | clamp(2.25rem → 2.75rem) | h3 |
| `--text-3xl` | clamp(1.75rem → 2rem) | h4 |
| `--text-2xl` | 1.5rem | h5 |
| `--text-xl` | 1.375rem | h6 |
| `--text-lg` | 1.25rem | large body |
| `--text-md` | 1.125rem | medium body |
| `--text-base` | 1rem | regular body |

### Color Palette

Each group has 7 shades: darkest → darker → dark → default → light → lighter → lightest.
Values marked `extracted` came directly from the Webflow CSS; `derived` values were interpolated using the same HSL hue/saturation profile.

**Navy** — primary brand (Big Stone blues)
| Token | Hex | Source |
|-------|-----|--------|
| `--color-navy-darkest` | #060b15 | extracted |
| `--color-navy-darker` | #0b1223 | derived |
| `--color-navy-dark` | #121e39 | extracted |
| `--color-navy` | #172648 | extracted |
| `--color-navy-light` | #5c677e | extracted |
| `--color-navy-lighter` | #d0d3da | extracted |
| `--color-navy-lightest` | #f1f2f4 | derived |

**Rose** — primary accent (Turkish Rose pinks)
| Token | Hex | Source |
|-------|-----|--------|
| `--color-rose-darkest` | #492c30 | extracted |
| `--color-rose-darker` | #693f45 | derived |
| `--color-rose-dark` | #925860 | extracted |
| `--color-rose` | #b76e79 | extracted |
| `--color-rose-light` | #f0e2e4 | extracted |
| `--color-rose-lighter` | #f6eeef | derived |
| `--color-rose-lightest` | #fbf9f9 | derived |

**Blush** — secondary accent (Oriental Pink dusty rose)
| Token | Hex | Source |
|-------|-----|--------|
| `--color-blush-darkest` | #3e2d28 | derived |
| `--color-blush-darker` | #6c5047 | derived |
| `--color-blush-dark` | #a07c70 | extracted |
| `--color-blush` | #cbaea4 | derived |
| `--color-blush-light` | #f4ebe8 | extracted |
| `--color-blush-lighter` | #f9f5f3 | extracted |
| `--color-blush-lightest` | #fdfcfc | derived |

**Cream** — warm backgrounds (Pot Pourri)
| Token | Hex | Source |
|-------|-----|--------|
| `--color-cream-darkest` | #484543 | extracted |
| `--color-cream-darker` | #78716d | derived |
| `--color-cream-dark` | #b3a098 | derived |
| `--color-cream` | #f3e6e1 | extracted |
| `--color-cream-light` | #f6edea | extracted |
| `--color-cream-lighter` | #fcfaf9 | extracted |
| `--color-cream-lightest` | #fefcfb | derived |

**Neutral** — pure grays
| Token | Hex | Source |
|-------|-----|--------|
| `--color-neutral-darkest` | #121212 | derived |
| `--color-neutral-darker` | #222222 | extracted |
| `--color-neutral-dark` | #444444 | extracted |
| `--color-neutral` | #666666 | extracted |
| `--color-neutral-light` | #aaaaaa | extracted |
| `--color-neutral-lighter` | #cccccc | extracted |
| `--color-neutral-lightest` | #eeeeee | extracted |

---

## Netlify Deployment

No special configuration needed beyond `netlify.toml`. Tailwind v4 + Vite is entirely build-time — zero runtime overhead. Netlify runs `npm run build`, outputs to `dist/`, done.

---

## Tailwind v4 Syntax Notes

### Arbitrary values vs. native utilities

Tailwind v4 introduced native support for many values that previously required arbitrary value syntax (square brackets). Always prefer the native v4 form — some linters and language servers flag the old bracket syntax as an error.

| v3 (arbitrary) | v4 (native) | Notes |
|----------------|-------------|-------|
| `aspect-[4/5]` | `aspect-4/5` | Aspect ratio |

When in doubt, check the [Tailwind v4 docs](https://tailwindcss.com/docs) before reaching for `[...]` — the native utility likely exists.

---

## Verification
1. Run `npm run dev` — no build errors
2. Add a test element: `<h1 class="font-heading text-7xl text-navy">Test</h1>`
3. Visit `localhost:4321` — verify Bricolage Grotesque renders, scaled by viewport width
4. Inspect in DevTools — confirm `--color-navy` and font variables resolve correctly
5. Remove test element
