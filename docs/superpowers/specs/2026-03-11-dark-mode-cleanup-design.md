# Dark Mode Cleanup Design — Item 7

**Date:** 2026-03-11
**Scope:** Remove unnecessary `!important` declarations; replace dark-mode force block with `color-scheme: only light`
**Goal:** Code quality improvement only — site remains light-only, but via standards-compliant declaration instead of fragile `!important` overrides
**Files affected:** `_sass/_variables.scss`, `_sass/_custom.scss`, `_sass/_base.scss`

---

## 1. Core Change: `color-scheme: only light`

Add to the top of Section 0 in `_sass/_custom.scss` (before the Typography block):

```scss
html {
  color-scheme: only light;
}
```

This single declaration tells browsers this site does not support dark mode. It replaces the entire `@media (prefers-color-scheme: dark)` force-light block and the body color overrides. Browser UI elements (form inputs, scrollbars, selection color) also automatically remain light-themed.

---

## 2. Deletions in `_sass/_custom.scss`

### 2a. Delete the dark mode force block (lines ~633–652, ~10 `!important`)

Delete the entire block:

```scss
// Force light theme — prevent dark mode interference
@media (prefers-color-scheme: dark) {
  .floating-card, .highlight-block, .paper-box, .quote-accent {
    background: #ffffff !important;
    color: $primary-color !important;
    border-color: rgba($primary-color, 0.08) !important;
  }
  .floating-card *, .highlight-block *, .paper-box *, .quote-accent * {
    color: $primary-color !important;
  }
  .floating-card a, .highlight-block a, .paper-box a, .quote-accent a {
    color: $primary-color !important;
    &:hover { color: $accent-color !important; }
  }
}
```

Replaced entirely by `html { color-scheme: only light; }`.

### 2b. Delete body force-light rules (lines ~655–657, 2 `!important`)

Delete:

```scss
body {
  background-color: #fff !important;
  color: $primary-color !important;
}
```

### 2c. Delete card/quote redundant background overrides (~lines 661–667, 4 `!important`)

Delete the block that forces white background on cards for light mode — these are now unnecessary since `color-scheme: only light` prevents dark mode from interfering:

```scss
.highlight-block, .paper-box, .floating-card, blockquote, .quote-accent {
  background: #ffffff !important;
  color: $primary-color !important;
}
.highlight-block *, .paper-box *, .floating-card *, blockquote *, .quote-accent * {
  color: $primary-color !important;
}
```

### 2d-extra. Delete standalone badge/button white-text catch-all (~line 670–672, 1 `!important`)

This is a separate block immediately after 2c — delete it:

```scss
.btn-accent, .tag-accent, .badge {
  color: white !important;
}
```

Note: The `.btn-accent` primary definition at lines ~141–179 already handles this color. This block is a duplicate catch-all added as a dark-mode defensive measure.

### 2d. Remove `!important` from individual card rules (lines ~187, 219, 226, 247, 4 `!important`)

These card background/color `!important` flags were added to fight dark mode. Remove just the `!important` keyword, keeping the property values:

- `.floating-card { background: #ffffff !important; }` → `background: #fff;`
- `.quote-accent { background: #f8f9fa !important; }` → `background: #f8f9fa;`
- `.quote-accent { color: $primary-color !important; }` → `color: $primary-color;`
- `.paper-box { background: #ffffff !important; }` → `background: #fff;`

---

## 3. Specificity Fixes in `_sass/_custom.scss`

Replace `!important` with higher-specificity selectors where the conflict is with the theme's heading styles.

### 3a. Heading font-weight (lines ~22)

**Current:**
```scss
h1, h2, h3, h4, h5, h6 {
  font-family: $header-font-family;
  font-weight: 400 !important;
}
```

**Replace with:**
```scss
.page__content h1,
.page__content h2,
.page__content h3,
.page__content h4,
.page__content h5,
.page__content h6,
.sidebar h1,
.sidebar h2,
.sidebar h3 {
  font-family: $header-font-family;
  font-weight: 400;
}
```

### 3b. `.paper-box-text h3` font-weight (line ~42)

**Current:**
```scss
.paper-box-text h3 {
  font-family: $header-font-family;
  font-weight: 400 !important;
}
```

**Replace with** (no change needed — `.paper-box-text h3` is already specific enough without `!important` if heading rule from 3a is also specific):
```scss
.paper-box-text h3 {
  font-family: $header-font-family;
  font-weight: 400;
}
```

### 3c. `.btn-accent` color (lines ~143, 173)

The primary `.btn-accent` block (lines ~141–179) contains `color: white !important` at lines ~143 and ~173. Remove the `!important` keyword from both of these lines — the value `white` stays. Then add `.page__content` as a prefix to raise specificity so the rule beats the theme without `!important`:

**Current (lines ~141–143):**
```scss
.btn-accent {
  ...
  color: white !important;
```

**Replace selector with:**
```scss
.page__content .btn-accent,
.btn-accent {
  ...
  color: white;
```

Apply the same selector change to the `&:hover` rule at line ~173 (remove `!important` from `color: white !important`).

---

## 4. Audit in `_sass/_base.scss`

Review the 7 `!important` declarations in `_base.scss`:

- **Blockquote background/color** (lines ~131, 136): Remove `!important` — with `color-scheme: only light`, dark mode won't interfere
- **`.highlight-block` background** (line ~376): Remove `!important` — same reason
- **`.highlight-block ul li` color** (line ~415): Remove `!important`
- **`.highlight-block a` color** (lines ~430, 433): Remove `!important`
- **`nav a, figure > a ::after display`** (line ~513): Keep — this is a layout fix unrelated to dark mode

---

## 5. Expected Outcome

| Category | Before | After |
|----------|--------|-------|
| Total `!important` | 39 | ~15 |
| Dark-mode-related `!important` | ~21 | 0 |
| Legitimate remaining | ~18 | ~15 |
| `color-scheme` declaration | absent | `html { color-scheme: only light }` |

**Legitimately kept `!important` (examples):**
- `.paper-box.hidden { display: none !important }` — needs to beat JS-applied styles
- `.sidebar .author__name { font-family: ... !important }` — must beat theme's high-specificity sidebar rules
- `nav a::after { display: none !important }` — layout fix

---

## 6. Implementation Order

1. Add `html { color-scheme: only light; }` to `_sass/_custom.scss` (Section 0)
2. Delete dark mode block + body force rules + card redundant blocks from `_sass/_custom.scss` (Sections 2a–2d-extra) — do all deletions together, then build once
3. Remove `!important` from individual card rules in `_sass/_custom.scss` (Section 2d)
4. Fix heading and button selectors in `_sass/_custom.scss` (Section 3)
5. Remove `!important` from dark-mode-related rules in `_sass/_base.scss` (Section 4)
6. Final build and full visual check
