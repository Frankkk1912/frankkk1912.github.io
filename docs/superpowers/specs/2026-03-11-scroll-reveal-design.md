# Scroll Reveal Animation Design — Item 5

**Date:** 2026-03-11
**Scope:** Scroll-triggered fade-in-up animation for section headings, stat cards, and education timeline
**Files affected:** `_pages/about.md`, `_sass/_custom.scss`

---

## 1. Animation Parameters

| Property | Value |
|----------|-------|
| Effect | `opacity: 0→1` + `translateY(8px→0)` |
| Duration | `0.4s` |
| Easing | `ease-out` |
| Trigger | `IntersectionObserver` (fires once per element) |
| Accessibility | `prefers-reduced-motion: reduce` disables all transitions |

## 2. Elements Animated

| Element | Selector | Stagger |
|---------|----------|---------|
| Section headings | `.page__content h2` | None — each triggers independently on scroll |
| Stat cards (group) | `.stat-cards` | None — all 3 cards enter together as one unit |
| Education timeline items | `.edu-item` | 0.1s delay between each item |

**Not animated:** Hero block (above fold), highlight cards (too close to top), paper cards (10+, too many).

## 3. CSS

Add to `_sass/_custom.scss`:

```scss
// -------------------------------------------------------------
// Scroll Reveal — Fade In Up
// -------------------------------------------------------------

.fade-in-up {
  opacity: 0;
  transform: translateY(8px);
  transition: opacity 0.4s ease-out, transform 0.4s ease-out;
}

.fade-in-up.visible {
  opacity: 1;
  transform: translateY(0);
}

@media (prefers-reduced-motion: reduce) {
  .fade-in-up {
    opacity: 1;
    transform: none;
    transition: none;
  }
}
```

## 4. HTML Changes (`_pages/about.md`)

Add `fade-in-up` class to:

1. **Each `h2` section heading** — Minimal Mistakes renders markdown `##` headings as plain `<h2>` tags. Since we cannot add classes to markdown headings directly, we either:
   - Convert the markdown headings to explicit HTML `<h2 class="fade-in-up">` tags, **or**
   - Apply the class via JS (simpler, no HTML churn)
   - **Decision: apply via JS** — the observer script adds `.fade-in-up` to all `.page__content h2` elements on `DOMContentLoaded`, before observing them. This keeps `about.md` clean.

2. **`.stat-cards` div** — add `fade-in-up` directly in HTML:
   ```html
   <div class="stat-cards fade-in-up">
   ```

3. **Each `.edu-item` div** — add `fade-in-up` directly in HTML:
   ```html
   <div class="edu-item fade-in-up">
   ```

## 5. JavaScript

Add **inside** the existing `document.addEventListener('DOMContentLoaded', function() { ... })` callback in `_pages/about.md`, as the last block before its closing `}`. Placing it inside `DOMContentLoaded` ensures the DOM is ready before querying elements.

```js
  // Scroll reveal — IntersectionObserver
  // Apply fade-in-up to h2 headings via JS (avoids HTML churn in markdown)
  document.querySelectorAll('.page__content h2').forEach(function(el) {
    el.classList.add('fade-in-up');
  });

  var revealObserver = new IntersectionObserver(function(entries) {
    entries.forEach(function(entry) {
      if (entry.isIntersecting) {
        var el = entry.target;
        var delay = parseInt(el.dataset.revealDelay || '0', 10);
        setTimeout(function() {
          el.classList.add('visible');
        }, delay);
        revealObserver.unobserve(el);
      }
    });
  }, { threshold: 0.1 });

  document.querySelectorAll('.fade-in-up').forEach(function(el) {
    revealObserver.observe(el);
  });
```

Note: stat cards (`.stat-cards.fade-in-up`) are observed as one unit — the three individual `.stat-card` children are not independently animated.

**Stagger for `.edu-item`:** Each `.edu-item` gets a `data-reveal-delay` attribute set in HTML:
```html
<div class="edu-item fade-in-up" data-reveal-delay="0">...</div>
<div class="edu-item fade-in-up" data-reveal-delay="100">...</div>
<div class="edu-item fade-in-up" data-reveal-delay="200">...</div>
```

The JS reads `el.dataset.revealDelay` (in ms) and applies a `setTimeout` before adding `.visible`.

## 6. Implementation Order

1. **Replace** the existing `.fade-in-up` block in `_sass/_custom.scss` (currently at ~line 548, uses `translateY(30px)` / `0.6s` / `.animate` trigger) with the new definition from Section 3. Do not append — replacing is required to avoid two conflicting definitions.
2. Add `fade-in-up` class + `data-reveal-delay` to `.edu-item` divs in `_pages/about.md`
3. Add `fade-in-up` class to `.stat-cards` div in `_pages/about.md`
4. Add IntersectionObserver JS inside the existing `document.addEventListener('DOMContentLoaded', ...)` callback in `_pages/about.md` — place it as the last block inside that callback, before its closing `}`

## 7. Verification

After `bundle exec jekyll serve`:
1. Open `http://localhost:4000` — all elements above fold visible immediately (no flash of invisible content)
2. Scroll down — section headings, stat cards, education items fade in smoothly
3. Education items stagger: first appears, then 100ms later second, then 200ms later third
4. Resize to mobile — animations still work
5. In macOS System Preferences → Accessibility → Reduce Motion → ON — page should show all content immediately with no transitions
