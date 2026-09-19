# Pagbutlak skill test: Website Development brief

This brief is the output of a planning session. It holds the task, the evidence gathered, and the design decisions already made. Treat decisions here as settled unless JP says otherwise. A working reference implementation of the mockup is in `pagbutlak-mockup.html`.

---

## 1. The task

JP (BS Computer Science, UP Visayas) is applying to join Pagbutlak as a web developer through its membership skill test. **Deadline: Sunday midnight. JP wants it finished as soon as possible.**

The Website Development section asks for two deliverables:

1. **A written evaluation of pagbutlak.org**: key strengths and points for improvement (what works, what doesn't, what can be improved), in paragraph format. Technical concepts are welcome but must also be explained in plain language. Graded on vision and suggestions, not on writing.
2. **A mockup webpage** that includes:
   - Header: PAGBUTLAK + navigation menu (Home, News, Features, Opinion, Kultura, Multimedia)
   - A main banner/photo for the featured article or coverage
   - At least three article previews with thumbnails and short excerpts, sourced from Pagbutlak's official site
   - Photos taken from Pagbutlak's official releases on its social media pages
   - Desktop or mobile-first, with clear hierarchy and a user-friendly layout

The lead-writing section is already done (see the appendix).

Source: `C:\Users\JP\Downloads\Copy of [Pagbutlak52] Skill Test Guidelines.pdf`. It gives no word limit and no submission method. The photo rule covers every photo in the layout, including the hero.

---

## 2. About the site

- Pagbutlak is the official student and community publication of the College of Arts and Sciences, UP Visayas. Site: https://pagbutlak.org
- Stack: WordPress.com (with plugins, so a Business/Atomic-type plan) + Jetpack, block theme "nook". The image CDN `i0.wp.com` already works for this domain (the logo is served through it). The REST API is public at `/wp-json/`.
- In practice the site is mainly an **archive**. Readers engage on Facebook (facebook.com/pagbutlakupv) and Instagram (instagram.com/pagbutlakupv). JP recalls the Twitter/X account being suspended.
- The test itself calls the site "a digital outlet and central archive of all publication content."

---

## 3. Evidence

### 3.1 PageSpeed Insights baseline (homepage, September 19, 2026, lab data only)

| | Mobile | Desktop |
|---|---|---|
| Performance | 68 | 76 |
| Accessibility | 100 | 100 |
| Best Practices | 100 | 100 |
| SEO | 100 | 100 |
| Largest Contentful Paint (simulated) | 18.8 s | 5.5 s |
| Largest Contentful Paint (observed, unthrottled) | 0.41 s | 0.35 s |
| Total page weight | 6,181 KiB (about 6 MB) | 6,998 KiB (about 7 MB) |
| Cumulative Layout Shift | 0.097 | 0.014 |
| Total Blocking Time | 20 ms | 0 ms |
| Server response time | 10 ms | 10 ms |

How to read it:

- **Two uploads are about 80% of the page's weight**: `/wp-content/uploads/2026/08/last-resol.png` (2.62 MiB) and `/wp-content/uploads/2026/08/dsc_0323-1.jpg` (2.30 MiB, filename suggests a straight-from-camera original). About 97% of each file's bytes are flagged as unnecessary for the size they're displayed at. Both are served as originals from `/wp-content/uploads/`, not through `i0.wp.com`.
- **The largest element is text** (the Tapatan headline). The 18.8 s figure is Lighthouse simulating a slow (~1.6 Mbps) mobile connection, where the images eat the bandwidth. **Do not write "the site takes 18.8 seconds to load."** The defensible claim is page weight: about 6 MB on a first homepage visit, which is slow on weak signal and costly on mobile data.
- Desktop downloads more only because it shows two more images (`img_9103.jpg`, `img_9205-1.jpg`). That alone doesn't prove responsive images are broken.
- Render-blocking: WordPress.com's bundled CSS files plus jQuery (estimated 990 ms on mobile). There's limited control over these on WordPress.com.
- Fonts: six raw `.ttf` files (Playfair Display, Instrument Sans) with no `font-display`. `.woff2` plus `font-display: swap` is the fix.
- Mobile layout shift comes from the site logo and the "Top stories" block (late font/logo reflow).
- Accessibility 100 means only that the automated checks passed. Ten manual checks (keyboard focus, tab order, landmarks, and so on) weren't run.
- No field data: Chrome's real-user dataset doesn't have enough traffic for this page. This is consistent with readers engaging on Facebook rather than the site.
- On the mobile test screen (412 × 823), the top headline starts about 600 px down.

### 3.2 Category structure (from `/wp-json/wp/v2/categories?per_page=100`)

Numbers are posts filed directly in each category; IDs in brackets.

```
News [103] (31)          → Live [12071] (8), News Bytes [439923] (4), In-Depth [35795] (2)
Features [9548] (16)     → Banwa Narratives [779412454] (5), Kampus [49962] (5), Essay [858] (4), Profiles [15224] (1)
Opinions [2244] (10)     → Column [25515] (15), First Person [215704] (4), Editorial [2697] (3)
Kultura [16731] (8)      → no subsections
Other top-level          → Issues [629] (5), From the Newsroom [5060088] (5), Sports [67] (0), About [256] (0), Uncategorized [1] (1)
```

### 3.3 Reader walkthrough (JP's findings, refined)

Desktop:
1. The nav doesn't show which section you're in (no active state).
2. The header and nav aren't sticky, so readers scroll back to the top to search or switch sections.
3. Hovering a story underlines the headline, but the image, which is also a link, gives no feedback. Worth checking: if the image and the title are separate links to the same article, keyboard and screen-reader users hit every story twice (not yet verified).
4. Empty sections show nothing at all. Sports has zero posts and no message.
5. Search has no suggestions and only shows results after pressing Enter. A "nothing matched" message exists. The results page shows a second search box in the page body.
6. On short pages (like empty sections), the footer floats up instead of staying at the bottom of the screen.

Mobile:
7. The search box in the header isn't scaled properly.
8. The nav items wrap and take up about a third of the first screen.

Dropped after review: skeleton loaders (WordPress sends the text already rendered, so only images arrive late), and "the logo linking home is redundant" (it's a near-universal web convention).

### 3.4 Content and structure findings

- The live menu is nine hand-typed links (Home, News, Features, Opinions, Kultura, Sports, Issues, NewsBytes, About) with no dropdowns, even though the back end has a real hierarchy (3.2). "News" points to a separate `/news/` page rather than the News category.
- **The menu has drifted from the data**: NewsBytes (a News subsection) sits in the top bar, From the Newsroom (5 posts) isn't in the menu at all, and Sports keeps its slot with zero posts.
- Hidden sections: From the Newsroom, plus Kampus and the other subsections, whose pages are reachable only by clicking a label on an article.
- "LIVE:" is typed into headlines while a Live category also exists. They've already fallen out of sync: "LIVE: Student councils set to advance campaigns as GASC enters Day 2" is labeled Issues only. Live labels never expire; the Tapatan post still reads LIVE days after the forum.
- The homepage lists the same six stories twice ("Top stories" and "Latest Stories").
- Excerpts are cut automatically from each article's opening, so they stop mid-sentence, and live posts preview as timestamps ("5:17 PM, August 6 5:21 PM, August 6 …").
- One post is Uncategorized (what WordPress assigns when nobody picks a category).
- The homepage's latest stories jump from August 8 to September 14, 2026. Coverage like First Day Rage (August 17) may exist only on Facebook, which means the archive is incomplete.
- The footer still links to twitter.com/pagbutlakupv (reportedly suspended; verify).

### 3.5 Verified September 20, 2026

- **Duplicate links (3.3 #3) confirmed**: on the homepage, each story's featured image and title are separate links, and one of the two lists also links the date. The image's alt text is the headline, so screen readers hear it twice. One story gets 5 links on the homepage.
- **Bylines aren't structured**: 99 of 102 posts are under the shared account "Pagbutlak UP Visayas CAS" (the others: Rey Mark Paran 2, Phil Liam Nono 1). Writer names are typed into the article body, so there are no writer pages.
- **Article pages send originals too**: the featured image has no `srcset` and loads from `/wp-content/uploads/`. Bontok: 4.28 MB original vs. 0.45 MB WebP at `i0.wp.com/...?w=800`. 15 of 29 Opinion/Features featured images are over 2 MiB (median 2.1, max 6.2).
- **Empty alt on article featured images** (`alt=""` on 6 of 6 checked). Valid markup, so Lighthouse passes it.
- **Opinion and Features have stopped on the site**: newest Opinion May 7, 2026; newest Features-family March 24, 2026. Check Facebook for newer pieces (that would be archive-gap evidence).
- **REST API allows cross-origin reads**: `Access-Control-Allow-Origin` echoes the requesting origin.
- Filing is inconsistent (some posts are under a subsection only, others under the parent too), so REST queries need every child ID: Opinion `2244,25515,215704,2697`; Features `9548,779412454,49962,858,15224`.

---

## 4. The evaluation

### Main argument

The site depends on people remembering: to shrink photos before uploading, update the menu when a section is added, remove LIVE labels after an event, pick a category, and copy Facebook coverage to the site. Each recommendation makes the system remember for them. The Twitter/X suspension is the strongest reason to take the archive seriously: when a platform removes an account, everything posted only there goes with it, and the website is the one copy Pagbutlak controls.

### Strengths (evidence-based; JP to confirm)

- Desktop works normally and is functional.
- Fast server (10 ms response) and very little JavaScript work (20 ms blocking time).
- Automated Accessibility, Best Practices, and SEO checks all score 100.
- A real category hierarchy already exists in the back end; it just isn't exposed.
- An image CDN (`i0.wp.com`) and a public data API are already available at no extra cost.
- Live coverage of events like #GASC61 and #Tapatan 2026 shows timely reporting.

### Recommendations

| Problem | Recommendation | Principle | In plain language |
|---|---|---|---|
| Oversized photos | Resize and convert every image automatically (route through `i0.wp.com` with width parameters and `srcset`, or optimize on upload). Use WordPress's built-in "Expand on click" on article pages for full-resolution viewing. | Performance, data cost | Send each reader a photo the size of their screen, not the camera's original. |
| Slow, shifting fonts | `.woff2` files with `font-display: swap` | Performance, layout stability | Text appears right away instead of waiting for fonts. |
| Hand-typed menu drifts | Build the nav from the category data, with rules: hide empty categories and Uncategorized, keep the six main sections in a fixed order | Single source of truth | The menu updates itself when editors add or empty a section. |
| No "you are here" | Highlight the current section and mark it with `aria-current` | Visibility of system status | Readers always know which section they're in. |
| Mobile nav fills the screen | One swipeable row of sections; a full-screen menu for subsections and search | Progressive disclosure, Fitts's law | Sections stay one tap away without pushing the news down. |
| Nav unreachable after scrolling | Header hides on scroll down and returns on scroll up (NN/g: "partially persistent header") | Efficiency | Scroll up a little and the menu is back. |
| Inconsistent card feedback | The whole card is one link with one hover effect | Consistent feedback, accessibility | Anything that looks clickable behaves the same way. |
| Stale LIVE labels | Generate the badge from the Live category and the post's last update; show "Live coverage ended" afterward | Visibility of system status | The label tells the truth without anyone editing old headlines. |
| Blank empty sections, dead-end searches | Empty states that explain what's happening and offer a next step | Error and empty-state design | An empty page should say why and where to go instead. |
| Search friction | Suggestions as you type; a single search box on the results page, prefilled with the search | Recognition over recall | Readers see matches before they finish typing. |
| Excerpts cut mid-sentence | Hand-written one-sentence excerpts | Scannability | Every preview reads like a complete thought. |
| Duplicate homepage lists | Show each story once | Clear hierarchy | No wasted space on repeats. |
| Incomplete archive | Publish all coverage on the site; link social posts back to the articles | Own your archive | The site is the copy no platform can take away. |
| Dead social link | Remove or fix the Twitter/X link | Trust | Every link should lead somewhere real. |
| Accessibility beyond the score | Manual keyboard and screen-reader testing | WCAG | A perfect automated score isn't the same as accessible. |

### Claims to avoid

- "The site takes 18.8 seconds to load."
- "The site is inaccessible" (automated checks pass; the gap is untested manual checks).
- "Responsive images are broken because desktop loads more bytes."
- Recommending skeleton loaders for the current site.
- Removing the logo-to-homepage link.

---

## 5. Mockup spec

The mockup is a single HTML/CSS/JS page, mobile-first, with a desktop layout from 900 px up. `pagbutlak-mockup.html` implements everything below and was tested at 412 × 823 and 1350 × 940. The photos and fonts were never seen rendered, because the planning sandbox had no internet access, so check them in a real browser first.

**Design tokens**
- Colors: maroon `#7A1027` (header, brand), gold `#C77A12` (active-section marker, focus ring), ink `#22181A`, muted `#6B6264`, lines `#ECE3E4`, wash `#F7F2F3`.
- Section colors, so readers can tell news from opinion: News `#7A1027`, Features `#1E5B4A`, Opinion `#2F3C7E`, Kultura `#8F520F`, Multimedia `#4A4F5A`.
- Type: Newsreader for headlines (a typeface designed for news). Instrument Sans for body and interface, which keeps continuity with the live theme. Both are loaded as woff2 from Google Fonts with `display=swap`.

**Header**
- "Pagbutlak" wordmark on the left, linking home.
- One combined search-and-menu button on the right (BBC-style), with at least a 44 px tap target.
- The header hides as you scroll down and returns as you scroll up. It stays visible while keyboard focus is inside it.

**Section strip**
- Home, News, Features, Opinion, Kultura, Multimedia in one swipeable row.
- The current section is bold, has a gold underline, and carries `aria-current="page"`. It scrolls into view on load.
- The edge fades only when there's more to scroll. The bold width is reserved so the row doesn't jump when the current section changes.

**Full-screen menu**
- Search sits at the top and is focused as soon as the menu opens, with suggestions as you type.
- Accordion sections: tapping a section with subsections expands it (the arrow appears only when there are subsections), and "All News" (and so on) is the first item. Subsections link to the live category pages.
- Secondary links: About, From the Newsroom, Facebook, Instagram.
- Esc closes the menu, focus stays trapped inside while it's open, and focus returns to the button when it closes.
- The strip and the menu are generated from one data array that mirrors the category API. Empty categories (Sports) are hidden automatically. Issues is nested under News as a proposal.

**Featured story (hero)**
- #Tapatan 2026 (News, live coverage, September 14, 2026).
- The headline drops the typed "LIVE:" prefix. The badge is computed: "Live coverage ended" in a muted style, or "Live now" in red with a pulsing dot only while coverage is actually ongoing.

**Previews**
- Three cards. Each has a thumbnail (square on mobile, 3:2 on desktop), one section label (for example "Opinion / Column"), a serif headline, a one- to two-line excerpt, and the date, plus a byline on opinion pieces.
- The whole card is one link, the hover effect covers the whole card, and keyboard focus draws a ring around the card.
- The current previews are all News (the 43rd Student Regent selections, August 8; the PUV timeline, August 7; the #GASC61 resolution, August 7) and should be swapped (see section 6).

**Empty and search states**
- Multimedia has no content on the live site, so it shows an empty state with "See photos on Facebook" and "Back to latest stories."
- Searches with no results explain what happened and offer the sections as a next step.
- The results page has a single search box, prefilled with the search, and a result count.

**Footer**
- Stays at the bottom on short pages.
- Links: About, From the Newsroom, Facebook, Instagram. No Twitter/X link until the account is verified.

**Performance and accessibility**
- Images load through `i0.wp.com` with `?w=` and `srcset`/`sizes`. Everything below the fold is lazy-loaded, and the hero uses `fetchpriority="high"`. Every image sits in a reserved aspect-ratio box with a placeholder color, and broken images hide gracefully.
- A skip link, visible focus styles, tap targets of 44 px or more, and respect for reduced-motion settings.
- `aria-expanded` on accordion toggles.
- When the view changes, focus moves to the new heading.

### Content and data

- Posts by category (public): `https://pagbutlak.org/wp-json/wp/v2/posts?categories=<ID>&per_page=5&_embed` (IDs in 3.2). Note that this lists posts filed directly under that category.
- Picking previews: the top story per section, with news first. Popularity comes from Facebook reactions and shares, since the site itself has no engagement data.
- Photo rule: every photo in the mockup must also appear in an official Pagbutlak Facebook or Instagram post. The Tapatan photo's filename follows Facebook's naming format.
- Every story links to the real article on pagbutlak.org. Don't invent content.

---

## 6. Open items

- [x] Swap in the top Opinion and Features stories. Done September 20: JP chose the most recent (#1234 Opinion/First Person, #1141 Features/Kampus), after the Student Regent card. The section is now titled "Top stories."
- [x] Verify that all photos appear on official Pagbutlak Facebook or Instagram posts. JP confirmed all four homepage photos on September 20: the Tapatan banner, the Student Regent card, `2026/05/for-site.png` (#1234), and `2026/03/for-site.png` (#1141).
- [x] Check the Twitter/X account: it has been suspended since December 2025 (confirmed by JP). The live footer still links to twitter.com/pagbutlakupv; the mockup has no Twitter/X link.
- [x] Open the mockup in a real browser and check the photos and fonts. Done September 20: both fonts load as woff2, and all photos load through `i0.wp.com` at mobile and desktop widths. Text-heavy graphics (for example `last-resol.png`) lose text at the 3:2 crop.
- [x] Deploy for a shareable link. Version 2 (live loading) is live at https://pagbutlak-mockup.vercel.app (project `pagbutlak-mockup`, scope `ysxiaixsys-projects`) and verified byte-identical to the local file on September 20. The Vercel MCP tools can't update the project, so use the CLI (JP is logged in): copy the mockup to `index.html` in a folder containing only that file plus `.vercelignore` (`*` / `!index.html`), then run `npx vercel deploy --prod --yes --scope ysxiaixsys-projects`. `vercel link` writes a `.env.local` holding a token; delete it.
- [ ] Revise `evaluation.md` (JP, September 20): "a bit incomplete and might be too verbose/technical." Plan the revision with JP before editing; JP hasn't said yet what feels missing.
- [x] Write the evaluation and decide the submission format. Both are Markdown: `evaluation.md` (final, about 1,950 words, links the mockup) and `lead-writing.md`. The Facebook-gap sentence now states only what the site shows, because JP couldn't confirm Facebook.
- [x] Load stories live from the REST API. Built September 20: section pages, in-mockup subsection routes (`#/opinion/column`), "Load more," and search/suggestions all use the API. Lists use the homepage story cards with thumbnails from `jetpack_featured_media_url`, resized through `i0.wp.com`. Posts whose content credits an outside photo source get a "Pagbutlak" tile instead (`OUTSIDE_PHOTOS`, 13 IDs from a scan of all 102 posts on September 20; recheck when new posts are added). Excerpts are never clipped by CSS. An API post shows its first complete sentence only if it's 200 characters or fewer (71 of 102 posts), otherwise just the headline. The host rate-limits bursts with HTTP 429 (no CORS header, so the browser reports "Failed to fetch"), so requests retry once after 1.5 s and then show an error with "Try again." Test sparingly. Local test server: `.claude/launch.json` → `mockup` (http://localhost:5173/pagbutlak-mockup.html); `file://` previews block hash routes.

---

## Appendix: lead writing (done, submitted separately)

**First Day Rage sets challenges for incoming UPV chancellor**

Student organizations and formations led by the UPV University Student Council (USC) staged the annual First Day Rage on August 17 at the UPV Iloilo City Campus, laying down challenges for the incoming chancellor as the academic year opened. UPV held its opening exercises at the City Campus for the first time in recent years, and sectoral groups joined the lightning protest as it proceeded outside the campus.
