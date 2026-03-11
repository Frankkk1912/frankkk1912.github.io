# Stat Cards Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add three publication stat cards (Publications, Citations, h-index) above the publications list, with citations dynamically fetched from Google Scholar JSON.

**Architecture:** CSS classes added to `_sass/_custom.scss`; HTML block + JS fetch snippet added to `_pages/about.md`. The JS reuses the Liquid-resolved `{{ url }}` variable already in the page to respect the CDN toggle in `_config.yml`. Citations update from JSON on load; h-index and publications count are hardcoded with DOM fallback values.

**Tech Stack:** Jekyll, SCSS, vanilla JS (fetch API).

**Prerequisite:** The `feat/visual-hierarchy` PR must be merged before executing this plan. Verify with `grep "## 📃 Publications" _pages/about.md` — it should return a match.

---

## Chunk 1: CSS

### Task 1: Add stat card styles to `_sass/_custom.scss`

**Files:**
- Modify: `_sass/_custom.scss` (append at end of file)

- [ ] **Step 1: Append stat card CSS**

  Add to the very end of `_sass/_custom.scss`:

  ```scss
  // -------------------------------------------------------------
  // Stat Cards
  // -------------------------------------------------------------

  .stat-cards {
    display: flex;
    gap: 1rem;
    margin: 1rem 0 1.5rem;
    flex-wrap: wrap;
  }

  .stat-card {
    flex: 1;
    min-width: 100px;
    border: 1px solid $border-color;
    border-radius: 8px;
    padding: 1.2rem;
    text-align: center;
    background: $body-color;
  }

  .stat-number {
    font-family: $header-font-family;
    font-size: 2.4rem;
    color: $primary-color;
    font-weight: 400;
    line-height: 1;
  }

  .stat-plus {
    font-size: 1.2rem;
    color: $accent-color;
  }

  .stat-label {
    font-size: 0.72rem;
    color: #6b7280;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    margin-top: 0.4rem;
  }
  ```

- [ ] **Step 2: Build and verify**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll build 2>&1 | tail -3
  ```
  Expected: `done in X seconds` with no SCSS errors.

- [ ] **Step 3: Commit**

  ```bash
  git add _sass/_custom.scss
  git commit -m "feat(style): add stat card CSS"
  ```

---

## Chunk 2: HTML + JS

### Task 2: Add stat cards HTML block to `_pages/about.md`

**Files:**
- Modify: `_pages/about.md`

- [ ] **Step 1: Verify prerequisite**

  ```bash
  grep "## 📃 Publications" _pages/about.md
  ```
  Expected: one matching line. If no match (still `#`), merge `feat/visual-hierarchy` PR first.

- [ ] **Step 2: Insert stat cards HTML**

  In `_pages/about.md`, find:

  ```html
  <span class='anchor' id='-publications'></span>

  ## 📃 Publications

  <div id="publications-wrapper">
  ```

  Replace with:

  ```html
  <span class='anchor' id='-publications'></span>

  ## 📃 Publications

  <div class="stat-cards">
    <div class="stat-card">
      <div class="stat-number">10<span class="stat-plus">+</span></div>
      <div class="stat-label">Publications</div>
    </div>
    <div class="stat-card">
      <div class="stat-number"><span id="citation-count">136</span><span class="stat-plus">+</span></div>
      <div class="stat-label">Citations</div>
    </div>
    <div class="stat-card">
      <div class="stat-number">4</div>
      <div class="stat-label">h-index</div>
    </div>
  </div>

  <div id="publications-wrapper">
  ```

- [ ] **Step 3: Build and verify HTML structure**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll build 2>&1 | tail -3
  ```
  Then confirm the cards appear in the built output:

  ```bash
  grep -c "stat-card" _site/index.html
  ```
  Expected: `3` (one match per card).

### Task 3: Add citation fetch JS

**Files:**
- Modify: `_pages/about.md` (inside the existing `<script>` block at the bottom, or add a new `<script>` block)

- [ ] **Step 1: Add fetch snippet**

  In `_pages/about.md`, find the closing `</script>` tag of the existing publications filter script (at the very bottom of the file). Add the following **before** that closing tag, as a separate self-invoking function:

  ```js
  // Fetch citation count from Google Scholar stats
  (function() {
    var gsUrl = '{{ url }}';
    if (!gsUrl) return;
    fetch(gsUrl)
      .then(function(r) { return r.json(); })
      .then(function(data) {
        var count = data && data.message;
        if (count) {
          var el = document.getElementById('citation-count');
          if (el) el.textContent = count;
        }
      })
      .catch(function() {
        // silently fall back — DOM already shows hardcoded value 136
      });
  })();
  ```

  > `{{ url }}` is a Liquid variable resolved at build time (defined at line ~21 of `about.md`). It resolves to the CDN or raw GitHub URL based on `site.google_scholar_stats_use_cdn`.

- [ ] **Step 2: Build and verify no Liquid/JS errors**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll build 2>&1 | tail -3
  ```
  Expected: clean build.

  Spot-check that the built `_site/index.html` contains the resolved URL (not the literal `{{ url }}`):

  ```bash
  grep "citation-count" _site/index.html | head -3
  ```
  Expected: lines containing `citation-count` and a `gsUrl` assignment with a real URL (starting with `https://`).

- [ ] **Step 3: Commit**

  ```bash
  git add _pages/about.md
  git commit -m "feat(content): add stat cards with live citation fetch"
  ```

---

## Chunk 3: Final verification

- [ ] **Serve locally**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll serve --livereload
  ```

  Open `http://localhost:4000` and verify:
  1. Three cards render in a row above the publications list
  2. Cards show: `10+` / `136+` / `4` (hardcoded fallback values)
  3. If the `google-scholar-stats` branch exists, open browser DevTools → Network and confirm a fetch to the raw GitHub / CDN URL fires on load

- [ ] **Mobile check** — resize to < 600px and confirm cards wrap gracefully (`.stat-cards` has `flex-wrap: wrap`; each card has `min-width: 100px`)

- [ ] **Final commit if mobile fixes needed**

  ```bash
  git add _sass/_custom.scss
  git commit -m "fix(style): stat cards mobile layout"
  ```
