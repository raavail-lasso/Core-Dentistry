# Core Dentistry — Layouts Plan

## Overview

This document describes the global layout architecture for the Core Dentistry site, including why it was created, what it contains, and how pages consume it.

---

## Directory Structure

```
src/
├── layouts/
│   └── GlobalLayout.astro   ← shared page shell
├── pages/
│   └── index.astro          ← page content only
└── components/
    ├── navbar.astro
    ├── FooterSection.astro
    └── ...
```

---

## GlobalLayout.astro

**Location:** `src/layouts/GlobalLayout.astro`

### Purpose

Centralises every piece of markup that is shared across all pages so that individual page files only contain their unique content. Any new page simply wraps its sections in `<GlobalLayout>` instead of duplicating boilerplate.

### What It Contains

| Responsibility | Details |
|---|---|
| `<html>` / `<head>` | charset, favicons, viewport, `Astro.generator` meta |
| `<title>` | set via `title` prop |
| `<meta name="description">` | set via `description` prop — required for SEO |
| Font preloading | `<link rel="preload">` for Inter Variable + Bricolage Grotesque woff2 files |
| Global CSS | `import '../styles/global.css'` |
| `<body>` wrapper | `bg-white font-body` base classes |
| `<Navbar />` | rendered at the top of every page |
| `<main>` + `<slot />` | injects the calling page's content |
| `<FooterSection />` | rendered at the bottom of every page with shared contact details |

### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `title` | `string` | `'Core Dentistry — Fort Mill & Tega Cay'` | Sets the `<title>` tag — always override per page |
| `description` | `string` | *(generic fallback)* | Sets `<meta name="description">` — always override per page with a unique value |

### Structure

```astro
---
import '../styles/global.css';
import Navbar from '../components/navbar.astro';
import FooterSection from '../components/FooterSection.astro';

interface Props {
  title?: string;
  description?: string;
}

const {
  title = 'Core Dentistry — Fort Mill & Tega Cay',
  description = '...generic fallback...',
} = Astro.props;
---

<html lang="en">
  <head>
    <!-- meta, favicons, fonts, title -->
  </head>
  <body class="bg-white font-body">
    <Navbar />
    <main>
      <slot />   <!-- page content goes here -->
    </main>
    <FooterSection ... />
  </body>
</html>
```

---

## How Pages Use the Layout

Pages import `GlobalLayout`, wrap all their sections inside it, and only import the components they actually use. The `<html>`, `<head>`, `<body>`, `<Navbar>`, and `<FooterSection>` are no longer present in page files.

### Example — index.astro

```astro
---
import GlobalLayout from '../layouts/GlobalLayout.astro';
import CTAGroup from '../components/CTAGroup.astro';
import ContactForm from '../components/ContactForm.astro';
---

<GlobalLayout>
  <!-- Section 1 — Hero -->
  <section ...>...</section>

  <!-- Section 2 — Agitation -->
  <section ...>...</section>

  <!-- ...remaining sections... -->
</GlobalLayout>
```

Always pass both `title` and `description`:

```astro
<GlobalLayout
  title="Dental Implants Fort Mill | Permanent Tooth Replacement | Core Dentistry"
  description="Restore your smile with dental implants in Fort Mill, SC. Permanent, natural-looking tooth replacement at Core Dentistry. Schedule your consultation today."
>
  ...
</GlobalLayout>
```

---

## Footer Contact Details

The `FooterSection` props are hardcoded in `GlobalLayout.astro`. Update them there when the practice details change:

| Prop | Current Value |
|---|---|
| `phone` | `555-555-5555` |
| `email` | `hello@coredentistry.com` |
| `addressLine1` | `123 Main Street, Suite 100` |
| `addressLine2` | `Fort Mill, SC 29715` |
| `mapsHref` | `#` |

---

## Adding a New Page

1. Create `src/pages/your-page.astro`
2. Import `GlobalLayout` and any page-specific components
3. Wrap all content in `<GlobalLayout title="..." description="...">` — both props are required
4. No need to touch the head, navbar, footer, or global CSS — the layout handles all of it

---

## SEO Requirements

Every page must satisfy the following. These are flagged by Google PageSpeed Insights under the SEO category and apply via `GlobalLayout`.

### Meta title & description rules

- **Title:** under 60 characters — Google truncates longer titles in SERPs
- **Description:** 120–160 characters — Google truncates longer descriptions
- Include the primary location keywords (Fort Mill, Tega Cay, SC) and the core service
- Write for the searcher — a natural, compelling summary, not a keyword list
- Must be **unique per page** — duplicate titles/descriptions across pages provide no SEO value
- Never rely on the layout's default fallbacks in production — always pass explicit values

### Current page descriptions

| Page | Title (chars) | Description (chars) |
|---|---|---|
| `index.astro` | Core Dentistry — Fort Mill & Tega Cay (38) | Judgment-free dental care in Fort Mill & Tega Cay, SC. General, cosmetic, and restorative services led by Dr. Elizabeth Kovalenko. Request your visit today. (158) |
| `emergency.astro` | Dental Emergency Fort Mill \| Same-Day Appointments \| Core Dentistry (58) | Dental emergency in Fort Mill or Tega Cay? Core Dentistry offers same-day appointments for tooth pain, broken teeth, and urgent care. Call now for fast relief. (160) |
| `implants.astro` | Dental Implants Fort Mill \| Permanent Tooth Replacement \| Core Dentistry (60) | Restore your smile with dental implants in Fort Mill, SC. Permanent, natural-looking tooth replacement at Core Dentistry. Schedule your consultation today. (156) |
| `invisalign.astro` | Invisalign Fort Mill \| Clear Aligners \| Core Dentistry (46) | Straighten your teeth discreetly with Invisalign at Core Dentistry in Fort Mill, SC. Clear aligners with no metal wires. Start your smile transformation today. (160) |

### Checklist for every new page

- [ ] `title` prop set — unique, under 60 characters
- [ ] `description` prop set — unique, 120–160 characters
- [ ] Page has exactly one `<h1>`
- [ ] All images have a meaningful `alt` attribute (or `alt=""` if purely decorative)
