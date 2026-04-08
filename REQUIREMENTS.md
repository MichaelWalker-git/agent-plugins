# Technical Requirements: 2026 Website Redesign & Development

> Developer-focused translation of the 2026 RFP for a nonprofit website serving ~25,000–30,000 annual users.
> Platform: WordPress. Audience: general public, donors, program participants.

---

## Table of Contents

1. [Accessibility — WCAG 2.1 AA](#1-accessibility--wcag-21-aa)
2. [SEO & AI-Search Optimization](#2-seo--ai-search-optimization)
3. [WordPress Theme & Plugin Requirements](#3-wordpress-theme--plugin-requirements)
4. [CRM & Email Marketing Integration](#4-crm--email-marketing-integration)
5. [Security Hardening](#5-security-hardening)
6. [Performance Targets (Core Web Vitals)](#6-performance-targets-core-web-vitals)
7. [Content Migration Plan](#7-content-migration-plan)

---

## 1. Accessibility — WCAG 2.1 AA

**Standard:** [WCAG 2.1 Level AA](https://www.w3.org/TR/WCAG21/)

### Perceivable

- All non-text content (images, icons, infographics) must have descriptive `alt` text; decorative images use `alt=""`.
- Videos require closed captions (auto-generated captions must be reviewed/corrected); transcripts provided for audio-only content.
- Color contrast ratio ≥ 4.5:1 for normal text, ≥ 3:1 for large text (≥ 18 pt or ≥ 14 pt bold).
- Do not use color as the sole means of conveying information.
- Text must be resizable up to 200% without loss of content or functionality.

### Operable

- All functionality must be operable via keyboard alone; no keyboard traps.
- Visible focus indicator on all interactive elements (`:focus-visible` with ≥ 3px offset or equivalent).
- Skip-navigation link as the first focusable element on every page.
- No content that flashes more than 3 times per second.
- Page titles must be unique and descriptive (`<title>` tag per page).
- Link text must be self-descriptive; avoid "click here" / "read more" without context.

### Understandable

- `<html lang="en">` (or appropriate locale) on every page.
- Form inputs must have associated `<label>` elements or `aria-label`/`aria-labelledby`.
- Error messages must identify the field and describe the fix (not just "invalid input").
- Consistent navigation and labeling across the site.

### Robust

- Valid semantic HTML5 — use `<nav>`, `<main>`, `<header>`, `<footer>`, `<section>`, `<article>` landmarks.
- ARIA roles and attributes only where native HTML semantics are insufficient; avoid redundant ARIA.
- Interactive widgets (modals, accordions, tabs) must implement [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/) patterns.

### Testing & Validation

| Tool                                   | Usage                                                   |
| -------------------------------------- | ------------------------------------------------------- |
| axe DevTools / axe-core                | Automated scan — zero critical/serious issues at launch |
| NVDA + Firefox, VoiceOver + Safari     | Manual screen-reader walkthrough of top 10 pages        |
| WAVE or Lighthouse accessibility audit | CI gate: score ≥ 90                                     |
| Keyboard-only navigation review        | All forms and CTAs reachable without mouse              |

> **Acceptance criterion:** Zero WCAG 2.1 AA violations reported by axe-core in automated CI scan at time of launch.

---

## 2. SEO & AI-Search Optimization

### On-Page SEO

- Unique `<title>` (50–60 chars) and `<meta name="description">` (150–160 chars) per page; managed via Yoast SEO or RankMath.
- Single `<h1>` per page; logical heading hierarchy (`h1 → h2 → h3`).
- Clean permalink structure: `/programs/youth-literacy/` not `/?p=42`.
- Canonical tags on all pages; `rel="noindex"` on staging/admin URLs.
- XML sitemap auto-generated (`/sitemap.xml`) and submitted to Google Search Console and Bing Webmaster Tools.
- `robots.txt` blocks `/wp-admin/`, `/wp-includes/`, search result pages.

### Structured Data (schema.org / JSON-LD)

Implement via JSON-LD `<script>` blocks in `<head>`. Required schemas:

| Page type            | Schema type(s)                                                           |
| -------------------- | ------------------------------------------------------------------------ |
| Homepage             | `Organization`, `WebSite` (with `SearchAction` for sitelinks searchbox)  |
| Blog / news articles | `Article` (with `datePublished`, `dateModified`, `author`, `image`)      |
| Events               | `Event` (with `startDate`, `endDate`, `location`, `organizer`, `offers`) |
| Staff / board bios   | `Person` (with `jobTitle`, `worksFor`)                                   |
| FAQ pages            | `FAQPage` + `Question` + `Answer`                                        |
| Programs / services  | `Service` (with `provider`, `areaServed`, `description`)                 |
| Contact page         | `LocalBusiness` or `NGO` (with `address`, `telephone`, `openingHours`)   |

Validation: Google Rich Results Test + Schema Markup Validator at launch and after major content changes.

### AI-Search Optimization (LLM Discoverability)

- Provide a `/llms.txt` file at domain root ([llms.txt spec](https://llmstxt.org/)) listing key pages and purpose, enabling LLM crawlers to index the site efficiently.
- Publish an OpenGraph image (1200×630 px) per page for preview rendering in AI-generated summaries.
- Write meta descriptions as complete sentences answering "What does this page do / offer?" — these are frequently used verbatim in AI overviews.
- Structured data `description` fields ≥ 50 words to provide grounding context.
- Avoid JavaScript-only rendering for primary content; all text content must be present in initial HTML response (no client-side-only rendering).

### Technical SEO

- HTTPS enforced site-wide (see Security section).
- Core Web Vitals targets met (see Performance section) — CWV are a direct ranking signal.
- Mobile-first responsive design; passes Google Mobile-Friendly Test.
- `hreflang` tags if multilingual content is added later.
- Image `width` and `height` attributes set to prevent layout shift.

---

## 3. WordPress Theme & Plugin Requirements

### Theme

- Custom child theme or purpose-built theme built on a maintained parent (e.g., GeneratePress, Blocksy, or fully custom block theme).
- **Block editor (Gutenberg) native** — no Classic Editor dependency; use Full-Site Editing (FSE) or hybrid approach.
- No unused CSS/JS shipped from the theme; critical CSS inlined.
- Theme must pass `wp theme check` with zero errors.
- Responsive breakpoints: 375 px (mobile), 768 px (tablet), 1024 px (laptop), 1280 px+ (desktop).
- Dark-mode support via `prefers-color-scheme` media query (nice-to-have; required if in scope).

### Required Plugins

| Plugin                      | Purpose                                  | Notes                                                 |
| --------------------------- | ---------------------------------------- | ----------------------------------------------------- |
| Yoast SEO or RankMath       | SEO meta, sitemap, schema                | Pick one; configure org schema at network level       |
| WP Rocket or Perfmatters    | Caching, CSS/JS optimization             | See Performance section                               |
| Wordfence or Solid Security | Firewall, malware scan                   | See Security section                                  |
| WP Offload Media            | Media CDN (S3 + CloudFront or similar)   | Required if hosting on managed WP (Kinsta, WP Engine) |
| WPForms or Gravity Forms    | Accessible forms with conditional logic  | Must generate ARIA-compliant markup                   |
| UpdraftPlus or ManageWP     | Automated backups                        | Daily off-site backup; 30-day retention               |
| CRM integration plugin      | See CRM section                          | Salesforce / HubSpot / Mailchimp (TBD)                |
| Redirection                 | 301 redirect management during migration | See Migration section                                 |

### Plugin Governance

- Plugins must be actively maintained (last update ≤ 6 months, tested with current WordPress version).
- No nulled/pirated plugins.
- Plugin count target: ≤ 20 active plugins (performance and attack-surface impact).
- All plugins auto-update enabled for security releases; major version updates tested in staging first.

### WordPress Configuration

- WordPress core, themes, and plugins managed via **WP-CLI** scripts committed to the repo for reproducibility.
- `wp-config.php` uses environment variables (no credentials in source control).
- `DISALLOW_FILE_EDIT` and `DISALLOW_FILE_MODS` set to `true` in production.
- REST API endpoints not required publicly must be disabled or restricted.
- WordPress version and plugin list hidden from unauthenticated responses (`remove_action('wp_head', 'wp_generator')`).
- Staging environment mirrors production; deployments via CI/CD pipeline (GitHub Actions or DeployHQ).

---

## 4. CRM & Email Marketing Integration

### CRM Requirements

The nonprofit uses (or will adopt) one of the following — confirm with stakeholder before build:

| Candidate                                | Integration method                                                           | Notes                                          |
| ---------------------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------- |
| Salesforce Nonprofit Success Pack (NPSP) | Salesforce Web-to-Lead / Web-to-Case forms, or `salesforce-wordpress` plugin | Most feature-rich; highest implementation cost |
| HubSpot CRM (free tier)                  | HubSpot WordPress plugin (official)                                          | Simplest; native form/contact sync             |
| Bloomerang                               | Embedded forms / Zapier webhook                                              | Common in nonprofit sector                     |

**Minimum integration requirements regardless of CRM:**

- Donation/contact/volunteer forms on WordPress submit data directly to the CRM via authenticated API (no manual CSV export/import).
- Form submissions trigger a CRM contact create-or-update (deduplication by email address).
- Opt-in consent captured at form level; stored in CRM contact record as a custom field (`email_opt_in: true/false`, `consent_date`, `consent_source`).
- UTM parameters from the URL captured and stored on the CRM lead/contact (`utm_source`, `utm_medium`, `utm_campaign`).

### Email Marketing

- Platform: Mailchimp (assumed; confirm with stakeholder) or the CRM's built-in email.
- WordPress integration: Mailchimp for WooCommerce (if ecomm used) or MC4WP plugin for list subscribe forms.
- Double opt-in enabled for all email list sign-ups (CAN-SPAM / CASL compliance).
- Unsubscribe link present in every email; one-click unsubscribe propagated back to WordPress/CRM within 24 hours.
- Audience segments: donors, volunteers, program participants, newsletter-only.
- Transactional emails (form confirmations, donation receipts) sent via SMTP with SPF/DKIM configured (use SendGrid, Mailgun, or AWS SES — not `wp_mail()` default).

### Data & Privacy

- Privacy Policy page updated to reflect all third-party data processors (CRM, email platform, analytics).
- Cookie consent banner (GDPR/CCPA) deployed before any non-essential scripts load; use a consent management platform (e.g., CookieYes, Complianz) integrated with GTM consent mode v2.
- User data deletion workflow documented; admin must be able to delete/export a contact's data from WordPress within 30 days of request.

---

## 5. Security Hardening

### TLS / HTTPS

- TLS 1.2 minimum; TLS 1.3 preferred. Disable TLS 1.0 and 1.1.
- HSTS header: `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload` — submit to HSTS preload list.
- Valid certificate from a trusted CA (Let's Encrypt via Certbot, or hosting provider). Auto-renewal configured and monitored.
- HTTPS enforced via server-level redirect (`301`) and WordPress `siteurl`/`home` set to `https://`.

### HTTP Security Headers

All headers set at the web server / CDN layer (not WordPress PHP):

| Header                      | Required value                                                                                                                                                                                 |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Content-Security-Policy`   | Strict policy; `default-src 'self'`; whitelist third-party domains (CRM, analytics, fonts) explicitly. Start with `Content-Security-Policy-Report-Only` to detect violations before enforcing. |
| `X-Content-Type-Options`    | `nosniff`                                                                                                                                                                                      |
| `X-Frame-Options`           | `SAMEORIGIN` (or CSP `frame-ancestors 'self'`)                                                                                                                                                 |
| `Referrer-Policy`           | `strict-origin-when-cross-origin`                                                                                                                                                              |
| `Permissions-Policy`        | Disable unneeded APIs: `camera=(), microphone=(), geolocation=()`                                                                                                                              |
| `Strict-Transport-Security` | See HSTS above                                                                                                                                                                                 |

Validate with [securityheaders.com](https://securityheaders.com) — target grade **A**.

### Web Application Firewall (WAF)

- **Option A (hosting-level):** Cloudflare WAF (free plan covers OWASP Top 10 rules) in front of the origin.
- **Option B (plugin-level):** Wordfence premium WAF with real-time threat intelligence feed.
- At minimum: rate limiting on `/wp-login.php` and `/xmlrpc.php`; block `xmlrpc.php` entirely if not needed.
- Geo-blocking optional; configure only if traffic analysis warrants it.

### WordPress-Specific Hardening

- Admin URL changed from `/wp-admin/` to a custom path (obscurity measure; not a replacement for strong auth).
- Two-factor authentication (2FA) enforced for all admin and editor accounts (Wordfence 2FA or WP 2FA plugin).
- Strong password policy enforced at account creation.
- Login attempt limiting: lock after 5 failed attempts within 5 minutes.
- `author` query string enumeration disabled (`/?author=1` → 404).
- File permissions: `wp-config.php` = 600; directories = 755; files = 644.
- Disable directory listing on web server (`Options -Indexes` or equivalent).
- Remove `readme.html` and `license.txt` from public web root post-install.
- Database table prefix changed from `wp_` to a random prefix at install time.

### Backups & Recovery

- Automated daily encrypted backups stored off-site (e.g., S3, Google Drive, remote FTP).
- Backup includes: database + `wp-content/` directory.
- Restore tested quarterly; RTO target < 4 hours.
- 30-day backup retention minimum.

---

## 6. Performance Targets (Core Web Vitals)

### Targets

| Metric                          | Target (Good)                 | Measurement                          |
| ------------------------------- | ----------------------------- | ------------------------------------ |
| Largest Contentful Paint (LCP)  | ≤ 2.5 s                       | Field data (CrUX) + Lab (Lighthouse) |
| Cumulative Layout Shift (CLS)   | ≤ 0.10                        | Field data (CrUX) + Lab (Lighthouse) |
| Interaction to Next Paint (INP) | ≤ 200 ms                      | Field data (CrUX)                    |
| First Contentful Paint (FCP)    | ≤ 1.8 s                       | Lab                                  |
| Time to First Byte (TTFB)       | ≤ 800 ms                      | Lab                                  |
| Lighthouse Performance score    | ≥ 90 (mobile), ≥ 95 (desktop) | Lab                                  |

All targets measured on a simulated mid-range Android mobile device (Lighthouse default throttling) and validated with real-user CrUX data 30 days post-launch.

### Implementation Checklist

#### Hosting

- Managed WordPress hosting with PHP 8.2+, HTTP/2 or HTTP/3, and server-side caching (Nginx FastCGI cache or LiteSpeed).
- Recommended hosts for nonprofit budget: Kinsta (nonprofit discount available), WP Engine, or SiteGround.
- CDN for static assets (Cloudflare or host-provided CDN).

#### Caching

- Page caching enabled (WP Rocket, W3 Total Cache, or LiteSpeed Cache).
- Object caching with Redis or Memcached for database query results.
- Browser cache headers: `Cache-Control: public, max-age=31536000, immutable` for versioned static assets.

#### Images

- All images served in WebP format (with JPEG/PNG fallback via `<picture>` or server-side conversion).
- Images lazy-loaded (`loading="lazy"`) except above-the-fold hero images.
- Hero/LCP image preloaded: `<link rel="preload" as="image" href="...">`.
- `srcset` and `sizes` attributes on all `<img>` tags for responsive images.
- Maximum image width 1600 px; hero images ≤ 200 KB after WebP compression.

#### Fonts

- Self-host web fonts (avoid Google Fonts CDN for privacy + performance).
- `font-display: swap` on all `@font-face` declarations.
- Preload critical font files: `<link rel="preload" as="font" crossorigin>`.
- Limit to ≤ 2 font families, ≤ 4 font weights site-wide.

#### JavaScript and CSS

- Minify and concatenate CSS/JS (WP Rocket or build pipeline).
- Defer or async non-critical JS; critical CSS inlined.
- Remove unused CSS (PurgeCSS or WP Rocket's "Remove Unused CSS" feature).
- Third-party scripts (analytics, CRM widgets) loaded async and deferred until after `DOMContentLoaded` where possible.
- Google Tag Manager: fire non-essential tags only after user consent (GTM consent mode v2).

#### Layout Stability (CLS)

- Explicit `width` and `height` on all `<img>` and `<video>` elements.
- Reserve space for ads/embeds with `min-height` CSS.
- Avoid inserting DOM nodes above existing content after page load.
- Font fallbacks matched to web font metrics using `size-adjust` descriptor to prevent FOUT-related shifts.

---

## 7. Content Migration Plan

### Inventory & Audit

1. **Crawl existing site** with Screaming Frog (or similar) to export: URL list, page titles, meta descriptions, H1s, word counts, inbound links, response codes.
2. **Content audit spreadsheet** columns: URL | Page type | Word count | Last updated | Keep / Rewrite / Archive / Delete | New URL | Redirects needed | Assets (images/PDFs).
3. **Asset inventory:** enumerate all media files, PDFs, and documents. Flag files > 5 MB for optimization.
4. Target: reduce total page count by ≥ 20% by merging thin/duplicate content before migration.

### URL Strategy

- Preserve high-value URLs where possible (check Google Search Console for pages with inbound links or organic traffic > 100 sessions/month).
- Where URLs must change, implement `301` redirects (not `302`).
- All redirect mappings documented in a CSV (`old_url, new_url, status_code`) committed to the repo.
- Redirect chains must not exceed 2 hops.
- Confirm zero broken internal links post-migration using Screaming Frog re-crawl.

### Migration Phases

| Phase               | Tasks                                                                                                                        | Owner         | Timeline            |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ------------- | ------------------- |
| 0 — Prep            | Content audit, URL mapping, redirect CSV, staging environment setup                                                          | Dev + Content | Week 1–2            |
| 1 — Scaffold        | Deploy new theme, configure plugins, set up CRM/email integrations on staging                                                | Dev           | Week 3–4            |
| 2 — Content import  | Migrate pages/posts via WordPress export/import or WP CLI; apply block editor formatting                                     | Content + Dev | Week 5–7            |
| 3 — Asset migration | Upload and re-link media; optimize images to WebP; migrate PDFs                                                              | Dev           | Week 6–7 (parallel) |
| 4 — QA              | Accessibility audit, performance audit, SEO audit, form/CRM integration testing, cross-browser/device testing                | Dev + QA      | Week 8              |
| 5 — Cutover         | DNS switch, activate redirects, submit new sitemap to Search Console, disable old site (keep server for 90 days as fallback) | Dev           | Week 9              |
| 6 — Post-launch     | Monitor CrUX, Search Console, 404 reports, CRM sync; 30-day post-launch review                                               | Dev + Content | Week 10–12          |

### SEO Preservation Checklist (Pre-Cutover)

- [ ] All redirects tested on staging
- [ ] XML sitemap generated and validated
- [ ] Google Search Console property created for new domain/protocol; ownership verified
- [ ] Google Analytics 4 (GA4) property configured and tracking verified
- [ ] Structured data tested with Rich Results Test
- [ ] Open Graph images verified in Facebook Sharing Debugger
- [ ] Core Web Vitals baseline recorded in PageSpeed Insights

### Tools

| Tool                               | Purpose                       |
| ---------------------------------- | ----------------------------- |
| Screaming Frog SEO Spider          | Pre- and post-migration crawl |
| WP-CLI (`wp export` / `wp import`) | Bulk content migration        |
| Redirection plugin                 | Runtime redirect management   |
| Google Search Console              | Index coverage monitoring     |
| Broken Link Checker (WP plugin)    | Post-migration link audit     |

---

## Appendix: Definition of Done

A feature or page is considered **Done** when:

- [ ] Passes axe-core automated accessibility scan (zero critical/serious issues)
- [ ] Lighthouse Performance ≥ 90 (mobile)
- [ ] Structured data validated (no errors in Rich Results Test)
- [ ] Renders correctly on Chrome, Firefox, Safari (latest) and iOS Safari / Chrome Android
- [ ] All forms submit to CRM and trigger confirmation email
- [ ] Security headers grade A on securityheaders.com
- [ ] Code committed, staging deployment successful, reviewed by one other team member

---

_Document version: 1.0 — April 2026. Review and update after stakeholder sign-off on CRM platform selection._
