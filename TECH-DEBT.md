# Tech Debt & Pre-Publication Audit
**The Gain Stage — audit date: 2026-05-26**

Goal: publish to GitHub Pages with a custom domain today. Items are grouped by priority.

---

## 1. GitHub Pages Deployment Blockers

These must be resolved before the site can go live with a custom domain.

| Issue | Detail |
|---|---|
| No `CNAME` file | GitHub Pages requires a `CNAME` file at the repo root containing the custom domain (e.g. `thegainstagetc.org`) |
| No `.gitignore` | `.DS_Store` is tracked and committed — it will be served as a public file |
| `.DS_Store` in git history | Must be removed from tracking with `git rm --cached .DS_Store` before publishing |
| GitHub Pages not configured | The repo needs Pages enabled in Settings → Pages, pointed at `main` branch, root directory |
| DNS not configured | Registrar must have an `A` record or `CNAME` pointing to GitHub Pages servers |

---

## 2. Unused / Template Files (Delete Before Publishing)

These three files are unmodified HTML5 UP template demos. They contain Lorem Ipsum, a Nashville TN fake address, "Strongly Typed by HTML5 UP" branding, and "Untitled" copyright. None of them are linked from any real page. Publishing them would look unprofessional and confuse visitors who land on them via search.

- `left-sidebar.html`
- `right-sidebar.html`
- `no-sidebar.html`

---

## 3. Unused Images

All `pic0*.jpg` stock images are only referenced from the three template files above. Once those are deleted, every stock image becomes orphaned.

- `images/pic01.jpg` — unreferenced anywhere in real pages
- `images/pic02.jpg` — unreferenced anywhere in real pages
- `images/pic03.jpg` — unreferenced anywhere in real pages
- `images/pic04.jpg` — only in template files
- `images/pic05.jpg` — unreferenced anywhere in real pages
- `images/pic06.jpg` — only in template files
- `images/pic07.jpg` — only in template files

`images/banner.jpg` and `images/TGS Logo v5.png (1).png` are actively used.

---

## 4. Broken & Stub Links

### `href="#"` (goes nowhere)
Every page in the footer has Instagram and Facebook displayed as text inside a stub link:
```html
<a href="#">instagram.com/thegainstagetc</a>
<a href="#">facebook.com/thegainstagetc</a>
```
The text looks like a URL but clicking does nothing. Affects: `index.html`, `about.html`, `programs.html`, `industry.html`.

### `industry.html` — all CTA buttons are dead
Both action groups on this page use `href="#"`. Eight buttons total:
- "Collaborate With Us", "Lead a Program", "Join the Founding Leadership" (first article)
- "Lead a Workshop", "Join the Advisory Circle", "Become a Consortium Partner", "Apply for Board Leadership", "Stay Connected" (Call to Action article)

### `workforce.html` — `#partner` anchor does not exist
Five buttons reference `#partner` but no element on the page has `id="partner"`:
- "Partner With Us" (hero)
- "Volunteer", "Partner With Us", "Donate" (Call to Action section)

`#programs` and `#involved` are valid anchors and work correctly.

---

## 5. WCAG AA Accessibility Failures

### A. `user-scalable=no` on all pages — WCAG 1.4.4 (Level AA)
Every page sets `<meta name="viewport" content="..., user-scalable=no" />`. This blocks pinch-to-zoom on mobile, which is a hard WCAG AA failure. The `user-scalable=no` value must be removed from all pages.

### B. Body text color `#777` on `#f0f0f0` — WCAG 1.4.3 (Level AA)
Contrast ratio: **~3.49:1**. WCAG AA requires 4.5:1 for normal-sized text. This applies to the global body text across the entire site (`main.css:103`).

### C. Heading color `#888` — WCAG 1.4.3 (Level AA)
- On `#fff` background: **~3.19:1** — fails for all text sizes below 18pt bold / 24pt regular
- On `#f0f0f0` background: **~2.82:1** — fails for all text sizes
Applied globally to `h1`–`h6` via `main.css:111`.

### D. Brand color `#ed786a` used as text — WCAG 1.4.3 (Level AA)
Contrast ratio on white: **~2.61:1**. Fails even the 3:1 threshold for large text.
Used for: link hover color (`main.css:153`), `.step-content strong`, `.step-number` text background, `h3` in `workforce.html` program cards.

### E. Multiple `<h1>` elements on `workforce.html`
The page has `<h1 id="logo">` in the nav and `<h1 class="hero-title">` in the hero. Only one `<h1>` per page is valid for screen reader document structure. The hero heading should be `<h2>`.

### F. No skip-navigation link
Keyboard-only users must tab through the entire nav on every page load. A "Skip to main content" link at the top of each page is required for WCAG 2.4.1 (Level A).

---

## 6. Missing HTML Head Elements (All Real Pages)

| Missing element | Impact |
|---|---|
| `<html lang="en">` | Missing on `index.html`, `about.html`, `programs.html`, `industry.html` — only `workforce.html` has it. Required for screen readers and WCAG 3.1.1 (Level A). |
| `<meta name="description">` | Missing on all pages. Affects SEO, link previews (Slack, iMessage, social). |
| `<link rel="icon">` (favicon) | Missing on all pages. Browsers show a blank tab icon. |

---

## 7. Structural / Anti-Pattern Issues

### `workforce.html` is missing JS scripts
The page has no `<script>` tags. Consequences:
- `is-preload` class is never removed → all CSS transitions and animations are permanently suppressed
- Mobile nav panel (`#navPanel`) never initializes → hamburger menu is broken on mobile
- `dropotron` never loads → nav behavior is inconsistent with other pages

### `workforce.html` is missing the site footer
Every other real page has the standard footer with contact info and social links. `workforce.html` ends at `</div><!-- /#page-wrapper -->` with no footer.

### `workforce.html` uses a `<header>` element as a hero section
`<header class="hero-section">` is a landmark element; using it as a visual marketing hero is semantically incorrect. It should be `<section class="hero-section">`.

### `workforce.html` has 250+ lines of inline `<style>`
Page-specific styles are embedded in a `<style>` block in `<head>`. These should live in `main.css` or a separate `workforce.css` file so they are cacheable. The inline block also re-fixes WCAG contrast issues that stem from the base theme — fixing the root colors in `main.css` would eliminate the need for these overrides.

### Image filename has spaces and parentheses
`images/TGS Logo v5.png (1).png` — spaces and parentheses in filenames require percent-encoding (`%20`, `%28`, `%29`). Some static servers and CDNs handle this poorly, and it's already being referenced without encoding in every HTML page. Should be renamed to something like `images/tgs-logo.png`.

### `industry.html` contains internal planning notes as public content
The final article ("Why This Is Strategically Important", lines 224–238) reads as internal brainstorming output, not public-facing copy. It includes phrases like "This also dovetails beautifully with your Emerald ecosystem thinking — without conflating entities" and bullet points addressed to the author ("✔ You align with NIVA data"). This section should be removed or rewritten before publishing.

### `industry.html` header subtitle is inconsistent
All other pages use: `"The stage is bigger than the spotlight."`
`industry.html` uses: `"Building the Creative Workforce — and Strengthening Independent Infrastructure."`
The subtitle is also used as the first `<h2>` inside the page content — so it appears twice.

### `index.html`: two CTA buttons use generic icon (`fa-file`)
```html
<a href="about.html" class="button icon solid fa-file">Get Involved</a>
<a href="programs.html" class="button icon solid fa-file">See What's Happening</a>
```
`fa-file` is the wrong icon for both of these actions.

### Global `p { text-align: center }` in `main.css:194`
This is an unusual global reset that forces every paragraph to be centered by default. Most pages then override this with additional styles. Left-aligned body text is the web convention for legibility; centering long-form paragraphs degrades readability.

### CSS `!important` overuse in industry page styles
`main.css:2028–2105` uses `!important` 8+ times in `body.industry` selectors to force text alignment. This is a sign the template's defaults are being fought rather than properly overridden.

### Dead CSS in `#banner` rule block
`main.css:2418` sets `background: #fff`, immediately overridden on line 2426 by `background: url("../../images/banner.jpg")` within the same rule block. The first declaration is dead.

### SASS source committed but no build process documented
`assets/sass/` contains SCSS source files that generate `main.css`. However, there is no `package.json`, build script, or README instruction explaining how to compile. Future edits to `.scss` files will have no effect on the live site without a known build step.

### Google Fonts loaded over the network
`main.css:2` loads `Source Sans Pro` from `fonts.googleapis.com`. For a static site that may be cached offline or accessed in low-connectivity environments, self-hosting the fonts would be more reliable and avoids a third-party dependency.

---

## 8. GitHub Pages Publishing Checklist

Steps required (in suggested order):

1. [ ] Add `.gitignore` with at minimum `*.DS_Store`
2. [ ] Remove `.DS_Store` from git tracking
3. [ ] Delete template files: `left-sidebar.html`, `right-sidebar.html`, `no-sidebar.html`
4. [ ] Delete unused stock images: `pic01`–`pic07`
5. [ ] Rename logo image to remove spaces/parentheses from filename, update all references
6. [ ] Add `CNAME` file with custom domain
7. [ ] Add `404.html` (simple page with nav and "Page not found" message)
8. [ ] Enable GitHub Pages in repo Settings → Pages → `main` branch, root `/`
9. [ ] Configure DNS at registrar to point to GitHub Pages
10. [ ] Verify custom domain is resolving before marking done
