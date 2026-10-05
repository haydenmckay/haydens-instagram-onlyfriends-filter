# Haydens Instagram OnlyFriends Filter — Brave Shields Filter

A custom filter list for Brave Shields that strips Instagram mobile web down to what matters: your friends' posts, stories, messages, and profile — nothing else.

**Version 1.1 — Verified October 2026 (Brave iOS)**

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
| Suggested posts in feed (incl. "Suggested for you" collabs) | ❌ Hidden |
| Ads in feed | ❌ Hidden |

---

## How to apply

### Option 1 — Subscribe via URL (easiest, stays up to date automatically)

1. Open Brave → `brave://settings/shields`
2. Scroll to **Content filtering** → **Add custom filter list**
3. Paste this URL:
   ```
   https://raw.githubusercontent.com/haydenmckay/haydens-instagram-onlyfriends-filter/main/haydens_instagram_onlyfriends.txt
   ```
4. Click **Add**
5. Reload `instagram.com`

Works on desktop and mobile (Brave iOS/Android). Any updates pushed to this repo will sync automatically.

### Option 2 — Paste manually

1. Open Brave → `brave://settings/shields`
2. Scroll to **Content filtering** → **Create custom filters**
3. Paste the contents of `haydens_instagram_onlyfriends.txt`
4. Click **Save**
5. Reload `instagram.com`

---

## How it was built

The filter was built by inspecting Instagram's mobile DOM using Brave DevTools in mobile emulation mode (iPhone 12 Pro, 390px). Instagram uses Meta's Stylex atomic CSS system, which generates single-property class names (e.g. `x1n2onr6`) that change frequently. Where possible, rules were anchored to stable attributes instead.

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
www.instagram.com##div:has(div.x1miatn0) ~ article:style(...)
www.instagram.com##div[role="button"]:has-text(/^Follow$/):upward(article):style(...)
www.instagram.com##span:has-text(/^Suggested for you$/):upward(article):style(...)
```
- **After the "caught up" banner:** every article after it is suggested, so all of them are hidden.
- **Follow button:** suggested posts from single accounts show a Follow button; friends' posts don't.
- **"Suggested for you" collabs:** collab posts ("A and B" byline) have no Follow button. The line under the username *looks* like it rotates between "Suggested for you" and "<audio> • Original audio", but both are in the DOM the whole time and only their visibility animates. So a `<span>` whose text is exactly "Suggested for you" is present on every suggested post and never on friends' posts, including friends' collabs. No non-text anchor exists: classes, `data-*`, aria-labels, roles and hrefs on the post header are the same for suggested and friends' posts (checked Oct 2026).

**What didn't work:**
- `div[data-reel-type="suggested"]` — Instagram no longer adds this data attribute
- `article:has(span.x193iq5w...)` — that class combo is shared with friends' usernames and captions
- Hiding with plain `##` (display:none) — see *Collapsing, not hiding* below

#### Collapsing, not hiding

Every rule that hides a feed post ends in:
```
:style(height: 40px !important; min-height: 0 !important; overflow: hidden !important; visibility: hidden !important)
```
Instagram's home feed is a virtualized list: posts that are off screen are replaced by one big padding block. A post hidden with `display:none` measures 0px, which breaks the list's position math. You end up scrolled into the empty padding and the feed goes permanently black. Shrinking hidden posts to an invisible 40px strip keeps them measurable, so the feed keeps loading and your friends' posts still sit almost back to back.

| Method tested | Result |
|---|---|
| `display:none` | Feed went black within ~15–25s on a suggestion-heavy account |
| `visibility: hidden` | Stable, but leaves a post-sized blank gap per hidden post |
| 40px invisible strip (used) | Near-stacked feed; stable on iPhone in normal use |

If the black feed comes back, change the `:style()` to just `visibility: hidden !important` (stable but gappy).

#### 4. Hide ads
```
www.instagram.com##article:has(a[href*="ads/ig_redirect"]):style(...)
```
Sponsored posts contain a `facebook.com/ads/ig_redirect` URL as their ad click tracker. This is tied to Meta's ad infrastructure rather than any CSS class, making it robust against redeployments.

#### 5. Pinch-zoom jump fix (iOS)
```
www.instagram.com##html:style(overflow-anchor: none !important)
www.instagram.com##body:style(overflow-anchor: none !important)
```
Pinch-zooming a photo made the page jump to a different post. Turning off scroll anchoring on the page roots reduced this on iPhone. It can't be reproduced in desktop emulation, so it's only been checked on a phone.

---

## Maintenance

Instagram's atomic CSS class names change with each deploy. Rules that rely on class names (currently only the explore grid rule) may stop working after updates. When a rule breaks:

1. Open instagram.com in Brave with DevTools → mobile emulation (iPhone 12 Pro)
2. Use the element inspector to find the relevant container
3. Find a stable attribute (`href`, `data-*`, `aria-label`, text content) to anchor to instead of classes
4. Update the rule

Href-based and `:has-text()`-based rules should remain stable long-term.

---

## Tested on

- Brave iOS on iPhone 12, Instagram mobile web, October 2026
- Brave desktop in mobile emulation (June 2026; rule logic re-checked October 2026)
