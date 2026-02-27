# Core Dentistry — Component Plan

## Stack

- **Framework:** Astro 5
- **Styling:** Tailwind CSS v4 with custom design tokens via `@theme` in `src/styles/global.css`
- **Fonts:** Bricolage Grotesque (headings), Inter Variable (body)
- **Deployment:** Netlify

---

## Design System Conventions

All components follow these shared rules:

- **Variants:** Every component accepts a `variant` prop — `'default'` for light backgrounds, `'on-dark'` for dark/navy backgrounds
- **Tokens:** Color, font, and spacing classes come from the design system tokens (e.g. `bg-navy`, `text-rose`, `font-body`)
- **Shape:** Pill shape (`rounded-full`) used for all button-style components
- **Transitions:** `transition-colors duration-200` on all interactive elements
- **Typography:** `font-body` (Inter) for UI controls; `font-heading` (Bricolage Grotesque) reserved for display text

---

## Components

### `CallNowButton`
**File:** `src/components/CallNowButton.astro`

A telephone CTA that renders as an `<a href="tel:...">` link. Displays a stacked two-line label ("CALL NOW" + the formatted phone number) alongside a phone icon.

**Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `variant` | `'default' \| 'on-dark'` | `'default'` | Visual context |

**Variants:**

| Variant | Background | Text | Hover |
|---------|-----------|------|-------|
| `default` | `bg-navy` | `text-white` | `hover:bg-navy-dark` |
| `on-dark` | `bg-white` | `text-navy` | `hover:bg-navy-lightest` |

**Notes:**
- Phone number is hardcoded internally (`'(803) 471-4898'`); not configurable via props
- `tel:` href uses the formatted number directly (`tel:(803) 471-4898`)
- Phone icon is an inline SVG (HeroIcons solid phone), `aria-hidden="true"`

---

### `RequestAppointmentButton`
**File:** `src/components/RequestAppointmentButton.astro`

A booking CTA that renders as an `<a>` link. Displays a label with a right-pointing arrow icon.

**Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `variant` | `'default' \| 'on-dark'` | `'default'` | Visual context |

**Variants:**

| Variant | Background | Text | Hover |
|---------|-----------|------|-------|
| `default` | `bg-rose` | `text-white` | `hover:bg-rose-dark` |
| `on-dark` | transparent + `border-white` | `text-white` | `hover:bg-white hover:text-navy` |

**Notes:**
- `href` (`'#request-appointment'`) and `label` (`'Request Appointment'`) are hardcoded internally; not configurable via props
- Arrow icon is an inline SVG (HeroIcons solid arrow-right), `aria-hidden="true"`
- `on-dark` uses a ghost/outlined style that fills white on hover

---

### `CTAGroup`
**File:** `src/components/CTAGroup.astro`

A wrapper that composes `CallNowButton` and `RequestAppointmentButton` side by side. Controls visibility of each button; contact details live inside the individual button components and are not configurable here.

**Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `variant` | `'default' \| 'on-dark'` | `'default'` | Passed to both buttons |
| `showCallNow` | `boolean` | `true` | Toggle `CallNowButton` visibility |
| `showRequestAppointment` | `boolean` | `true` | Toggle `RequestAppointmentButton` visibility |

**Usage examples:**

```astro
<!-- Both buttons, light background -->
<CTAGroup />

<!-- Both buttons, dark background -->
<CTAGroup variant="on-dark" />

<!-- Call button only -->
<CTAGroup showRequestAppointment={false} />

<!-- Appointment button only, dark background -->
<CTAGroup showCallNow={false} variant="on-dark" />
```

---

### `ContactForm`
**File:** `src/components/ContactForm.astro`

A contact form wired to [Basin](https://usebasin.com/) as the backend form provider. Renders four labelled fields and a submit button, with spam protection and redirect-on-success behaviour handled by Basin via HTML attributes.

**Props:** none — the Basin endpoint is hardcoded internally (`https://usebasin.com/f/02b56b01e7c0`). Use `<ContactForm />` with no props everywhere.

**Fields:**

| Field | Input type | `name` | Required |
|-------|-----------|--------|---------|
| Name | `text` | `name` | Yes |
| Phone | `tel` | `phone` | No |
| Email | `email` | `email` | Yes |
| Message | `textarea` | `message` | No |

**Basin attributes (set on `<form>`):**

| Attribute | Value | Purpose |
|-----------|-------|---------|
| `data-basin-form` | `"true"` | Marks the form for Basin processing |
| `data-basin-success-action` | `"redirect"` | Redirects after successful submission |
| `data-spam-protection` | `"recapture"` | Enables Basin's reCAPTURE spam protection |

**Notes:**
- Redirect destination is managed by Basin's dashboard / backend configuration — no client-side redirect prop
- Focus ring uses `focus:ring-rose/20` to match the brand accent colour
- Textarea has `resize-none` to preserve layout integrity

---

---

### `navbar`
**File:** `src/components/navbar.astro`

The top navigation bar. Contains the site logo on the left and a `CTAGroup` on the right. Sticky-positioned at the top of the viewport with a white background.

**Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `showCallNow` | `boolean` | `true` | Toggle `CallNowButton` visibility |
| `showRequestAppointment` | `boolean` | `true` | Toggle `RequestAppointmentButton` visibility |

**Notes:**
- Contact details (phone, appointment href/label) are not configurable on this component; they live inside `CallNowButton` and `RequestAppointmentButton` respectively
- Logo is currently an inline SVG placeholder (`140×40`); swap for an `<img>` pointing to the real asset in `/public` when available
- White background — `CTAGroup` uses `variant="default"` implicitly (navy call button, rose appointment button)
- `<header>` is `sticky top-0 z-50` with a `border-navy-lighter` bottom border
- Inner `<nav>` is constrained to `max-w-7xl` with responsive horizontal padding

---

---

### `FooterSection`
**File:** `src/components/FooterSection.astro`

The site footer. Dark navy background with the logo on the left and three highlighted contact items on the right: address, phone, and email.

**Props:** none — all contact details are hardcoded internally.

**Hardcoded values:**

| Field | Value |
|-------|-------|
| `phone` | `'(803) 471-4898'` |
| `email` | `'admin@coredentistrysc.com'` |
| `addressLine1` | `'485 Tom Hall St Suite 106'` |
| `addressLine2` | `'Fort Mill, SC 29715'` |
| `mapsHref` | `'https://maps.app.goo.gl/jsrJMBriZoArMoLy9'` |

**Notes:**
- Background is `bg-navy-dark` with a `border-navy` top border
- Logo is the same inline SVG placeholder as the navbar; swap for `<img>` when the real asset is available
- Address renders on two lines (`addressLine1` + `addressLine2`) wrapped in a single `<a href={mapsHref} target="_blank" rel="noopener noreferrer">` that opens Google Maps in a new tab; turns `text-rose` on hover
- Phone renders as `<a href="tel:...">`, email as `<a href="mailto:">`; `tel:` href uses the formatted number directly
- Contact icons use `text-rose` to accent against the dark background
- Contact values (`text-white`) turn `hover:text-rose` on interactive items
- Bottom bar shows a copyright line separated by a `border-navy` rule

---

## Planned Components

> Components to be designed and built in future sessions.
- [ ] **HeroSection** — Full-width hero with headline, subtext, and `CTAGroup`
- [ ] **ServiceCard** — Individual service feature card
- [ ] **TestimonialCard** — Patient review card
