# Stat Cards Design — Item 4

**Date:** 2026-03-11
**Scope:** Publication statistics display cards
**Files affected:** `_pages/about.md`, `_sass/_custom.scss`

**Note:** This feature is implemented after the `feat/visual-hierarchy` PR is merged. At that point `# 📃 Publications` will already have been changed to `## 📃 Publications`.

---

## 1. Visual Design (Style A — Minimal Counter)

Three cards displayed in a horizontal row:

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│     10+      │  │    136+      │  │      4       │
│  PUBLICATIONS│  │   CITATIONS  │  │   H-INDEX    │
└──────────────┘  └──────────────┘  └──────────────┘
```

- **Number:** DM Serif Display, ~2.4rem, `$primary-color`
- **`+` suffix:** smaller (~1.2rem), `$accent-color` — only on Publications and Citations
- **Label:** DM Sans, 0.72rem, uppercase, `letter-spacing: 0.06em`, `#6b7280`
- **Card:** `$body-color` background, `border: 1px solid $border-color`, `border-radius: 8px`, `padding: 1.2rem`
- **Row:** `display: flex`, `gap: 1rem`, `flex-wrap: wrap`

## 2. Placement

Inserted directly after the `## 📃 Publications` heading (which exists after `feat/visual-hierarchy` is merged), before `<div id="publications-wrapper">`.

```html
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

## 3. Data Source

| Stat | Source | Notes |
|------|--------|-------|
| Publications | Hardcoded `10+` | Google Scholar JSON does not expose paper count |
| Citations | JS fetch from `gs_data_shieldsio.json` | Falls back to DOM value `136` on failure |
| h-index | Hardcoded `4` | Cannot reliably source from crawler JSON |

### JSON format (`gs_data_shieldsio.json`)

The crawler writes a Shields.io-compatible JSON with citations in the `message` field:

```json
{
  "schemaVersion": 1,
  "label": "citations",
  "message": "136",
  "color": "blue"
}
```

### Fetch URL

The page already computes the correct URL via Liquid (respects `site.google_scholar_stats_use_cdn`):

```liquid
{% assign url = gsDataBaseUrl | append: "google-scholar-stats/gs_data_shieldsio.json" %}
```

The JS snippet reuses this resolved URL via a Liquid interpolation in the `<script>` block:

```html
<script>
(function() {
  var gsUrl = '{{ url }}';
  fetch(gsUrl)
    .then(function(r) { return r.json(); })
    .then(function(data) {
      var count = data && data.message;
      if (count) {
        document.getElementById('citation-count').textContent = count;
      }
    })
    .catch(function() {
      // silently fall back — DOM already shows hardcoded value
    });
})();
</script>
```

This automatically uses the CDN URL or raw GitHub URL based on site config.

## 4. CSS Classes

Add to `_sass/_custom.scss`:

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

## 5. Implementation Order

1. Merge `feat/visual-hierarchy` PR first (so `## 📃 Publications` heading exists)
2. Add CSS to `_sass/_custom.scss`
3. Add HTML block + JS fetch to `_pages/about.md`
