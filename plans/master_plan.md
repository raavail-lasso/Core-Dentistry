# Astro 5 + Tailwind v4 + Netlify — Framework Playbook

This is a reusable starting-point reference for static marketing sites built with **Astro 5**, **Tailwind CSS v4**, and **Netlify** hosting. It covers the full stack: design system setup, layout architecture, component patterns, image standards, build optimization, security headers, and accessibility. Before writing any code, fill in every `[PLACEHOLDER]` value using the reference table in Section 10.

**What this document covers:**
- Design system (tokens, typography, color scale)
- Layout and SEO architecture
- Component patterns (variants, CTAs, forms, nav, footer)
- Image and asset standards
- Build and performance optimization
- Security headers and CSP
- Accessibility rules and checklists
- Deployment checklist

---

# 1. Project Setup and File Structure

## 1.1 Prerequisites

- Node.js (LTS version)
- Create a new Astro project: `npm create astro@latest`
- Install Tailwind for Astro: `npm install tailwindcss @tailwindcss/vite`

## 1.2 astro.config.mjs Starter Template

This configuration applies to every project. The three build options — inline stylesheets, rollup CSS naming, and the Tailwind Vite plugin — should be present from day one.

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  build: {
    inlineStylesheets: 'always',
  },
  vite: {
    plugins: [tailwindcss()],
    build: {
      rollupOptions: {
        output: {
          assetFileNames: (assetInfo) => {
            if (assetInfo.names?.[0]?.endsWith('.css')) {
              return '_astro/styles.[hash][extname]';
            }
            return '_astro/[name].[hash][extname]';
          },
        },
      },
    },
  },
});
```

See Section 6 for why each of these options is included.

## 1.3 Canonical Directory Layout

```
src/
  styles/
    global.css          ← @font-face declarations + Tailwind entry + @theme design tokens
  layouts/
    GlobalLayout.astro  ← shared page shell (head, navbar, footer, scripts)
  components/
    Navbar.astro
    Footer.astro
    CTAGroup.astro
    ContactForm.astro
    [other components]
  pages/
    index.astro
    [service pages].astro
public/
  fonts/
    [HEADING_FONT].woff2
    [BODY_FONT].woff2
  images/
    favicon.png
    webclip.png
    [logo].webp
    [hero images].webp
astro.config.mjs
netlify.toml
```

---

# 2. Design System

## 2.1 Font Selection and Self-Hosting

Choose two variable fonts:
- **Heading font** — a display that reflects the brand personality
- **Body font** — a text-optimized variable font for readability

Self-host both as `.woff2` variable fonts in `public/fonts/`. This eliminates external DNS lookups and the network round-trip that Google Fonts requires on every page load.

> **Critical:** The `font-family` name declared in `@font-face` must exactly match the name referenced in `@theme`. A mismatch causes a silent fallback to system fonts — the page loads, the font appears correct in development (if the font is cached), and the issue only surfaces when auditing the network tab.

## 2.2 The @font-face Pattern

Declare all `@font-face` blocks at the top of `global.css`, above the `@theme` block. Adjust `font-weight` to match the variable font's actual axis range (check the font foundry's documentation).

```css
@font-face {
  font-family: '[HEADING_FONT]';
  font-style: normal;
  font-weight: 200 800;   /* match the variable font's wght axis range */
  font-display: swap;
  src: url('/fonts/[HEADING_FONT].woff2') format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6,
    U+02DA, U+02DC, U+0304, U+0308, U+0329, U+2000-206F, U+20AC,
    U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}

@font-face {
  font-family: '[BODY_FONT]';
  font-style: normal;
  font-weight: 100 900;   /* match the variable font's wght axis range */
  font-display: swap;
  src: url('/fonts/[BODY_FONT].woff2') format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6,
    U+02DA, U+02DC, U+0304, U+0308, U+0329, U+2000-206F, U+20AC,
    U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
```

## 2.3 @theme Token Structure

Tailwind v4 replaces `tailwind.config.js` with a `@theme` block inside CSS. Every custom property declared inside `@theme` automatically becomes a Tailwind utility class (e.g., `--font-heading` → `font-heading`, `--color-navy` → `text-navy`, `bg-navy`).

```css
@import "tailwindcss";

/* @font-face declarations go here, above @theme */

@theme {
  /* ── Typography ─────────────────────────────── */
  --font-heading: "[HEADING_FONT]", Arial, sans-serif;
  --font-body:    "[BODY_FONT]", Arial, sans-serif;

  /* Type scale — fluid via clamp() */
  --text-7xl: /* h1 */ clamp([mobile-size], [fluid], [desktop-size]);
  --text-5xl: /* h2 */ clamp([mobile-size], [fluid], [desktop-size]);
  --text-4xl: /* h3 */ clamp([mobile-size], [fluid], [desktop-size]);
  --text-3xl: /* h4 */ clamp([mobile-size], [fluid], [desktop-size]);
  --text-2xl: /* h5 */ [fixed-size];
  --text-xl:  /* h6 */ [fixed-size];
  --text-lg:  /* large body */ [size];
  --text-md:  /* medium body */ [size];

  /* ── Colors ─────────────────────────────────── */
  /* [PRIMARY_COLOR] group */
  /* [ACCENT_COLOR] group */
  /* [SECONDARY_ACCENT] group — if applicable */
  /* [WARM_BACKGROUND] group — if applicable */
  /* neutral group */
}
```

## 2.4 Typography Scale

Use CSS `clamp()` for all heading sizes. Derive the minimum from the mobile design and the maximum from the desktop design. This eliminates responsive breakpoint classes on every heading in markup — headings scale fluidly across all viewport widths.

**Recommended starting scale** (adjust to match the client's brand):

| Token | Clamp Range | Element |
|-------|-------------|---------|
| `--text-7xl` | clamp(3.25rem → 5rem) | h1 |
| `--text-5xl` | clamp(2.75rem → 3.25rem) | h2 |
| `--text-4xl` | clamp(2.25rem → 2.75rem) | h3 |
| `--text-3xl` | clamp(1.75rem → 2rem) | h4 |
| `--text-2xl` | 1.5rem | h5 |
| `--text-xl`  | 1.375rem | h6 |
| `--text-lg`  | 1.25rem | large body |
| `--text-md`  | 1.125rem | medium body |
| `--text-base`| 1rem | regular body |

**Line heights:** `1.2` for large headings (h1–h3), `1.3–1.4` for smaller headings, `1.5` for body text.

**Weights used:** 400 (regular body), 500 (heading default), 600 (semi-bold emphasis), 700 (bold labels and small caps).

## 2.5 Color Palette Methodology — The 7-Shade Scale

Each brand color group follows the same naming convention:

```
[color]-darkest  →  [color]-darker  →  [color]-dark  →  [color]  →  [color]-light  →  [color]-lighter  →  [color]-lightest
```

**How to build the palette:**
1. Extract the core brand colors from the client's existing site or brand guide. Mark each as `extracted`.
2. For each extracted color, derive the missing shades by maintaining the same HSL hue/saturation profile at stepped lightness values. Mark each as `derived`.
3. A typical project needs:
   - **Primary brand color** (often a dark navy or deep neutral) — used for backgrounds, buttons, text on light
   - **Primary accent** (often a warm or vibrant color) — used for CTAs, icons, highlights
   - **Secondary accent** (a muted or complementary tone) — optional; for section differentiation
   - **Warm background** (a cream or off-white) — optional; for section backgrounds
   - **Neutral grays** — for text, borders, and subtle backgrounds

**CSS token template:**

```css
/* [PRIMARY_COLOR] — [name / description] */
--color-[primary]-darkest: [HEX]; /* extracted | derived */
--color-[primary]-darker:  [HEX];
--color-[primary]-dark:    [HEX];
--color-[primary]:         [HEX]; /* default — the core brand color */
--color-[primary]-light:   [HEX];
--color-[primary]-lighter: [HEX];
--color-[primary]-lightest:[HEX];

/* [ACCENT_COLOR] — [name / description] */
--color-[accent]-darkest: [HEX];
/* ...repeat pattern... */
```

**The contrast-safe floor:**
After defining each accent color scale, identify the darkest shade of the accent that still reads as accent rather than as a dark background tone. Verify that shade passes **4.5:1 contrast ratio** against white and against the lightest warm background in the palette. Document this shade explicitly — it is the minimum safe shade for text in light-background sections. The default accent shade (the mid-point) is often too light for body text on white; `-dark` is usually the safe floor.

## 2.6 Tailwind v4 Syntax Notes

Tailwind v4 introduced native support for many values that previously required arbitrary value syntax. Always prefer the native v4 form — some linters flag the bracket syntax as an error.

| v3 (arbitrary) | v4 (native) |
|----------------|-------------|
| `aspect-[4/5]` | `aspect-4/5` |

When in doubt, check the [Tailwind v4 docs](https://tailwindcss.com/docs) before reaching for `[...]`.

## 2.7 Verification

- [ ] `npm run dev` runs with no errors
- [ ] Test element `<h1 class="font-heading text-7xl text-[primary-color]">Test</h1>` renders in the heading font at the correct size
- [ ] DevTools confirms CSS custom properties resolve correctly (`--color-[primary]`, `--font-heading`, etc.)
- [ ] No fallback system fonts appear (check Network tab → Fonts — should list woff2 files, not "system-ui")
- [ ] Remove test element before committing

---

# 3. Layout Architecture

## 3.1 GlobalLayout Pattern

Centralise every piece of markup shared across all pages into a single `GlobalLayout.astro` component. Individual page files contain only their unique section content. Any new page wraps its content in `<GlobalLayout>` — no need to duplicate `<html>`, `<head>`, `<body>`, Navbar, or Footer.

**What GlobalLayout must contain:**

| Responsibility | Details |
|---|---|
| `<html lang="en">` / `<head>` | charset, favicons, viewport, generator meta |
| `<title>` | set via `title` prop |
| `<meta name="description">` | set via `description` prop — required for SEO |
| Font preloading | `<link rel="preload">` for each self-hosted `.woff2` file |
| Global CSS | `import '../styles/global.css'` in frontmatter |
| `<body>` base classes | background color and body font class |
| `<Navbar />` | rendered at the top of every page |
| `<main>` + `<slot />` | page-specific content injection point |
| `<Footer />` | rendered at the bottom of every page |
| Inline `<script is:inline>` | lazy-loading and Trusted Types (see Sections 6.4 and 7.5) |

## 3.2 Props Interface

| Prop | Type | Default | Description |
|---|---|---|---|
| `title` | `string` | `'[SITE_NAME] — [DEFAULT_LOCATION_OR_TAGLINE]'` | Sets `<title>` — always override per page |
| `description` | `string` | `'[DEFAULT_META_DESCRIPTION]'` | Sets `<meta name="description">` — always override per page |

> Never rely on the default fallbacks in production. Every page must pass its own unique `title` and `description`.

## 3.3 GlobalLayout Template

```astro
---
import '../styles/global.css';
import Navbar from '../components/Navbar.astro';
import Footer from '../components/Footer.astro';

interface Props {
  title?: string;
  description?: string;
}

const {
  title = '[SITE_NAME] — [DEFAULT_LOCATION_OR_TAGLINE]',
  description = '[DEFAULT_META_DESCRIPTION]',
} = Astro.props;
---

<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" type="image/png" href="/images/favicon.png" />
    <link rel="apple-touch-icon" href="/images/webclip.png" />
    <meta name="viewport" content="width=device-width" />
    <meta name="description" content={description} />
    <meta name="generator" content={Astro.generator} />
    <link rel="preload" as="font" type="font/woff2" crossorigin href="/fonts/[BODY_FONT].woff2" />
    <link rel="preload" as="font" type="font/woff2" crossorigin href="/fonts/[HEADING_FONT].woff2" />
    <title>{title}</title>
  </head>
  <body class="bg-white font-body">
    <Navbar />
    <main>
      <slot />
    </main>
    <Footer />
    <script is:inline>
      /* Lazy-load + Trusted Types inline script — see Sections 6.4 and 7.5 */
    </script>
  </body>
</html>
```

## 3.4 How Pages Consume the Layout

Pages import only `GlobalLayout` and the components they actually use. The `<html>`, `<head>`, `<body>`, `<Navbar>`, and `<Footer>` are not present in page files.

```astro
---
import GlobalLayout from '../layouts/GlobalLayout.astro';
import CTAGroup from '../components/CTAGroup.astro';
import ContactForm from '../components/ContactForm.astro';
---

<GlobalLayout
  title="[Page-specific title | Site Name]"
  description="[120–160 character unique description for this page]"
>
  <section>...</section>
  <section>...</section>
</GlobalLayout>
```

## 3.5 Adding a New Page — Checklist

1. Create `src/pages/[page-name].astro`
2. Import `GlobalLayout` and any page-specific components
3. Wrap all content in `<GlobalLayout title="..." description="...">`
4. Pass unique, production-ready values for both props — do not rely on layout defaults

## 3.6 SEO Requirements

Every page must satisfy these rules. Google PageSpeed Insights audits them under the SEO category.

### Meta title and description rules

- **Title:** under 60 characters — Google truncates longer titles in SERPs
- **Description:** 120–160 characters — Google truncates longer descriptions
- Include the primary service and location keywords (city, region)
- Write for the searcher — a natural, compelling summary, not a keyword list
- Must be **unique per page** — duplicate titles/descriptions provide no SEO value
- Never rely on layout default fallbacks in production

### New page SEO checklist

- [ ] `title` prop set — unique, under 60 characters
- [ ] `description` prop set — unique, 120–160 characters
- [ ] Page has exactly one `<h1>`
- [ ] All images have a meaningful `alt` attribute (or `alt=""` if purely decorative)

---

# 4. Component Patterns

## 4.1 Design System Conventions

All components follow these shared rules:

- **Variants:** Every component accepts a `variant` prop — `'default'` for light backgrounds, `'on-dark'` for dark/primary-color backgrounds
- **Tokens:** All color, font, and spacing classes come from `@theme` tokens (e.g. `bg-[primary]`, `text-[accent]`, `font-body`)
- **Shape:** Pill shape (`rounded-full`) for all button-style components
- **Transitions:** `transition-colors duration-200` on all interactive elements
- **Typography:** `font-body` for UI controls and labels; `font-heading` reserved for display and hero text

## 4.2 The Variant Pattern

The `variant` prop is the primary mechanism for adapting a component to its background context. Every significant component should implement it.

```astro
---
interface Props {
  variant?: 'default' | 'on-dark';
}

const { variant = 'default' } = Astro.props;
const isOnDark = variant === 'on-dark';
---

<element
  class:list={[
    'base-classes-shared-by-both-variants',
    isOnDark
      ? 'bg-white text-[primary] hover:bg-[primary]-lightest'
      : 'bg-[primary] text-white hover:bg-[primary]-dark',
  ]}
>
  content
</element>
```

**Typical variant patterns:**

| Variant | Button (primary action) | Button (secondary action) |
|---|---|---|
| `default` (light bg) | `bg-[primary] text-white hover:bg-[primary]-dark` | `bg-[accent]-dark text-white hover:bg-[accent]-darker` |
| `on-dark` (dark bg) | `bg-white text-[primary] hover:bg-[primary]-lightest` | transparent + `border-white text-white hover:bg-white hover:text-[primary]` |

## 4.3 CTA Button Components

Most service or local-business sites have two CTA button types:

**Primary contact button** (e.g. a phone CTA):
- Renders as `<a href="tel:[CONTACT_PHONE]">`
- Displays a stacked two-line label (e.g. "CALL NOW" + formatted phone number)
- Phone icon: inline SVG with `aria-hidden="true"` alongside a visually hidden label
- Contact number is hardcoded internally — not passed as a prop

**Primary action button** (e.g. an appointment or booking CTA):
- Renders as `<a href="[BOOKING_HREF]">` (can be a form anchor or booking tool link)
- Displays a label with a right-pointing arrow icon (`aria-hidden="true"`)
- Label and href are hardcoded internally
- `on-dark` variant uses a ghost/outlined style that fills on hover

**Rule:** Hardcode critical contact details and labels inside CTA components rather than passing them as props. The phone number and booking link are business constants — they don't vary per-instance, so prop complexity adds no value.

## 4.4 CTA Group Wrapper

A `CTAGroup` component composes the two CTA buttons side by side. It:
- Accepts a `variant` prop and passes it through to both children
- Exposes `showPrimaryAction` and `showSecondaryAction` boolean props (default `true`) so individual buttons can be hidden at call-site

```astro
---
import PhoneButton from './PhoneButton.astro';
import ActionButton from './ActionButton.astro';

interface Props {
  variant?: 'default' | 'on-dark';
  showPhoneButton?: boolean;
  showActionButton?: boolean;
}

const {
  variant = 'default',
  showPhoneButton = true,
  showActionButton = true,
} = Astro.props;
---

<div class="flex items-center gap-3">
  {showPhoneButton && <PhoneButton {variant} />}
  {showActionButton && <ActionButton {variant} />}
</div>
```

Usage:

```astro
<!-- Both buttons, light background -->
<CTAGroup />

<!-- Both buttons, dark background -->
<CTAGroup variant="on-dark" />

<!-- Phone button only -->
<CTAGroup showActionButton={false} />
```

## 4.5 Contact Form

Standard four-field contact form wired to a backend form service (Basin, Netlify Forms, Formspree, etc.).

**Standard fields:**

| Field | Input type | `name` | Required |
|-------|-----------|--------|---------|
| Name | `text` | `name` | Yes |
| Phone | `tel` | `phone` | No |
| Email | `email` | `email` | Yes |
| Message | `textarea` | `message` | No |

**Implementation notes:**
- `action` attribute is `[FORM_ENDPOINT]`
- Focus ring: `focus:ring-[accent]/20` (matches brand accent)
- Textarea: `resize-none` to preserve layout
- Form service attributes (e.g. Basin's `data-basin-form`, `data-spam-protection`) go on the `<form>` element
- The form endpoint is hardcoded internally; `<ContactForm />` takes no props

**Example (Basin):**
```html
<form
  action="[FORM_ENDPOINT]"
  method="POST"
  data-basin-form="true"
  data-basin-success-action="redirect"
  data-spam-protection="recapture"
>
  ...
</form>
```

See Section 6.4 for lazy-loading the form service script.

## 4.6 Navbar

- `<header>` element: `sticky top-0 z-50` with a light bottom border
- White background — `CTAGroup` uses `variant="default"` (primary-color phone button, accent action button)
- Logo on the left as `<img>` with explicit `width` and `height` (see Section 5 on CLS)
- Inner `<nav>` constrained to `max-w-7xl` with responsive horizontal padding
- `CTAGroup` on the right

## 4.7 Footer

- Dark primary-color background (`bg-[primary]-dark`) with a matching top border
- Logo on the left as `<img>` with explicit `width` and `height`
- Contact details on the right: address (links to Google Maps), phone (`<a href="tel:[CONTACT_PHONE]">`), email (`<a href="mailto:[CONTACT_EMAIL]">`)
- Contact icons in accent color against the dark background
- Bottom bar: copyright line separated by a border rule
- All contact details (`[CONTACT_PHONE]`, `[CONTACT_EMAIL]`, `[CONTACT_ADDRESS]`, `[MAPS_HREF]`) are hardcoded in the component — update them there when practice details change

## 4.8 Planned Components — Tracking Convention

Use a `- [ ]` checklist to track components not yet built:

```markdown
## Planned Components
- [ ] **HeroSection** — Full-width hero with headline, subtext, and CTAGroup
- [ ] **ServiceCard** — Individual service feature card
- [ ] **TestimonialCard** — Patient review card
```

Check items off as they are built. This makes the plan file an accurate reflection of what is and isn't implemented.

---

# 5. Image and Asset Standards

## 5.1 Directory Structure

All static assets live in `public/images/` and are served at `/images/<filename>`. Reference them with root-relative paths (no `../`):

```html
<img src="/images/filename.webp" ... />
```

## 5.2 Image Optimization Rules

These rules apply to every image on every project. Following them consistently produces clean Google PageSpeed scores and avoids rework.

**1. Format — always prefer WebP**
- Use `.webp` for all photos and UI images (logos, heroes, illustrations)
- Use `.png` only when true transparency is required and WebP transparency is insufficient (rare — favicons, webclips)
- Never use `.jpg`/`.jpeg` or `.gif` for new assets; convert existing ones to `.webp` when they surface in a PageSpeed audit

**2. Bit depth — use 8-bit WebP**
- Export WebP at 8-bit colour depth, never 16-bit
- 16-bit WebP files are significantly larger with no visible quality improvement for web use
- Google PageSpeed explicitly flags oversized images caused by unnecessary bit depth

**3. Dimensions — match the render size**
- Provide an image whose intrinsic dimensions are as close as possible to the largest size it will ever render
- Do not supply a 2000px wide image for an element that never exceeds 440px
- For retina/HiDPI screens, up to 2× the CSS render size is acceptable; beyond 2× is wasteful
- Always set explicit `width` and `height` HTML attributes to match the intrinsic dimensions — this prevents layout shift (CLS)

**4. Compression — verify before committing**
- Run every new image through Squoosh, ImageOptim, or `cwebp` before adding it to the repo
- Target: under **100 KB** for logos and icons; under **200 KB** for full-bleed hero images
- If PageSpeed flags "Serve images in next-gen formats" or "Properly size images", re-export and replace

**5. `<img>` attributes — required every time**
- `src` — always `/images/<filename>` (root-relative)
- `alt` — always a short, meaningful description (never empty for meaningful images)
- `width` / `height` — always the intrinsic pixel dimensions of the file (not the CSS render size)
- CSS `class` controls rendered size; `width`/`height` attributes prevent layout shift

**6. Filenames — no spaces**
- Use hyphens: `hero-dentist.webp`, not `hero dentist.webp`
- Spaces require `%20` encoding in `src` attributes and are a common source of broken references

## 5.3 Required img Attributes

```html
<img
  src="/images/[filename].webp"
  alt="[Descriptive text]"
  width="[intrinsic-px-width]"
  height="[intrinsic-px-height]"
  class="[rendered-size-classes]"
/>
```

To verify intrinsic dimensions on macOS: `sips -g pixelWidth -g pixelHeight <file>`

## 5.4 Favicon and Webclip

Both files go in `public/images/`. Reference them in `GlobalLayout.astro`:

```html
<link rel="icon" type="image/png" href="/images/favicon.png" />
<link rel="apple-touch-icon" href="/images/webclip.png" />
```

## 5.5 Asset Inventory

Fill this table in as assets are added to the project:

**Logo:**

| File | Dimensions | Path | Used In |
|------|-----------|------|---------|
| `[logo].webp` | [W]×[H] px | `/images/[logo].webp` | `Navbar.astro`, `Footer.astro` |

**Hero Images:**

| File | Path | Page |
|------|------|------|
| `[filename].webp` | `/images/[filename].webp` | `src/pages/[page].astro` |

## 5.6 Adding a New Asset — Checklist

1. Export as **8-bit WebP** at the correct intrinsic dimensions
2. Compress with Squoosh, ImageOptim, or `cwebp` — verify file size
3. Use a **hyphenated filename** with no spaces
4. Drop the file into `public/images/`
5. Reference it as `/images/<filename>` in the `src` attribute
6. Add explicit `width` and `height` attributes matching the intrinsic pixel dimensions
7. Write a meaningful `alt` attribute
8. Document the asset in Section 5.5

---

# 6. Build and Performance Optimization

## 6.1 Inline CSS — Eliminate Render-Blocking Stylesheets

By default, Astro/Vite emits a separate `.css` file linked via `<link rel="stylesheet">`. The browser must fetch this file before it can render anything — a render-blocking resource.

Setting `build.inlineStylesheets: 'always'` embeds all CSS directly in a `<style>` tag in the HTML. A `<style>` tag is parsed inline and never blocks rendering.

```js
// astro.config.mjs
build: {
  inlineStylesheets: 'always',
},
```

**Before:** Browser fetches `/_astro/styles.[hash].css` before first paint.
**After:** CSS is in the HTML document — no extra network round-trip, no render block.

> This is always the right choice for a static site with a reasonable CSS bundle size (under ~50 KB). Keep `inlineStylesheets: 'always'` unless profiling reveals a specific reason to change it.

## 6.2 CSS Output Naming

By default, Astro/Vite names the CSS bundle after the first page it processes alphabetically. This produces confusing output filenames (e.g. the global stylesheet named after a service page).

The `assetFileNames` Rollup option forces the CSS output to use the name `styles`:

```js
// astro.config.mjs
assetFileNames: (assetInfo) => {
  if (assetInfo.names?.[0]?.endsWith('.css')) {
    return '_astro/styles.[hash][extname]';
  }
  return '_astro/[name].[hash][extname]';
},
```

**Note:** This rule has no effect while `inlineStylesheets: 'always'` is active. Keep it anyway — it applies if the mode is ever changed.

## 6.3 Font Preloading — Break the CSS-to-Font Dependency Chain

Without preloading, the browser must download and parse CSS before it can discover the `@font-face` declarations and begin fetching font files. This creates a sequential dependency chain:

**Without preload:** HTML → CSS → fonts discovered → fonts downloaded
**With preload:** HTML → CSS + fonts download in parallel

Add `<link rel="preload">` tags to `<head>` in `GlobalLayout.astro` for each self-hosted font:

```html
<link rel="preload" as="font" type="font/woff2" crossorigin href="/fonts/[BODY_FONT].woff2" />
<link rel="preload" as="font" type="font/woff2" crossorigin href="/fonts/[HEADING_FONT].woff2" />
```

> **`crossorigin` is required** even for same-origin fonts when using `<link rel="preload" as="font">`. Omitting it causes the browser to fetch the font twice — once for the preload and once when the CSS `@font-face` is processed.

## 6.4 Lazy-Loading Third-Party Scripts via IntersectionObserver

Third-party form scripts (Basin, reCAPTCHA, Formspree, etc.) contribute significantly to main-thread blocking time and long tasks. These scripts are only needed when the user is about to interact with a form — there is no reason to load them at page start.

Use an `IntersectionObserver` in the inline script to inject the third-party script only when the form element scrolls within 200px of the viewport:

```html
<script is:inline>
  (function () {
    var loaded = false;
    var observer = new IntersectionObserver(function (entries) {
      if (entries[0].isIntersecting && !loaded) {
        loaded = true;
        var s = document.createElement('script');
        s.src = '[THIRD_PARTY_SCRIPT_URL]';
        document.head.appendChild(s);
        observer.disconnect();
      }
    }, { rootMargin: '200px' });
    var trigger = document.querySelector('[FORM_ELEMENT_SELECTOR]');
    if (trigger) observer.observe(trigger);
  })();
</script>
```

- `rootMargin: '200px'` — gives the script 200px of lead time before the form is visible, so it is initialised before the user reaches it
- The `loaded` guard prevents double-injection if the observer fires multiple times
- The `if (trigger)` guard prevents console errors on pages without a form

**Before:** Script loads on every page, every visit, at page start.
**After:** Script loads only when a page with a form is scrolled near the form section.

> This script is the `<script is:inline>` block in `GlobalLayout.astro`. Its exact content (including whitespace) determines the SHA-256 hash for the CSP. See Section 7.6.

## 6.5 CLS Prevention — Explicit Image Dimensions

Without explicit `width` and `height` HTML attributes, the browser cannot reserve space for an image before it downloads. This causes a layout shift (CLS) when the image loads — the page reflows around it, which PageSpeed flags and which is jarring for users.

**Rule:** Every `<img>` must have `width` and `height` attributes matching the intrinsic pixel dimensions of the source file. Tailwind classes control the rendered size; the HTML attributes prevent layout shift.

```html
<!-- WRONG — no reserved space, causes CLS -->
<img src="/images/logo.webp" alt="Logo" class="h-10 w-auto" />

<!-- CORRECT — browser reserves 440×60 space before image loads -->
<img src="/images/logo.webp" alt="Logo" width="440" height="60" class="h-10 w-auto" />
```

---

# 7. Security Headers

## 7.1 Implementation: netlify.toml

All headers are set in `netlify.toml` using a `[[headers]]` block. For any static Astro site on Netlify, this is the correct approach — no middleware or server code needed. Headers are enforced at the CDN edge with zero performance cost.

**PageSpeed audit items resolved by this section:**
- No CSP found in enforcement mode ✅
- No COOP header found ✅
- No frame control policy found ✅
- No Trusted Types directive found ✅

## 7.2 netlify.toml Template

Replace `[INLINE_SCRIPT_HASH]` with the computed value (see Section 7.6) and `[FORM_ENDPOINT_DOMAIN]` with the domain of your form service.

```toml
[[headers]]
  for = "/*"
  [headers.values]
    X-Content-Type-Options = "nosniff"
    X-Frame-Options = "DENY"
    Referrer-Policy = "strict-origin-when-cross-origin"
    Cross-Origin-Opener-Policy = "same-origin"
    Permissions-Policy = "camera=(), microphone=(), geolocation=()"
    Content-Security-Policy = "default-src 'self'; script-src 'self' 'sha256-[INLINE_SCRIPT_HASH]' 'strict-dynamic' 'unsafe-inline' https: http:; style-src 'self' 'unsafe-inline'; font-src 'self'; img-src 'self' data:; form-action https://[FORM_ENDPOINT_DOMAIN]; frame-src https://www.google.com; frame-ancestors 'none'; connect-src 'self' https://[FORM_ENDPOINT_DOMAIN] https://www.google.com; base-uri 'self'; require-trusted-types-for 'script'; trusted-types default goog#html"
```

## 7.3 Header-by-Header Reference

### X-Content-Type-Options
```
X-Content-Type-Options: nosniff
```
Prevents browsers from MIME-sniffing a response away from its declared content type. Stops attacks where a disguised file is executed as a different type (e.g. an image treated as a script). **Always include. No downsides.**

---

### X-Frame-Options
```
X-Frame-Options: DENY
```
Prevents the site from being embedded in an `<iframe>` on any other domain, blocking clickjacking attacks. `DENY` is stricter than `SAMEORIGIN`.

**Note:** This is a legacy header. Modern browsers use `frame-ancestors` in the CSP. Include both — XFO covers old browsers that don't support CSP.

---

### Referrer-Policy
```
Referrer-Policy: strict-origin-when-cross-origin
```
Sends the full URL for same-origin requests but only the origin (no path) for cross-origin requests — protecting any sensitive URL parameters from leaking to third parties. **Always include. No downsides.**

---

### Cross-Origin-Opener-Policy (COOP)
```
Cross-Origin-Opener-Policy: same-origin
```
Isolates the browsing context so cross-origin windows can't access the `window` object. Prevents cross-origin attacks via shared browsing context references. **Always include for sites that don't deliberately open cross-origin popups.**

---

### Permissions-Policy
```
Permissions-Policy: camera=(), microphone=(), geolocation=()
```
Explicitly disables browser features the site doesn't use. An empty `()` denies the feature for all origins including the page itself. Prevents compromised third-party scripts from silently accessing hardware. **Adjust based on what the site actually uses.**

---

## 7.4 CSP Deep Dive

The Content Security Policy is the most complex header. This is the Google-recommended modern CSP pattern for sites with inline scripts and dynamically-loaded third-party scripts.

```
default-src 'self';
script-src 'self' 'sha256-[HASH]' 'strict-dynamic' 'unsafe-inline' https: http:;
style-src 'self' 'unsafe-inline';
font-src 'self';
img-src 'self' data:;
form-action https://[FORM_ENDPOINT_DOMAIN];
frame-src https://www.google.com;
frame-ancestors 'none';
connect-src 'self' https://[FORM_ENDPOINT_DOMAIN] https://www.google.com;
base-uri 'self';
require-trusted-types-for 'script';
trusted-types default goog#html
```

### `script-src` — The Most Important Directive

| Token | Effect | Why |
|---|---|---|
| `'self'` | Allows scripts from the same origin | Covers any JS files in `/_astro/` |
| `'sha256-HASH'` | Allows the specific inline script by content hash | The secure way to permit inline scripts |
| `'strict-dynamic'` | Any script loaded by a trusted (hashed) script is also trusted | Eliminates the need to list external CDN domains |
| `'unsafe-inline'` | Fallback for old browsers (ignored when hashes are present) | Backward compat only |
| `https: http:` | Fallback for browsers that support schemes but not `'strict-dynamic'` | Backward compat only |

**The trust chain with `'strict-dynamic'`:**
```
Our inline script  ──(sha256 hash verified)──▶  trusted
  └─ dynamically loads [form script]            ──▶  trusted via strict-dynamic
       └─ dynamically loads reCAPTCHA           ──▶  trusted via strict-dynamic
```
No external CDN domains need to be listed in `script-src`.

### `style-src 'self' 'unsafe-inline'`
Required because Astro inlines all CSS as `<style>` blocks when `inlineStylesheets: 'always'` is set. The generated CSS changes every build, making hashing impractical. This is acceptable — CSS-based attacks are far less dangerous than JS-based ones. Remove `'unsafe-inline'` if `inlineStylesheets` is not set to `'always'`.

### `font-src 'self'`
Only allows fonts from the same origin. Works because all fonts are self-hosted in `/public/fonts/`. Add `https://fonts.gstatic.com` if using Google Fonts.

### `img-src 'self' data:`
`data:` covers SVGs and base64-encoded images. Add external image CDN domains if used.

### `form-action`
Restricts where forms can POST to. Even if an attacker injects a form, it can only submit to listed endpoints. **Always lock this down to your specific form service domain.**

### `frame-src https://www.google.com`
reCAPTCHA renders its challenge widget inside a Google-hosted iframe. Required when reCAPTCHA is active. Add other iframe providers as needed (YouTube, Vimeo, Maps embeds — see Section 7.7).

### `frame-ancestors 'none'`
Prevents this site from being embedded as an iframe anywhere. The modern equivalent of `X-Frame-Options: DENY`. Include both for full browser coverage.

### `connect-src`
Controls URLs reachable via `fetch()`, `XMLHttpRequest`, and WebSocket. Add the form endpoint and any APIs the page calls.

### `base-uri 'self'`
Prevents injection of a `<base>` tag pointing to an attacker's origin (which would redirect all relative URLs). Always include.

### `require-trusted-types-for 'script'`
Requires all DOM XSS sinks (`innerHTML`, `script.src`, `eval`, etc.) to receive a `TrustedType` object rather than a plain string. Prevents DOM-based XSS. Third-party scripts that weren't built with Trusted Types support require the inline policy below.

### `trusted-types default goog#html`

| Policy name | Owner | Purpose |
|---|---|---|
| `default` | Our inline script | Passthrough fallback for any string-based sink assignment not covered by a named policy |
| `goog#html` | Google reCAPTCHA | reCAPTCHA's own internal policy for safe HTML creation |

## 7.5 Trusted Types Pattern

When `require-trusted-types-for 'script'` is active, any inline script that dynamically loads another script must use a Trusted Types policy. Add this to the inline `<script is:inline>` block in `GlobalLayout.astro`, before any dynamic script loading:

```javascript
<script is:inline>
  if (window.trustedTypes && trustedTypes.createPolicy) {
    trustedTypes.createPolicy('default', {
      createScriptURL: function(url) { return url; }
    });
  }
  /* IntersectionObserver lazy-load block follows here */
</script>
```

**Why `default` policy:** Third-party scripts (form providers, reCAPTCHA) do their own DOM sink assignments internally. Since we can't modify their code, the `default` policy acts as a passthrough fallback — automatically invoked for any sink assignment without a named policy. This keeps `require-trusted-types-for` active and satisfies the audit while not breaking third-party scripts.

**If a third-party script creates its own named policy** (you'll see a console error like `"Creating a TrustedTypePolicy named 'xyz' violates..."`), add that policy name to the `trusted-types` directive in `netlify.toml`.

## 7.6 Computing the Inline Script Hash

The `sha256-[HASH]` value in `script-src` must exactly match the inline script content. Run this from the project root after finalising the script:

```bash
node -e "
const fs = require('fs');
const crypto = require('crypto');
const html = fs.readFileSync('src/layouts/GlobalLayout.astro', 'utf8');
const match = html.match(/<script is:inline>([\s\S]*?)<\/script>/);
const hash = crypto.createHash('sha256').update(match[1]).digest('base64');
console.log('sha256-' + hash);
"
```

> **The hash is computed over the exact whitespace between the `<script>` and `</script>` tags.** Any formatting change — extra spaces, tab vs. space, added/removed lines — invalidates the hash and breaks the CSP. Always recompute after modifying the inline script, then update `netlify.toml`.

## 7.7 Feature-to-CSP Quick Reference

| Feature | CSP additions required |
|---|---|
| Google Analytics (GA4) | `connect-src`: `https://www.google-analytics.com https://analytics.google.com` |
| Google Tag Manager | `connect-src`: `https://www.googletagmanager.com` |
| Google Fonts | `font-src`: `https://fonts.gstatic.com`; `style-src`: `https://fonts.googleapis.com` |
| YouTube embed | `frame-src`: `https://www.youtube.com https://www.youtube-nocookie.com` |
| Vimeo embed | `frame-src`: `https://player.vimeo.com` |
| Google Maps embed | `frame-src`: `https://www.google.com` (already in template) |
| Stripe | `frame-src`: `https://js.stripe.com`; `connect-src`: `https://api.stripe.com` |
| Netlify Forms | `form-action` covered by `'self'` — no additions needed |
| Formspree | `form-action`: `https://formspree.io`; `connect-src`: `https://formspree.io` |
| Hotjar | `connect-src`: `https://*.hotjar.com`; `frame-src`: `https://*.hotjar.com` |
| Intercom | `frame-src`: `https://widget.intercom.io`; `connect-src`: `https://api-iam.intercom.io` |

> For GA4, GTM, Stripe, and other dynamically-loaded scripts: `'strict-dynamic'` handles the script loading automatically. Only `connect-src` additions are typically needed.

## 7.8 Setup Checklist

**Step 1: Audit external dependencies**
Before writing any CSP, list every external resource the site loads:
- [ ] External scripts (form services, analytics, chat widgets, etc.)
- [ ] External stylesheets
- [ ] Fonts (self-hosted or CDN)
- [ ] External images or CDNs
- [ ] Form endpoints
- [ ] Iframe embeds (maps, video, payment widgets)

**Step 2: Set netlify.toml headers**
Copy the template from Section 7.2 and customise:
- Replace `[FORM_ENDPOINT_DOMAIN]` in `form-action` and `connect-src`
- Add any iframe providers to `frame-src` (use Section 7.7)
- Add Google Fonts domains to `font-src` and `style-src` if used

**Step 3: Handle the inline script**
Add the Trusted Types `default` policy block (Section 7.5) before any dynamic script loading.

**Step 4: Compute and set the inline script hash**
Run the Node command from Section 7.6 and paste the result into `netlify.toml`.

**Step 5: Deploy and test**
- Deploy to Netlify
- Open DevTools Console — look for CSP violation errors
- Test all interactive features (forms, maps, videos, etc.)
- Test contact form end-to-end including spam protection
- Re-run the Google PageSpeed audit to confirm all items are resolved
- Optionally validate at [csp-evaluator.withgoogle.com](https://csp-evaluator.withgoogle.com/)

## 7.9 Audit Reference

| PageSpeed Audit Item | What It Checks | Fix |
|---|---|---|
| Ensure CSP is effective against XSS | Presence of `Content-Security-Policy` in enforcement mode | Add CSP to `netlify.toml` headers |
| Ensure proper origin isolation with COOP | Presence of `Cross-Origin-Opener-Policy` header | Add `COOP: same-origin` |
| Mitigate clickjacking with XFO or CSP | Presence of `X-Frame-Options` or `frame-ancestors` | Add both `X-Frame-Options: DENY` and `frame-ancestors 'none'` |
| Mitigate DOM-based XSS with Trusted Types | Presence of `require-trusted-types-for 'script'` | Add directive + `trusted-types` allowlist + inline policy |

---

# 8. Accessibility

## 8.1 ARIA Labelling Rules

- **Never add `aria-label` to a plain `<div>` or `<span>`.** These have the `generic` ARIA role, which prohibits naming attributes. The label is silently ignored and flagged as an error by auditors.
- **Add a meaningful `role` first**, then the label. Use semantic HTML where possible — ARIA should only fill gaps that HTML alone cannot cover.
- **`aria-label` is valid on:** interactive elements (`<button>`, `<a>`, `<input>`), landmark elements (`<nav>`, `<main>`, `<section>` with a label), and elements with an explicit supporting role (e.g. `role="img"`, `role="group"`, `role="region"`).

**Do not:**
```html
<div aria-label="Five-star rating">
  <!-- SVG icons -->
</div>
```

**Do:**
```html
<div role="img" aria-label="Five-star rating">
  <!-- SVG icons, each with aria-hidden="true" -->
</div>
```

## 8.2 Star Rating and Icon Group Pattern

When a group of decorative icons together conveys a single semantic meaning (e.g. a star rating, a checkmark list):

1. Wrap the group in a container with `role="img"` and `aria-label` describing the meaning
2. Add `aria-hidden="true"` to each individual icon

```html
<div role="img" aria-label="Five-star patient review">
  <svg aria-hidden="true">...</svg>
  <svg aria-hidden="true">...</svg>
  <svg aria-hidden="true">...</svg>
  <svg aria-hidden="true">...</svg>
  <svg aria-hidden="true">...</svg>
</div>
```

## 8.3 Color Contrast — WCAG AA Standards

**Minimum contrast ratios:**
- **4.5:1** — normal text (below 18px regular or 14px bold)
- **3:1** — large text (18px+ regular or 14px+ bold)
- **3:1** — UI components and graphical objects (buttons, input borders, icons that convey information)

**Key rules:**
- Always verify accent colors against **every background they will appear on** — not just the primary/dark ones. An accent that passes on a dark section will frequently fail on a light or white section.
- Button backgrounds must meet 4.5:1 against the button's text color (white text on the button's background color, not against the page).
- Decorative elements are exempt. Low-opacity watermarks and `aria-hidden` icon glyphs are not subject to contrast requirements. Do not add `aria-hidden` purely to dodge contrast checks on visible text.

## 8.4 The Contrast-Safe Scale Methodology

When building an accent color scale, validate contrast early:

1. For each accent color, identify the **darkest shade that still reads as accent** (not as a dark background tone)
2. Check that shade against white (#ffffff) — must pass 4.5:1
3. Check that shade against the lightest warm background in the palette — must pass 4.5:1
4. Document this shade as the **contrast-safe floor** for that color group
5. Use this shade (or darker) for all text uses on light backgrounds — the default mid-point shade is typically too light for body text

> Example pattern: if the default accent is the mid-point of the scale, `-dark` is usually the contrast-safe floor for light-background text use.

## 8.5 New Page Accessibility Checklist

- [ ] Every `aria-label` is on an element with a role that supports naming (not a plain `<div>` or `<span>`)
- [ ] Decorative images and icons have `aria-hidden="true"`
- [ ] Interactive elements have accessible names (visible text, `aria-label`, or `aria-labelledby`)
- [ ] Landmark regions (`<nav>`, `<main>`, `<aside>`, `<footer>`) are used correctly and not duplicated
- [ ] No duplicate IDs in the document
- [ ] All star ratings or icon groups use `role="img"` with `aria-label` on the container; individual icons are `aria-hidden="true"`
- [ ] Accent colors on light backgrounds use the contrast-safe floor shade or darker (verified at 4.5:1)

---

# 9. Deployment Checklist

## 9.1 Pre-Deployment

- [ ] All `[PLACEHOLDER]` values in the codebase have been replaced with project-specific values
- [ ] Section 10 (Project Configuration Reference) is fully filled in
- [ ] `netlify.toml` has the computed inline script hash (not `[INLINE_SCRIPT_HASH]`)
- [ ] All pages have unique `title` and `description` props (not layout defaults)
- [ ] All `<img>` elements have `width`, `height`, and `alt` attributes
- [ ] All images are 8-bit WebP at the correct intrinsic dimensions, compressed under target file sizes
- [ ] Contact form submits successfully in local dev and redirects correctly
- [ ] DevTools Console shows no CSP violations, no font fallback warnings, no 404s on local build
- [ ] `npm run build` completes with no errors or warnings

## 9.2 Netlify Configuration

- [ ] `netlify.toml` `[build]` section: `command = "npm run build"` and `publish = "dist"`
- [ ] Security `[[headers]]` block is present with all 6 headers
- [ ] CSP `[INLINE_SCRIPT_HASH]` has been replaced with the production computed value
- [ ] `[FORM_ENDPOINT_DOMAIN]` in `form-action` and `connect-src` matches the actual form service domain
- [ ] Any environment variables required by form services are set in the Netlify dashboard

## 9.3 Post-Deployment

- [ ] Visit the deployed URL — fonts render correctly (not fallback Arial/system-ui)
- [ ] DevTools Network tab: CSS is inlined in the HTML document; no separate `.css` file request
- [ ] DevTools Console: no CSP errors, no mixed content warnings
- [ ] Submit the contact form end-to-end (spam protection triggers correctly; redirect occurs)
- [ ] Test all internal links and CTA buttons
- [ ] Validate security headers at [securityheaders.com](https://securityheaders.com)
- [ ] Optionally validate CSP at [csp-evaluator.withgoogle.com](https://csp-evaluator.withgoogle.com/)
- [ ] Run Google PageSpeed Insights on the deployed URL (Performance, Accessibility, Best Practices, SEO — all should score 90+)

---

# 10. Project Configuration Reference

Fill in this table at the start of every project. These are the only values that change between projects — everything else in this playbook is universal.

| Placeholder | Description | Where Used |
|---|---|---|
| `[SITE_NAME]` | The brand or business name | GlobalLayout default title, Footer |
| `[DEFAULT_LOCATION_OR_TAGLINE]` | Short location or tagline for the default page title | GlobalLayout default `title` prop |
| `[DEFAULT_META_DESCRIPTION]` | Generic fallback meta description (120–160 chars) | GlobalLayout default `description` prop |
| `[HEADING_FONT]` | Variable font family name for headings | `@font-face`, `@theme`, font files, `<link rel="preload">` |
| `[BODY_FONT]` | Variable font family name for body text | `@font-face`, `@theme`, font files, `<link rel="preload">` |
| `[PRIMARY_COLOR]` | CSS token name for the primary brand color group (e.g. `navy`) | `@theme`, component classes, Navbar bg |
| `[ACCENT_COLOR]` | CSS token name for the primary accent color group (e.g. `rose`) | `@theme`, CTA buttons, icon accents |
| `[CONTACT_PHONE]` | Formatted phone number (e.g. `(555) 000-0000`) | Phone CTA button, Footer |
| `[CONTACT_EMAIL]` | Contact email address | Footer |
| `[CONTACT_ADDRESS]` | Street address line 1 | Footer |
| `[CONTACT_ADDRESS_2]` | City, state, zip | Footer |
| `[MAPS_HREF]` | Google Maps link for the practice address | Footer address link |
| `[BOOKING_HREF]` | Appointment booking URL or form anchor (e.g. `#contact`) | Action CTA button |
| `[FORM_ENDPOINT]` | Full form submission URL | ContactForm `action` attribute |
| `[FORM_ENDPOINT_DOMAIN]` | Domain only of the form endpoint (e.g. `usebasin.com`) | CSP `form-action` and `connect-src` |
| `[THIRD_PARTY_SCRIPT_URL]` | URL of the form service script to lazy-load | IntersectionObserver inline script |
| `[FORM_ELEMENT_SELECTOR]` | CSS selector targeting the form element (e.g. `[data-basin-form]`) | IntersectionObserver `querySelector` |
| `[INLINE_SCRIPT_HASH]` | SHA-256 hash of `<script is:inline>` content | CSP `script-src` in `netlify.toml` |
---

## Google Tag Manager (GTM) Integration

### Overview
GTM is added globally via `src/layouts/GlobalLayout.astro` so it fires on every page.

### Implementation Pattern

**1. Head script** — place as the **first child** of `<head>`:
```html
<!-- Google Tag Manager -->
<script is:inline>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-XXXXXXX');</script>
<!-- End Google Tag Manager -->
```

**2. Noscript fallback** — place as the **first child** of `<body>`:
```html
<!-- Google Tag Manager (noscript) -->
<noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-XXXXXXX"
height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
<!-- End Google Tag Manager (noscript) -->
```

### Astro-Specific Rules
- Always use `is:inline` on the head `<script>` tag to prevent Astro from bundling or fingerprinting it.
- Do **not** use `<Script>` (framework component) — use the raw HTML `<script is:inline>` directive.
- The `<noscript>` block requires no special Astro attributes; plain HTML works fine.

### Placeholder Reference
| Placeholder | Value | Location |
|---|---|---|
| `GTM-XXXXXXX` | Client's GTM container ID | Both head script and noscript iframe `src` |
