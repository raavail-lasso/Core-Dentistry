# Accessibility Plan

A running log of accessibility issues encountered, fixes applied, and rules to follow going forward. The goal is to avoid repeating the same mistakes across projects and ensure Google PageSpeed / Lighthouse accessibility audits stay clean.

---

## Issues & Fixes

### 1. Prohibited ARIA attributes on generic `div` elements

**Audit error:** "Elements use prohibited ARIA attributes"

**Failing pattern:**
```html
<div aria-label="Five-star patient review">
  <!-- decorative SVG stars with aria-hidden="true" -->
</div>
```

**Why it fails:** A plain `<div>` has an implicit `generic` ARIA role. The `generic` role prohibits `aria-label` and `aria-labelledby`. Applying them has no effect and is flagged as an error by accessibility auditors.

**Fix applied:** Add `role="img"` to the container, which gives it a semantic role that supports `aria-label`.

```html
<div role="img" aria-label="Five-star patient review">
  <!-- decorative SVG stars with aria-hidden="true" -->
</div>
```

**Files fixed:**
- `src/pages/index.astro:308`
- `src/pages/emergency.astro:308`
- `src/pages/implants.astro:308`
- `src/pages/invisalign.astro:310`

---

### 2. Insufficient color contrast on light backgrounds

**Audit error:** "Background and foreground colors do not have a sufficient contrast ratio."

**Failing pattern:**
- `text-rose` (#b76e79) on `bg-white` (#ffffff) — ratio ≈ 3.6:1 (needs 4.5:1 for normal text)
- `text-rose` (#b76e79) on `bg-cream-lighter` (#fcfaf9) — ratio ≈ 3.5:1
- `bg-rose text-white` button — white on #b76e79 — ratio ≈ 3.6:1

**Context:** The rose accent color was fine on dark navy sections (`bg-navy`, `bg-navy-dark`) where it scored ≥ 4:1, but it did not have sufficient contrast on light cream/white backgrounds. The button used the same rose as its background, making white text fail too.

**Fix applied:** Stepped down to `text-rose-dark` (#925860) for text on light backgrounds, and changed the button to `bg-rose-dark / hover:bg-rose-darker`.

- `text-rose-dark` (#925860) on white: **5.0:1** ✓
- `text-rose-dark` (#925860) on cream-lighter: **4.83:1** ✓

**Elements fixed across all 4 pages (index, emergency, implants, invisalign):**
- Section eyebrow `<span>` labels on light-bg sections → `text-rose-dark`
- Card subheading `<p>` elements (`font-body text-sm text-rose font-semibold`) → `text-rose-dark`
- `RequestAppointmentButton.astro` (default variant): `bg-rose` → `bg-rose-dark`, `hover:bg-rose-dark` → `hover:bg-rose-darker`

**Left unchanged (dark-background sections):**
- Hero (`bg-navy-dark`) eyebrow spans — `text-rose` passes on dark bg
- Testimonial (`bg-navy`) eyebrow and `<cite>` — `text-rose` passes on dark bg
- Footer icon spans (`aria-hidden="true"`) — on dark bg, not read by screen readers
- Decorative `opacity-[0.08]` background text — not meant to be legible

---

## Rules & Learnings

### ARIA labelling

- **Never add `aria-label` to a plain `<div>` or `<span>`.** These have the `generic` role, which prohibits naming attributes. Either add a meaningful `role`, or use a semantic HTML element instead.
- **`role="img"` is the correct pattern** for a group of decorative icons/SVGs that together convey a single meaning (e.g. a star rating). The individual icons should be `aria-hidden="true"` and the container carries the accessible name via `aria-label`.
- **`aria-label` is valid on:** interactive elements (`<button>`, `<a>`, `<input>`), landmark elements (`<nav>`, `<main>`, `<section>` with a label), and elements with an explicit `role` that supports naming (e.g. `role="img"`, `role="group"`, `role="region"`).
- **When in doubt**, prefer semantic HTML over ARIA. ARIA should only be used to fill gaps that HTML alone cannot cover.

### Star ratings / icon groups

- Always wrap a group of decorative icons in a container with `role="img"` and an `aria-label` that describes the meaning (e.g. `"Five-star patient review"`).
- Mark each individual icon `aria-hidden="true"` so screen readers skip them and only read the container label.

### Color contrast

- **Always verify accent colors against every background they appear on**, not just the primary/dark ones. An accent that passes on a dark section will often fail on a light one.
- **WCAG AA minimum ratios:** 4.5:1 for normal text (below 18px regular or 14px bold), 3:1 for large text (≥ 18px regular or ≥ 14px bold), 3:1 for UI components and graphical objects.
- **Button backgrounds need checking too.** A colored button with white text must meet 4.5:1 (white on the button bg color, not against the page).
- **Build a contrast-safe scale.** For each accent color, identify which shade is the darkest one that is still "accent" yet passes 4.5:1 on both white and the lightest cream background. Use that as the floor for all text uses on light bgs. For this project: `rose-dark` (#925860) is that floor.
- **Decorative elements are exempt.** Low-opacity background watermarks and `aria-hidden` icon glyphs are not subject to contrast requirements. Don't add `aria-hidden` purely to dodge contrast checks on visible text.

### General checklist for new pages

- [ ] Every `aria-label` is on an element with a role that supports naming
- [ ] Decorative images and icons have `aria-hidden="true"`
- [ ] Interactive elements have accessible names (via visible text, `aria-label`, or `aria-labelledby`)
- [ ] Landmark regions (`<nav>`, `<main>`, `<aside>`, `<footer>`) are used correctly
- [ ] No duplicate IDs in the document
