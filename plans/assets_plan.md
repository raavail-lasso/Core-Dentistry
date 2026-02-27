# Assets Plan: Images & Media

## Overview

All static assets live in the `public/images/` directory and are served directly at `/images/<filename>`. No import or build step is required — reference them in `src` attributes as shown below.

---

## Image Optimisation Guidelines

These rules apply to every image on every project. Following them consistently produces perfect Google PageSpeed scores and avoids rework.

### 1. Format — always prefer WebP

- Use **`.webp`** for all photos and UI images (logos, hero images, illustrations).
- Use **`.png`** only when true transparency is required *and* WebP transparency is insufficient (rare — favicons, webclips).
- Never use `.jpg`/`.jpeg` or `.gif` for new assets; convert existing ones to `.webp` if they appear in a PageSpeed audit.

### 2. Bit depth — use 8-bit WebP

- Export WebP images at **8-bit** colour depth (not 16-bit).
- 16-bit WebP files are significantly larger and offer no visible quality improvement for web use.
- Google PageSpeed explicitly flags oversized images caused by unnecessary bit depth.

### 3. Dimensions — match the render size, not the source size

- Provide an image whose **intrinsic dimensions are as close as possible to the largest size it will ever render**.
- Do not supply a 2000 px wide image for an element that never exceeds 440 px.
- For retina/HiDPI screens, up to 2× the CSS render size is acceptable (e.g. 880 px wide for a 440 px element). Beyond 2× is wasteful.
- Always set explicit `width` and `height` attributes on `<img>` elements to match the intrinsic dimensions of the file. This prevents layout shift (CLS) and lets the browser reserve space before the image loads.

### 4. Compression — verify file size before committing

- Run every new image through a tool such as **Squoosh**, **ImageOptim**, or `cwebp` before adding it to the repo.
- Target under **100 KB** for logos and icons; under **200 KB** for full-bleed hero images.
- If PageSpeed flags an image as "Serve images in next-gen formats" or "Properly size images", re-export and replace.

### 5. `<img>` attributes — required every time

```html
<img
  src="/images/filename.webp"
  alt="Descriptive text"
  width="440"
  height="60"
  class="h-10 w-auto"
/>
```

- `src` — always `/images/<filename>` (no `../` relative paths).
- `alt` — always a short, meaningful description (never empty for meaningful images).
- `width` / `height` — always the **intrinsic pixel dimensions of the file** (not the CSS render size). Use `sips -g pixelWidth -g pixelHeight <file>` on macOS to verify.
- CSS (`class`) controls the rendered size; `width`/`height` attributes prevent layout shift.

### 6. Filenames — no spaces

- Use hyphens, not spaces or underscores: `logo-primary.webp`, not `Logo optimised.webp`.
- Spaces require `%20` encoding in `src` attributes and are a common source of broken references.

---

## Current Assets

### Logo

| File | Dimensions | Path | Used In |
|------|-----------|------|---------|
| `Logo-new16bit.webp` | 440 × 60 px | `/images/Logo-new16bit.webp` | `src/components/navbar.astro`, `src/components/FooterSection.astro` |

**Usage:**
```html
<img src="/images/Logo-new16bit.webp" alt="Core Dentistry" width="440" height="60" class="h-10 w-auto" />
```

The logo renders at 40px tall (`h-10`) with `w-auto` to preserve aspect ratio. It appears in both the sticky top navbar and the footer.

#### Logo history

| File | Status | Notes |
|------|--------|-------|
| `Logo.png` | Replaced | Original PNG; no explicit dimensions on `<img>` |
| `Logo-optimised.webp` | Replaced | WebP conversion of original; 293 × 40 px |
| `Logo-new16bit.webp` | **Current** | 440 × 60 px, 8-bit WebP; dimensions meet PageSpeed recommendations |

---

### Hero Images

Each service page has a dedicated hero image displayed in the right column of the Hero section (desktop only, hidden on mobile via `hidden lg:block`).

| File | Path | Page |
|------|------|------|
| `General Dentistry.webp` | `/images/General%20Dentistry.webp` | `src/pages/index.astro` |
| `Emergency.webp` | `/images/Emergency.webp` | `src/pages/emergency.astro` |
| `Implants.webp` | `/images/Implants.webp` | `src/pages/implants.astro` |
| `Invisalign.webp` | `/images/Invisalign.webp` | `src/pages/invisalign.astro` |

> **Note:** `General Dentistry.webp` contains a space in the filename. Use `%20` encoding in the `src` attribute: `/images/General%20Dentistry.webp`. New assets should avoid spaces per the guidelines above.

**Usage pattern (all four hero images):**
```html
<div class="w-full aspect-4/5 rounded-3xl overflow-hidden">
  <img src="/images/<filename>.webp" alt="..." class="w-full h-full object-cover" />
</div>
```

Images are displayed in a `4/5` aspect-ratio container with `rounded-3xl` corners and `object-cover` cropping. A floating badge is absolutely positioned at `-bottom-6 -left-6` over the image container.

---

### Favicon & Webclip

Both files are referenced in `src/layouts/GlobalLayout.astro` and apply to every page on the site.

| File | Path | Tag | Purpose |
|------|------|-----|---------|
| `favicon.png` | `/images/favicon.png` | `<link rel="icon">` | Browser tab icon |
| `webclip.png` | `/images/webclip.png` | `<link rel="apple-touch-icon">` | iOS home screen icon |

**Usage (in `<head>`):**
```html
<link rel="icon" type="image/png" href="/images/favicon.png" />
<link rel="apple-touch-icon" href="/images/webclip.png" />
```

The favicon appears in browser tabs, bookmarks, and browser history. The webclip is used by iOS/iPadOS when a visitor saves the site to their home screen.

---

## Adding New Assets — Checklist

1. Export as **8-bit WebP** at the correct intrinsic dimensions.
2. Compress with Squoosh, ImageOptim, or `cwebp`. Verify file size is reasonable.
3. Use a **hyphenated filename** with no spaces.
4. Drop the file into `public/images/`.
5. Reference it as `/images/<filename>` in the `src` attribute.
6. Add explicit `width` and `height` attributes matching the file's intrinsic pixel dimensions.
7. Write a meaningful `alt` attribute.
8. Document the new asset in this file (name, dimensions, path, and where it is used).
