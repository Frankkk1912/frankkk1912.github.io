# Dark Mode Cleanup Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remove ~24 unnecessary `!important` declarations by adding `color-scheme: only light` and raising selector specificity where needed.

**Architecture:** Single `html { color-scheme: only light }` declaration replaces the entire dark-mode force block and body overrides. Individual card `!important` flags that existed to fight dark mode are then safe to remove. Remaining specificity conflicts are resolved by prefixing with `.page__content`.

**Tech Stack:** Jekyll, SCSS.

---

## Chunk 1: Add `color-scheme: only light`

### Task 1: Insert declaration into `_sass/_custom.scss`

**Files:**
- Modify: `_sass/_custom.scss` (line ~9, before `body {}`)

- [ ] **Step 1: Insert `color-scheme` rule**

  In `_sass/_custom.scss`, find (lines 7–10):
  ```scss
  // -------------------------------------------------------------
  // 0. Typography — Font Stack
  // -------------------------------------------------------------

  body {
  ```

  Replace with:
  ```scss
  // -------------------------------------------------------------
  // 0. Typography — Font Stack
  // -------------------------------------------------------------

  // Declare light-only site — replaces all dark-mode force overrides
  html {
    color-scheme: only light;
  }

  body {
  ```

- [ ] **Step 2: Build and verify**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll build 2>&1 | tail -3
  ```
  Expected: clean build.

- [ ] **Step 3: Commit**

  ```bash
  git add _sass/_custom.scss
  git commit -m "feat(style): declare light-only color-scheme"
  ```

---

## Chunk 2: Delete dark-mode force blocks from `_sass/_custom.scss`

### Task 2: Delete the `@media (prefers-color-scheme: dark)` block and body/card overrides

**Files:**
- Modify: `_sass/_custom.scss` (lines 633–672)

Do all deletions in this task together, then build once.

- [ ] **Step 1: Delete the dark-mode media block (lines 633–652)**

  Find and delete this entire block:
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

      &:hover {
        color: $accent-color !important;
      }
    }
  }
  ```

- [ ] **Step 2: Delete the body force-light block (lines 654–658)**

  Find and delete:
  ```scss
  // Ensure base body color
  body {
    background-color: #ffffff !important;
    color: $primary-color !important;
  }
  ```

- [ ] **Step 3: Delete the card redundant background block (lines 660–667)**

  Find and delete:
  ```scss
  .highlight-block, .paper-box, .floating-card, blockquote, .quote-accent {
    background: #ffffff !important;
    color: $primary-color !important;
  }

  .highlight-block *, .paper-box *, .floating-card *, blockquote *, .quote-accent * {
    color: $primary-color !important;
  }
  ```

- [ ] **Step 4: Delete the badge catch-all block (lines 669–672)**

  Find and delete:
  ```scss
  // Keep button and badge text white
  .btn-accent, .tag-accent, .badge {
    color: white !important;
  }
  ```

- [ ] **Step 5: Build and verify**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll build 2>&1 | tail -3
  ```
  Expected: clean build with no SCSS errors.

- [ ] **Step 6: Commit**

  ```bash
  git add _sass/_custom.scss
  git commit -m "refactor(style): delete dark-mode force blocks (replaced by color-scheme)"
  ```

---

## Chunk 3: Remove `!important` from individual card rules in `_sass/_custom.scss`

### Task 3: Strip `!important` from `.floating-card`, `.quote-accent`, `.paper-box`

**Files:**
- Modify: `_sass/_custom.scss` (lines ~187, 219, 226, 247)

- [ ] **Step 1: `.floating-card` background (line ~187)**

  Find:
  ```scss
  .floating-card {
    background: #ffffff !important;
  ```
  Replace with:
  ```scss
  .floating-card {
    background: #fff;
  ```

- [ ] **Step 2: `.quote-accent` background and color (lines ~219, 226)**

  Find:
  ```scss
  .quote-accent {
    background: #f8f9fa !important;
    border-left: 4px solid $accent-color;
    padding: 1.5rem 2rem;
    border-radius: $border-radius;
    box-shadow: $box-shadow;
    position: relative;
    margin: 2rem 0;
    color: $primary-color !important;
  }
  ```
  Replace with:
  ```scss
  .quote-accent {
    background: #f8f9fa;
    border-left: 4px solid $accent-color;
    padding: 1.5rem 2rem;
    border-radius: $border-radius;
    box-shadow: $box-shadow;
    position: relative;
    margin: 2rem 0;
    color: $primary-color;
  }
  ```

- [ ] **Step 3: `.paper-box` background (line ~247)**

  Find:
  ```scss
  .paper-box {
    background: #ffffff !important;
  ```
  Replace with:
  ```scss
  .paper-box {
    background: #fff;
  ```

- [ ] **Step 4: Build and verify**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll build 2>&1 | tail -3
  ```
  Expected: clean build.

- [ ] **Step 5: Commit**

  ```bash
  git add _sass/_custom.scss
  git commit -m "refactor(style): remove !important from card background/color rules"
  ```

---

## Chunk 4: Fix heading and button specificity in `_sass/_custom.scss`

### Task 4: Replace `!important` with higher-specificity selectors

**Files:**
- Modify: `_sass/_custom.scss` (lines ~20–23, ~41–43, ~141–178)

- [ ] **Step 1: Fix heading font-weight (lines ~20–23)**

  Find:
  ```scss
  // DM Serif Display has only one weight (400).
  // Must override the theme's default font-weight: bold on all headings.
  h1, h2, h3, h4, h5, h6 {
    font-family: $header-font-family;
    font-weight: 400 !important;
  }
  ```
  Replace with:
  ```scss
  // DM Serif Display has only one weight (400).
  // Scoped to page__content + sidebar — beats theme's global h1-h6 rule without !important.
  .page__content h1, .page__content h2, .page__content h3,
  .page__content h4, .page__content h5, .page__content h6,
  .sidebar h1, .sidebar h2, .sidebar h3 {
    font-family: $header-font-family;
    font-weight: 400;
  }
  ```

- [ ] **Step 2: Fix `.paper-box-text h3` font-weight (lines ~41–43)**

  Find:
  ```scss
  .paper-box-text h3 {
    font-family: $header-font-family;
    font-weight: 400 !important;
  }
  ```
  Replace with:
  ```scss
  .paper-box-text h3 {
    font-family: $header-font-family;
    font-weight: 400;
  }
  ```

- [ ] **Step 3: Fix `.btn-accent` color (lines ~141–179)**

  Find the `.btn-accent` block opening and the two `color: white !important` lines:
  ```scss
  .btn-accent {
    background: $accent-color;
    color: white !important;
  ```
  and inside `&:hover`:
  ```scss
    &:hover {
      transform: translateY(-2px);
      box-shadow: 0 8px 25px rgba($accent-color, 0.4);
      color: white !important;
  ```

  Change the selector and remove both `!important`:
  ```scss
  .page__content .btn-accent,
  .btn-accent {
    background: $accent-color;
    color: white;
  ```
  and:
  ```scss
    &:hover {
      transform: translateY(-2px);
      box-shadow: 0 8px 25px rgba($accent-color, 0.4);
      color: white;
  ```

- [ ] **Step 4: Build and verify**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll build 2>&1 | tail -3
  ```

  Then spot-check the built output for key elements:
  ```bash
  grep -c "!important" _site/index.html || echo "0"
  ```
  Expected: `0` (no inline `!important` in HTML — it's all in the compiled CSS).

  Visual check: `bundle exec jekyll serve` → open `http://localhost:4000` → confirm buttons and headings still look correct.

- [ ] **Step 4b: Fix `.blog-link` color (lines ~768, 769, 797, 798)**

  `.blog-link` uses `color: white !important` and `text-decoration: none !important` (lines 768–769 and 797–798). These are the same pattern — dark-mode defensive `!important`. Remove them:

  Find (lines ~768–769):
  ```scss
    color: white !important;
    text-decoration: none !important;
  ```
  Replace with:
  ```scss
    color: white;
    text-decoration: none;
  ```

  Find (lines ~797–798, inside `&:hover`):
  ```scss
    color: white !important;
    text-decoration: none !important;
  ```
  Replace with:
  ```scss
    color: white;
    text-decoration: none;
  ```

- [ ] **Step 5: Build and verify**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll build 2>&1 | tail -3
  ```

  Visual check: `bundle exec jekyll serve` → open `http://localhost:4000` → confirm buttons and headings still look correct.

- [ ] **Step 6: Commit**

  ```bash
  git add _sass/_custom.scss
  git commit -m "refactor(style): replace heading/button !important with specificity selectors"
  ```

---

## Chunk 5: Clean up `_sass/_base.scss`

### Task 5: Remove dark-mode defensive `!important` from `_base.scss`

**Files:**
- Modify: `_sass/_base.scss` (lines ~131, 136, 376, 415, 430, 433)

- [ ] **Step 1: Blockquote background (line ~131)**

  Find:
  ```scss
  background: #f8f9fa !important; /* 强制使用浅色背景 */
  ```
  Replace with:
  ```scss
  background: #f8f9fa; /* light background */
  ```

- [ ] **Step 2: Blockquote color (line ~136)**

  Find:
  ```scss
  color: var(--primary-color) !important; /* 确保文字颜色 */
  ```
  Replace with:
  ```scss
  color: var(--primary-color);
  ```

- [ ] **Step 3: `.highlight-block` background (line ~376)**

  Find:
  ```scss
  background: #ffffff !important; /* 强制使用白色背景 */
  ```
  Replace with:
  ```scss
  background: #fff;
  ```

- [ ] **Step 4: `.highlight-block ul li` color (line ~415)**

  Find:
  ```scss
  color: var(--primary-color) !important; /* 确保文字颜色 */
  ```
  Replace with:
  ```scss
  color: var(--primary-color);
  ```

- [ ] **Step 5: `.highlight-block a` colors (lines ~430, 433)**

  Find:
  ```scss
  a {
    color: var(--primary-color) !important;

    &:hover {
      color: var(--accent-color) !important;
    }
  ```
  Replace with:
  ```scss
  a {
    color: var(--primary-color);

    &:hover {
      color: var(--accent-color);
    }
  ```

- [ ] **Step 6: Build and verify**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll build 2>&1 | tail -3
  ```
  Expected: clean build.

- [ ] **Step 7: Commit**

  ```bash
  git add _sass/_base.scss
  git commit -m "refactor(style): remove dark-mode defensive !important from _base.scss"
  ```

---

## Chunk 6: Final audit and verification

- [ ] **Count remaining `!important` in SCSS files**

  ```bash
  grep -c "!important" _sass/_custom.scss _sass/_base.scss
  ```
  Expected: `_sass/_custom.scss: ~15`, `_sass/_base.scss: 1` (the `display: none !important` on nav `::after`)

- [ ] **Full visual check**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll serve --livereload
  ```
  Open `http://localhost:4000` and confirm:
  1. Section headings still use DM Serif Display at weight 400
  2. Floating cards / paper boxes / quote-accent still have white/light backgrounds
  3. Buttons (`.btn-accent`) still show white text
  4. Highlight blocks (Researcher / Developer / Collaboration) look correct
  5. No visual regressions anywhere on the page

- [ ] **Final commit if any touch-up fixes needed**

  ```bash
  git add _sass/_custom.scss _sass/_base.scss
  git commit -m "fix(style): !important cleanup touch-ups"
  ```
