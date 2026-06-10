# Instagram Focused Feed — Brave Shields Filter

A custom filter list for Brave Shields that strips Instagram mobile web down to what matters: your friends' posts, stories, messages, and profile — nothing else.

**Version 1.0 — Verified June 2026**

---

## What it does

| Feature | Status |
|---|---|
| Home feed (friends' posts + stories) | ✅ Visible |
| Messages | ✅ Visible |
| Profile | ✅ Visible |
| Search tab (can type names) | ✅ Visible |
| Reels tab | ❌ Hidden |
| Explore discovery grid | ❌ Hidden |
| Suggested posts in feed | ❌ Hidden |
| Ads in feed | ❌ Hidden |

---

## How to apply

1. Open Brave → `brave://settings/shields`
2. Scroll to **Content filtering** → **Create custom filters**
3. Paste the contents of `instagram_brave_filter.txt`
4. Click **Save**
5. Reload `instagram.com`

For mobile: apply via Brave on Android/iOS (same location in Shields settings).

---

## How it was built

The filter was built by reverse-engineering Instagram's mobile DOM using Brave DevTools in mobile emulation mode (iPhone 12 Pro, 390px). Instagram uses Meta's Stylex atomic CSS system, which generates obfuscated single-property class names (e.g. `x1n2onr6`) that change frequently. Where possible, rules were anchored to stable attributes instead.

### Rule breakdown

#### 1. Hide Reels tab
```
www.instagram.com##a[href="/reels/"]
```
The bottom nav links use `href` attributes tied to route paths. These are stable indefinitely — Instagram would have to change its URL structure to break this rule. Class-based nav targeting was tried first but abandoned because Meta's atomic CSS classes change with each deploy.

#### 2. Hide explore discovery grid
```
www.instagram.com##div.xph46j
```
On the explore page, the search input lives in a fixed header outside `[role="main"]`. The discovery grid below it sits in a container with class `xph46j`. This class was verified absent from the home feed DOM (June 2026), making it safe to target globally without URL-path scoping. If it breaks, inspect `[role="main"] > div > div > div:first-child` on `/explore/` and update the class.

**What didn't work:**
- `www.instagram.com/explore/##[role="main"]` — hid the entire main content area, causing a black screen because Brave doesn't properly scope cosmetic filters by URL path
- `www.instagram.com/explore/##[role="main"] > div > div > div:first-child` — same black screen issue

#### 3. Hide suggested posts
```
www.instagram.com##article:has-text(Suggested post)
```
Instagram labels suggested posts with the visible text "Suggested post" inside the article. `:has-text()` matches the full text content of the article element. This is the most durable approach since user-facing label text is far more stable than CSS class names.

**What didn't work:**
- `div[data-reel-type="suggested"]` — Instagram no longer adds this data attribute (may have worked in earlier versions)
- `article:has(span:has-text(Suggested post))` — nested procedural filter partially worked but missed some posts
- `article:has(span.x193iq5w.xeuugli.x1fj9vlw.x13faqbe.x1vvkbs.x1i0vuye)` — class combo is shared with username/caption text in regular friend posts, causing friend posts to be hidden too

#### 4. Hide ads
```
www.instagram.com##article:has(a[href*="ads/ig_redirect"])
```
Sponsored posts contain a `facebook.com/ads/ig_redirect` URL as their ad click tracker. This is tied to Meta's ad infrastructure rather than any CSS class, making it robust against redeployments.

---

## Maintenance

Instagram's atomic CSS class names change with each deploy. Rules that rely on class names (currently only the explore grid rule) may stop working after updates. When a rule breaks:

1. Open instagram.com in Brave with DevTools → mobile emulation (iPhone 12 Pro)
2. Inspect the relevant element
3. Find a stable attribute (`href`, `data-*`, `aria-label`, text content) to anchor to instead of classes
4. Update the rule

Href-based and `:has-text()`-based rules should remain stable long-term.

---

## Tested on

- Brave browser (desktop + mobile emulation)
- Instagram mobile web, June 2026
