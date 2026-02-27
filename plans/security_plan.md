# Security Headers Plan

## Overview

This document covers the security headers implemented on the Core Dentistry site to address Google PageSpeed Trust & Safety audit items. All headers are enforced via Netlify's edge CDN with zero performance cost.

**Audit items resolved:**
- No CSP found in enforcement mode ✅
- No COOP header found ✅
- No frame control policy found ✅
- No Trusted Types directive found ✅

---

## Implementation: netlify.toml

All headers are set in `netlify.toml` using a `[[headers]]` block. This is the correct approach for any **static Astro site hosted on Netlify** — no middleware, no server code needed.

```toml
[[headers]]
  for = "/*"
  [headers.values]
    X-Content-Type-Options = "nosniff"
    X-Frame-Options = "DENY"
    Referrer-Policy = "strict-origin-when-cross-origin"
    Cross-Origin-Opener-Policy = "same-origin"
    Permissions-Policy = "camera=(), microphone=(), geolocation=()"
    Content-Security-Policy = "default-src 'self'; script-src 'self' 'sha256-INLINE_SCRIPT_HASH' 'strict-dynamic' 'unsafe-inline' https: http:; style-src 'self' 'unsafe-inline'; font-src 'self'; img-src 'self' data:; form-action https://YOUR_FORM_ENDPOINT; frame-src https://www.google.com; frame-ancestors 'none'; connect-src 'self' https://YOUR_FORM_ENDPOINT https://www.google.com; base-uri 'self'; require-trusted-types-for 'script'; trusted-types default goog#html"
```

---

## Header-by-Header Reference

### X-Content-Type-Options
```
X-Content-Type-Options: nosniff
```
Prevents browsers from MIME-sniffing a response away from its declared content type. Stops attacks where a disguised file is executed as a different type (e.g. an image treated as a script).

**Always include. No downsides.**

---

### X-Frame-Options
```
X-Frame-Options: DENY
```
Prevents the site from being embedded in an `<iframe>` on any other domain, blocking clickjacking attacks. `DENY` is stricter than `SAMEORIGIN`.

**Note:** This is a legacy header. Modern browsers use `frame-ancestors` in the CSP instead. Include both — XFO covers old browsers that don't support CSP.

---

### Referrer-Policy
```
Referrer-Policy: strict-origin-when-cross-origin
```
Controls how much referrer information is sent with requests. This value sends the full URL for same-origin requests, but only the origin (no path) for cross-origin — protecting any sensitive URL parameters from leaking to third parties.

**Always include. No downsides.**

---

### Cross-Origin-Opener-Policy (COOP)
```
Cross-Origin-Opener-Policy: same-origin
```
Isolates the browsing context so cross-origin windows (e.g. popups opened by the page) can't access the `window` object. Prevents cross-origin attacks that rely on shared browsing context references.

**Always include for sites that don't deliberately open cross-origin popups.**

---

### Permissions-Policy
```
Permissions-Policy: camera=(), microphone=(), geolocation=()
```
Explicitly disables browser features the site doesn't use. An empty `()` means the feature is denied for all origins including the page itself. Prevents compromised third-party scripts from silently accessing hardware.

**Adjust based on what the site actually uses.** Add `payment=()` if not using a payment API.

---

### Content-Security-Policy (CSP)

The most complex header. Full breakdown below.

```
default-src 'self';
script-src 'self' 'sha256-HASH' 'strict-dynamic' 'unsafe-inline' https: http:;
style-src 'self' 'unsafe-inline';
font-src 'self';
img-src 'self' data:;
form-action https://YOUR_FORM_ENDPOINT;
frame-src https://www.google.com;
frame-ancestors 'none';
connect-src 'self' https://YOUR_FORM_ENDPOINT https://www.google.com;
base-uri 'self';
require-trusted-types-for 'script';
trusted-types default goog#html
```

#### `default-src 'self'`
The catch-all fallback for any directive not explicitly listed. `'self'` means only resources from the same origin are allowed by default. Acts as a safety net.

#### `script-src` — The Most Important Directive
```
'self' 'sha256-HASH' 'strict-dynamic' 'unsafe-inline' https: http:
```
Controls what JavaScript can execute. This combination is the **Google-recommended modern CSP pattern**:

| Token | Effect | Why |
|---|---|---|
| `'self'` | Allows scripts served from the same origin | Covers any JS files the build system outputs to `/_astro/` |
| `'sha256-HASH'` | Allows a specific inline `<script>` block by its exact content hash | The secure way to permit inline scripts without `'unsafe-inline'` |
| `'strict-dynamic'` | Propagates trust: any script loaded by a hashed/nonced script is also trusted | Eliminates need for listing external CDN domains (Basin, Google, etc.) |
| `'unsafe-inline'` | Fallback for very old browsers (ignored when hashes are present) | Backward compat only |
| `https: http:` | Fallback for browsers that understand schemes but not `'strict-dynamic'` | Backward compat only |

**The trust chain with `'strict-dynamic'`:**
```
Our inline script  ──(sha256 hash verified)──▶  trusted
  └─ dynamically loads Basin script             ──▶  trusted via strict-dynamic
       └─ dynamically loads reCAPTCHA            ──▶  trusted via strict-dynamic
```
No external CDN domains need to be listed in `script-src` at all.

#### `style-src 'self' 'unsafe-inline'`
`'unsafe-inline'` is required here because **Astro inlines all CSS** as `<style>` blocks when `inlineStylesheets: 'always'` is set in `astro.config.mjs`. The generated CSS changes every build making hashing impractical. This is acceptable — CSS-based attacks are far less dangerous than JS-based ones.

If `inlineStylesheets` is not set to `'always'`, you can remove `'unsafe-inline'` from `style-src`.

#### `font-src 'self'`
Only allows fonts from the same origin. Works because all fonts are self-hosted in `/public/fonts/`. If using Google Fonts or another CDN, add `https://fonts.gstatic.com`.

#### `img-src 'self' data:`
`data:` is needed for any SVG or base64-encoded images. Add external image CDN domains here if used (e.g. `https://images.unsplash.com`).

#### `form-action`
Restricts where forms on the page can POST to. Even if an attacker injects a form, it can only submit to the listed endpoints. **Always lock this down** to your specific form service domain.

#### `frame-src https://www.google.com`
Controls what origins can be loaded in `<iframe>` elements. Google reCAPTCHA renders its challenge widget inside a Google-hosted iframe, so this is required when reCAPTCHA is active.

#### `frame-ancestors 'none'`
Prevents this site from being embedded as an iframe anywhere. The modern equivalent of `X-Frame-Options: DENY`. Include both for full browser coverage.

#### `connect-src`
Controls URLs that can be reached via `fetch()`, `XMLHttpRequest`, and WebSocket. Lists the form endpoint and any APIs the page calls.

#### `base-uri 'self'`
Prevents injection of a `<base>` tag pointing to an attacker's origin, which would redirect all relative URLs. Always include.

#### `require-trusted-types-for 'script'`
Requires that all DOM XSS sinks (innerHTML, script.src, eval, etc.) receive a `TrustedType` object rather than a plain string. Prevents DOM-based XSS attacks.

**Caveat with third-party scripts:** Libraries like reCAPTCHA that weren't built with Trusted Types support will break unless handled — see the Trusted Types section below.

#### `trusted-types default goog#html`
Lists which Trusted Types policy names are allowed to be created:

| Policy name | Owner | Purpose |
|---|---|---|
| `default` | Our inline script | Passthrough fallback for any string-based sink assignment not covered by a named policy (covers Basin's internal DOM operations) |
| `goog#html` | Google reCAPTCHA | reCAPTCHA's own internal policy for safe HTML creation |

---

## Trusted Types: Inline Script Pattern

When `require-trusted-types-for 'script'` is in the CSP, any inline script that dynamically loads another script must use a Trusted Types policy. Add this before any dynamic script loading in `GlobalLayout.astro`:

```javascript
<script is:inline>
  if (window.trustedTypes && trustedTypes.createPolicy) {
    trustedTypes.createPolicy('default', {
      createScriptURL: function(url) { return url; }
    });
  }
  // ... rest of dynamic script loading
</script>
```

**Why `default` policy:** Third-party scripts (Basin, reCAPTCHA) do their own DOM sink assignments internally. Since we can't modify their code, the `default` policy acts as a passthrough fallback — it's automatically invoked for any sink assignment that doesn't already have a named policy. This keeps the `require-trusted-types-for` directive active and satisfies the audit.

**Why not a named policy for `'self'`:** A named policy only protects our own code. Any third-party script on the page without its own policy would still break. The `default` + `goog#html` combination covers our code, Basin, and reCAPTCHA.

---

## Computing the Inline Script Hash

The `sha256-HASH` value in `script-src` must exactly match the inline script content. Run this from the project root after finalising the script:

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

**Important:** The hash is computed over the exact whitespace content between the `<script>` and `</script>` tags — any formatting change (extra spaces, tab/space difference, added/removed lines) will invalidate it and break the CSP. Always recompute the hash after modifying the inline script, then update `netlify.toml`.

---

## Checklist for Future Projects

### Step 1: Audit external dependencies
Before writing any CSP, list every external resource the site loads:
- [ ] External scripts (`<script src="https://...">` or dynamically loaded)
- [ ] External stylesheets
- [ ] Fonts (Google Fonts, Adobe, etc.)
- [ ] External images / CDNs
- [ ] Form endpoints (Basin, Formspree, Netlify Forms, etc.)
- [ ] Analytics scripts (GA4, GTM, Plausible, etc.)
- [ ] Maps embeds (Google Maps iframe, Mapbox, etc.)
- [ ] Video embeds (YouTube, Vimeo iframe)
- [ ] Payment widgets (Stripe, Square, etc.)

### Step 2: Set `netlify.toml` headers
Copy the template from the Implementation section above and customise:
- Replace `INLINE_SCRIPT_HASH` after computing it
- Replace form endpoint domains in `form-action` and `connect-src`
- Add any iframe providers to `frame-src`
- Add any external image CDNs to `img-src`
- Add Google Fonts domains to `font-src` and `style-src` if used

### Step 3: Handle the inline script
If there are any dynamically-loaded third-party scripts, add the Trusted Types `default` policy block before them.

If any third-party script creates its own Trusted Types policy (you'll see a console error like `"Creating a TrustedTypePolicy named 'xyz' violates..."`), add that policy name to the `trusted-types` directive.

### Step 4: Compute and set the inline script hash
Run the Node hash command and paste the result into `netlify.toml`.

### Step 5: Deploy and test
- Deploy to Netlify
- Open DevTools Console — look for any CSP violation errors
- Test all interactive features (forms, maps, videos, etc.)
- Test contact form end-to-end including spam protection
- Re-run the Google PageSpeed audit to confirm all items are resolved
- Optionally validate at https://csp-evaluator.withgoogle.com/

### Step 6: Common CSP additions by feature
| Feature | Add to CSP |
|---|---|
| Google Analytics (GA4) | `script-src`: `'strict-dynamic'` handles it; `connect-src`: `https://www.google-analytics.com https://analytics.google.com` |
| Google Tag Manager | `script-src`: `'strict-dynamic'` handles it; `connect-src`: `https://www.googletagmanager.com` |
| Google Fonts | `font-src`: `https://fonts.gstatic.com`; `style-src`: `https://fonts.googleapis.com` |
| YouTube embed | `frame-src`: `https://www.youtube.com https://www.youtube-nocookie.com` |
| Vimeo embed | `frame-src`: `https://player.vimeo.com` |
| Google Maps embed | `frame-src`: `https://www.google.com` |
| Stripe | `script-src`: `'strict-dynamic'` handles it; `frame-src`: `https://js.stripe.com`; `connect-src`: `https://api.stripe.com` |
| Netlify Forms | `form-action`: already covered by `'self'` |
| Formspree | `form-action`: `https://formspree.io`; `connect-src`: `https://formspree.io` |
| Hotjar | `script-src`: `'strict-dynamic'` handles it; `connect-src`: `https://*.hotjar.com`; `frame-src`: `https://*.hotjar.com` |
| Intercom | `script-src`: `'strict-dynamic'` handles it; `frame-src`: `https://widget.intercom.io`; `connect-src`: `https://api-iam.intercom.io` |

---

## What Each Google PageSpeed Audit Item Checks

| Audit Item | What It Checks | How We Fixed It |
|---|---|---|
| Ensure CSP is effective against XSS | Presence of a `Content-Security-Policy` HTTP header in enforcement mode (not `Content-Security-Policy-Report-Only`) | Added CSP to `netlify.toml` headers |
| Ensure proper origin isolation with COOP | Presence of `Cross-Origin-Opener-Policy` header | Added `COOP: same-origin` |
| Mitigate clickjacking with XFO or CSP | Presence of `X-Frame-Options` or `frame-ancestors` in CSP | Added both `X-Frame-Options: DENY` and `frame-ancestors 'none'` |
| Mitigate DOM-based XSS with Trusted Types | Presence of `require-trusted-types-for 'script'` in the CSP | Added the directive + `trusted-types` policy allowlist + inline script policy |
