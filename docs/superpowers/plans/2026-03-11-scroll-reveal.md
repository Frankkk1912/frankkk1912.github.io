# Scroll Reveal Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add scroll-triggered fade-in-up animations to section headings, stat cards, and education timeline items.

**Architecture:** CSS transition on `.fade-in-up` class; `.visible` class added by an `IntersectionObserver` when elements enter the viewport. h2 headings get the class via JS (avoids markdown churn); `.edu-item` and `.stat-cards` get it directly in HTML. All three are observed inside the existing `DOMContentLoaded` callback.

**Tech Stack:** Jekyll, SCSS, vanilla JS (`IntersectionObserver`).

---

## Chunk 1: CSS

### Task 1: Replace `.fade-in-up` in `_sass/_custom.scss`

**Files:**
- Modify: `_sass/_custom.scss` (~line 548)

An existing `.fade-in-up` block already exists (uses `.animate` trigger, `translateY(30px)`, `0.6s`). It must be **replaced**, not appended, to avoid two conflicting definitions.

- [ ] **Step 1: Find and replace the existing block**

  In `_sass/_custom.scss`, find:

  ```scss
  .fade-in-up {
    opacity: 0;
    transform: translateY(30px);
    transition: all 0.6s ease;

    &.animate {
      opacity: 1;
      transform: translateY(0);
    }
  }
  ```

  Replace with:

  ```scss
  // Scroll reveal — triggered by IntersectionObserver adding .visible
  .fade-in-up {
    opacity: 0;
    transform: translateY(8px);
    transition: opacity 0.4s ease-out, transform 0.4s ease-out;

    &.visible {
      opacity: 1;
      transform: translateY(0);
    }
  }

  // Respect user's reduced-motion preference — show all content immediately
  @media (prefers-reduced-motion: reduce) {
    .fade-in-up {
      opacity: 1;
      transform: none;
      transition: none;
    }
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
  git commit -m "feat(style): replace fade-in-up with IntersectionObserver-triggered animation"
  ```

---

## Chunk 2: HTML — add classes to animated elements

### Task 2: Add `fade-in-up` to `.edu-item` and `.stat-cards` in `_pages/about.md`

**Files:**
- Modify: `_pages/about.md`

- [ ] **Step 1: Add classes to education timeline items**

  In `_pages/about.md`, find and replace the three `.edu-item` divs (lines ~81–105). Add `fade-in-up` class and `data-reveal-delay` attribute to each:

  **Find:**
  ```html
  <div class="edu-item">
    <div class="edu-dot"></div>
    <div class="edu-body">
      <div class="edu-date">2025.09 – Present</div>
      <div class="edu-school">Sun Yat-sen University (SYSU)</div>
      <div class="edu-degree">PhD Candidate in Pharmacology</div>
    </div>
  </div>
  <div class="edu-item">
    <div class="edu-dot"></div>
    <div class="edu-body">
      <div class="edu-date">2022.09 – 2025.06</div>
      <div class="edu-school">Sun Yat-sen University (SYSU)</div>
      <div class="edu-degree">Master in Pharmacology</div>
    </div>
  </div>
  <div class="edu-item">
    <div class="edu-dot"></div>
    <div class="edu-body">
      <div class="edu-date">2018.06 – 2022.09</div>
      <div class="edu-school">Southern Medical University</div>
      <div class="edu-degree">Bachelor</div>
    </div>
  </div>
  ```

  **Replace with:**
  ```html
  <div class="edu-item fade-in-up" data-reveal-delay="0">
    <div class="edu-dot"></div>
    <div class="edu-body">
      <div class="edu-date">2025.09 – Present</div>
      <div class="edu-school">Sun Yat-sen University (SYSU)</div>
      <div class="edu-degree">PhD Candidate in Pharmacology</div>
    </div>
  </div>
  <div class="edu-item fade-in-up" data-reveal-delay="100">
    <div class="edu-dot"></div>
    <div class="edu-body">
      <div class="edu-date">2022.09 – 2025.06</div>
      <div class="edu-school">Sun Yat-sen University (SYSU)</div>
      <div class="edu-degree">Master in Pharmacology</div>
    </div>
  </div>
  <div class="edu-item fade-in-up" data-reveal-delay="200">
    <div class="edu-dot"></div>
    <div class="edu-body">
      <div class="edu-date">2018.06 – 2022.09</div>
      <div class="edu-school">Southern Medical University</div>
      <div class="edu-degree">Bachelor</div>
    </div>
  </div>
  ```

- [ ] **Step 2: Add `fade-in-up` to `.stat-cards`**

  Find:
  ```html
  <div class="stat-cards">
  ```

  Replace with:
  ```html
  <div class="stat-cards fade-in-up">
  ```

- [ ] **Step 3: Build and verify**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll build 2>&1 | tail -3
  ```
  Expected: clean build.

  Confirm classes appear in output:
  ```bash
  grep "fade-in-up" _site/index.html | wc -l
  ```
  Expected: at least `4` (3 edu-items + 1 stat-cards).

- [ ] **Step 4: Commit**

  ```bash
  git add _pages/about.md
  git commit -m "feat(content): add fade-in-up classes to edu-items and stat-cards"
  ```

---

## Chunk 3: JavaScript — IntersectionObserver

### Task 3: Add observer inside DOMContentLoaded callback

**Files:**
- Modify: `_pages/about.md`

The existing `<script>` block has two parts:
1. A `document.addEventListener('DOMContentLoaded', function() { ... });` block that closes at line ~331
2. A citation-fetch IIFE starting at line ~333

The observer code goes **inside** the `DOMContentLoaded` callback — before its closing `});` on line ~331.

- [ ] **Step 1: Insert observer code**

  Find (the closing lines of the DOMContentLoaded callback, before `});`):
  ```js
      innerBadges.forEach(badge => {
        if (activeTags.has(badge.textContent)) {
          badge.classList.add('active');
        } else {
          badge.classList.remove('active');
        }
      });
    });
  }
});
  ```

  Replace with:
  ```js
      innerBadges.forEach(badge => {
        if (activeTags.has(badge.textContent)) {
          badge.classList.add('active');
        } else {
          badge.classList.remove('active');
        }
      });
    });
  }

  // Scroll reveal — IntersectionObserver
  // h2 headings get .fade-in-up via JS to avoid markdown churn
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
});
  ```

- [ ] **Step 2: Build and verify**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll build 2>&1 | tail -3
  ```
  Expected: clean build with no Liquid or JS errors.

  Confirm `revealObserver` appears in built output:
  ```bash
  grep "revealObserver" _site/index.html | wc -l
  ```
  Expected: `4` (declaration + unobserve + observe + reference inside callback).

- [ ] **Step 3: Commit**

  ```bash
  git add _pages/about.md
  git commit -m "feat(js): add IntersectionObserver scroll reveal for headings, stat cards, timeline"
  ```

---

## Chunk 4: Final verification

- [ ] **Serve locally**

  ```bash
  eval "$(rbenv init -)" && bundle exec jekyll serve --livereload
  ```

  Open `http://localhost:4000` and verify:
  1. Elements below the fold start **invisible** (opacity 0) before scrolling — check via DevTools Elements panel
  2. Scroll down — section headings (`h2`) fade in smoothly as they enter viewport
  3. Stat cards fade in as a group above Publications
  4. Education timeline items appear one by one with ~100ms stagger between each
  5. No layout shift or content jumping during animation

- [ ] **Reduced motion check**

  In macOS: System Settings → Accessibility → Display → Reduce Motion → ON.
  Reload `http://localhost:4000` — all content should be immediately visible, no transitions.

- [ ] **Final commit if any fixes needed**

  ```bash
  git add _sass/_custom.scss _pages/about.md
  git commit -m "fix(style): scroll reveal adjustments"
  ```
