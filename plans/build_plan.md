# Build Configuration Plan

## CSS Output Naming

**File:** `astro.config.mjs`

By default, Astro/Vite names the shared CSS bundle after the first page it processes alphabetically. In this project that resulted in the output file being named `emergency.[hash].css`, which was misleading since it is the global stylesheet shared by all pages.

### Change

Added `rollupOptions.output.assetFileNames` to Vite's build config to force all CSS output files to use the name `styles` instead:

```js
assetFileNames: (assetInfo) => {
  if (assetInfo.names?.[0]?.endsWith('.css')) {
    return '_astro/styles.[hash][extname]';
  }
  return '_astro/[name].[hash][extname]';
},
```

**Output before:** `dist/_astro/emergency.[hash].css`
**Output after:** `dist/_astro/styles.[hash].css`

### Notes

- No functional impact. The content hash still updates whenever CSS changes, so browser caching is unaffected.
- All pages (`index`, `emergency`, `implants`, `invisalign`) reference the same single CSS bundle, so this rename applies site-wide.
- Non-CSS assets (images, fonts, etc.) continue to use the default `[name].[hash][extname]` pattern.

---

## Performance Optimisations (PageSpeed)

The following changes were made in response to Google PageSpeed Insights findings.

---

### 1. Inline CSS — eliminate render-blocking stylesheet

**File:** `astro.config.mjs`

PageSpeed flagged `/_astro/styles.[hash].css` as a render-blocking resource (estimated 360 ms savings). Astro's `build.inlineStylesheets: 'always'` option instructs the build to embed all CSS directly in a `<style>` tag in the HTML instead of emitting a separate file. A `<style>` tag is parsed inline and never blocks rendering.

```js
build: {
  inlineStylesheets: 'always',
},
```

**Before:** Browser had to fetch `/_astro/styles.[hash].css` before it could paint anything.
**After:** CSS is in the HTML document itself — no extra network round-trip, no render block.

**Note:** The existing `rollupOptions.output.assetFileNames` CSS naming rule remains in place but has no effect while `inlineStylesheets: 'always'` is active. It will apply again if `inlineStylesheets` is ever changed to `'auto'` or `'never'`.

---

### 2. Font preloading — break the CSS → font dependency chain

**File:** `src/layouts/GlobalLayout.astro`

Fonts were loaded via `@font-face` in `global.css`. This creates a dependency chain where the browser can't even discover the font files until after the CSS finishes downloading — adding ~600 ms to font load time and contributing to the LCP render delay (PageSpeed reported 1,140 ms element render delay).

Added two `<link rel="preload">` tags in `<head>`:

```html
<link rel="preload" as="font" type="font/woff2" crossorigin href="/fonts/Inter-Variable.woff2" />
<link rel="preload" as="font" type="font/woff2" crossorigin href="/fonts/BricolageGrotesque-Variable.woff2" />
```

**Before:** HTML loads → CSS loads → fonts discovered and loaded (sequential).
**After:** HTML loads → CSS and fonts load in parallel.

**Note:** `crossorigin` is required even for same-origin fonts when using `<link rel="preload" as="font">`. Omitting it causes the browser to fetch the font twice.

---

### 3. Lazy-load Basin/reCAPTCHA via IntersectionObserver

**File:** `src/layouts/GlobalLayout.astro`

Basin (with its reCAPTCHA spam protection) was loaded globally on every page with a `defer` attribute:

```html
<script src="https://js.usebasin.com/v2.9.0.min.js" defer></script>
```

PageSpeed attributed 1.9 s of JS execution time and 11 long main-thread tasks to `recaptcha__en.js`. While the reCAPTCHA script itself can't be optimised (it's Google's code), we can control *when* it loads. It only needs to be present when the user is about to submit a form.

Replaced with an IntersectionObserver that injects the Basin script only when the contact form scrolls within 200 px of the viewport:

```html
<script is:inline>
  (function () {
    var loaded = false;
    var observer = new IntersectionObserver(function (entries) {
      if (entries[0].isIntersecting && !loaded) {
        loaded = true;
        var s = document.createElement('script');
        s.src = 'https://js.usebasin.com/v2.9.0.min.js';
        document.head.appendChild(s);
        observer.disconnect();
      }
    }, { rootMargin: '200px' });
    var form = document.querySelector('[data-basin-form]');
    if (form) observer.observe(form);
  })();
</script>
```

**Before:** reCAPTCHA loaded and ran on every page, on every visit, at page load time.
**After:** reCAPTCHA only loads when a page with a contact form is scrolled near the form section.

**Note:** The `rootMargin: '200px'` gives the script ~200 px of lead time before the form is visible, so it is initialised and ready before the user reaches it. The `loaded` guard prevents double-injection.

---

### 4. Explicit dimensions on logo images — prevent CLS

**Files:** `src/components/navbar.astro`, `src/components/FooterSection.astro`

Both logo `<img>` elements were missing explicit `width` and `height` HTML attributes. Without them, the browser cannot reserve space for the image before it downloads, which can cause a layout shift (CLS). PageSpeed flagged both instances.

Added the actual pixel dimensions of `Logo-optimised.webp` (293 × 40 px):

```html
<img src="/images/Logo-optimised.webp" alt="Core Dentistry" width="293" height="40" class="h-10 w-auto" />
```

The Tailwind class `h-10 w-auto` continues to control the rendered size. The HTML attributes simply give the browser the correct aspect ratio so it can reserve the right amount of space in the layout immediately, before the image is fetched.

changed for branding test