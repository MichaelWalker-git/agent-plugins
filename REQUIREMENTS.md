# Website Redesign Requirements

> Developer-facing checklist translated from the RFP for a nonprofit serving ~25,000–30,000 annual users.
> Target stack: WordPress (self-hosted or managed), CRM/email marketing integrations, standard CDN/WAF.

---

## 1. WCAG 2.1 AA Accessibility

| ID  | Requirement                                                                                       | Acceptance Criteria                                      |
| --- | ------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| A01 | All images have descriptive `alt` text                                                            | axe-core / WAVE: zero missing-alt errors                 |
| A02 | Color contrast ratio ≥ 4.5:1 (normal text), ≥ 3:1 (large text / UI)                               | Automated scan passes; manual check for brand palette    |
| A03 | Keyboard navigation: all interactive elements reachable via `Tab`, operable via `Enter`/`Space`   | Manual smoke test + automated axe run                    |
| A04 | Visible focus indicators on all focusable elements                                                | No `:focus { outline: none }` without custom replacement |
| A05 | Form inputs have associated `<label>` or `aria-label`; errors identified by text, not color alone | axe-core: zero label errors                              |
| A06 | Logical heading hierarchy (H1 → H2 → H3); no skipped levels                                       | HTML inspector + axe                                     |
| A07 | Skip-to-main-content link as first focusable element                                              | Keyboard tab test                                        |
| A08 | All videos have captions; audio-only content has transcript                                       | Manual review                                            |
| A09 | ARIA landmarks: `<header>`, `<main>`, `<nav>`, `<footer>` used correctly                          | axe-core landmark check                                  |
| A10 | No content flashes more than 3 times/second                                                       | Manual review of animations/carousels                    |
| A11 | Responsive layout usable at 320 px width, 400% browser zoom                                       | Browser resize test                                      |
| A12 | PDFs linked from the site are tagged/accessible or replaced with HTML alternatives                | PDF accessibility checker                                |

**Tooling:** axe DevTools, WAVE, Lighthouse accessibility audit, manual keyboard test on Chrome/Firefox/Safari.

---

## 2. SEO and AI-Search Optimization

### 2.1 Technical SEO

- [ ] Canonical URLs on all pages; no duplicate-content paths
- [ ] `robots.txt` allows all important content; blocks `/wp-admin`, staging subdomains
- [ ] XML sitemap auto-generated (Yoast SEO or Rank Math) and submitted to Google Search Console
- [ ] Hreflang tags if multilingual content exists
- [ ] Breadcrumb navigation with matching structured data
- [ ] 301 redirects map preserved from legacy URLs (export old sitemap, create redirect matrix before launch)
- [ ] Core Web Vitals targets met (see §6) — Google uses CWV as ranking signal

### 2.2 Structured Data (schema.org)

Implement via JSON-LD `<script>` blocks in `<head>` or injected by SEO plugin.

```jsonc
// Required schemas
{
  "@context": "https://schema.org",
  "@type": "NonprofitOrganization", // extends Organization
  "name": "...",
  "url": "...",
  "logo": "...",
  "sameAs": ["https://twitter.com/...", "https://linkedin.com/..."],
  "contactPoint": {
    "@type": "ContactPoint",
    "contactType": "customer support",
    "availableLanguage": "English"
  }
}
```

| Schema Type                   | Where Applied         | Plugin / Method                     |
| ----------------------------- | --------------------- | ----------------------------------- |
| `NonprofitOrganization`       | Site-wide (homepage)  | Yoast SEO / custom `functions.php`  |
| `Event`                       | Events/programs pages | The Events Calendar schema output   |
| `FAQPage`                     | FAQ / help sections   | Rank Math FAQ block or custom block |
| `BreadcrumbList`              | All interior pages    | Yoast SEO (auto)                    |
| `Article`                     | Blog/news posts       | Yoast SEO (auto)                    |
| `WebSite` with `SearchAction` | Homepage              | Custom JSON-LD snippet              |

### 2.3 AI-Search / LLM Discoverability

- [ ] Add `/llms.txt` (plain-text site map for LLM crawlers) — list canonical URLs + one-line descriptions
- [ ] Keep meta descriptions ≤ 160 chars; write as complete, factual sentences (optimized for featured snippets and AI-generated answers)
- [ ] Provide an `<meta name="description">` that answers the most common user question for each page
- [ ] Ensure Open Graph tags (`og:title`, `og:description`, `og:image`) are present on all pages

---

## 3. WordPress Theme and Plugin Requirements

### 3.1 Theme

- Block-based (Full-Site Editing) theme or a classic theme with full Gutenberg block support
- Child theme required if modifying a parent/commercial theme
- No inline styles injected by theme that override brand CSS variables
- Theme must pass `Theme Check` plugin scan with no errors

### 3.2 Required Plugins

| Plugin                                  | Purpose                      | Notes                                  |
| --------------------------------------- | ---------------------------- | -------------------------------------- |
| Yoast SEO or Rank Math                  | SEO, sitemaps, schema        | Choose one; do not run both            |
| Wordfence or Solid Security             | Firewall, malware scan       | See §5                                 |
| WP Rocket or Perfmatters + cache plugin | Performance                  | See §6                                 |
| Gravity Forms or WPForms                | Accessible form builder      | Must output proper `<label>` markup    |
| The Events Calendar (if applicable)     | Event management             | Free tier or Pro                       |
| UpdraftPlus or similar                  | Automated backups            | Daily DB + weekly full-site to offsite |
| WP Mail SMTP                            | Reliable transactional email | Connect to SendGrid / SES              |
| Redirection                             | 301 redirect management      | Import legacy redirect matrix          |

### 3.3 Plugin Governance

- [ ] Pin plugin versions in deployment notes; update only after staging test
- [ ] Remove all inactive/deactivated plugins before launch
- [ ] Audit plugins quarterly; replace abandoned plugins (last update > 18 months)
- [ ] No nulled/pirated premium plugins

### 3.4 Custom Development

- All custom PHP in child theme or a site-specific plugin (not `functions.php` of parent theme)
- Follow WordPress Coding Standards (`phpcs` with `WordPress` ruleset)
- Sanitize/validate all user inputs; use nonces for form submissions
- Enqueue scripts/styles via `wp_enqueue_scripts`; no hardcoded `<script>` tags in templates

---

## 4. CRM and Email Marketing Integration

### 4.1 CRM Integration

| Platform                        | Integration Method                                         | Minimum Data Fields                                |
| ------------------------------- | ---------------------------------------------------------- | -------------------------------------------------- |
| Salesforce                      | Gravity Forms + Salesforce Web-to-Lead or native connector | First name, last name, email, interest area        |
| HubSpot                         | HubSpot for WordPress plugin or Gravity Forms Add-On       | Contact properties matching HubSpot property names |
| Bloomerang / Little Green Light | Zapier webhook from form submission                        | Constituent record fields per provider API docs    |

- [ ] Double opt-in or explicit consent checkbox on every lead-capture form (CAN-SPAM / CASL compliance)
- [ ] Map form field names to CRM field API names before development begins
- [ ] Test end-to-end: submit test record → verify it appears in CRM within 60 seconds
- [ ] Error handling: failed CRM push logs to site error log and sends admin email alert

### 4.2 Email Marketing Integration

| Platform         | Integration Method                                               |
| ---------------- | ---------------------------------------------------------------- |
| Mailchimp        | Mailchimp for WordPress plugin or Gravity Forms Mailchimp Add-On |
| Constant Contact | Constant Contact Forms plugin                                    |
| Klaviyo          | Gravity Forms webhook + Klaviyo API                              |

- [ ] Subscription forms render visible, labeled consent checkbox — pre-ticked boxes are prohibited
- [ ] Unsubscribe / list management handled entirely by the ESP (do not build custom)
- [ ] Welcome email triggered automatically on list join; test in staging
- [ ] Suppression lists synced: CRM opt-outs reflected in ESP within 24 hours (or real-time via webhook)

---

## 5. Security Hardening

### 5.1 Transport Security (SSL/TLS)

- [ ] TLS 1.2 minimum; TLS 1.3 preferred — disable SSL 3, TLS 1.0, TLS 1.1
- [ ] HSTS header: `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`
- [ ] Submit domain to HSTS preload list after confirming all subdomains serve HTTPS
- [ ] Auto-renew certificate (Let's Encrypt via hosting panel, or AWS ACM)
- [ ] Mixed-content scan before launch: no HTTP assets on HTTPS pages

### 5.2 HTTP Security Headers

Add via server config (Nginx/Apache), Cloudflare Transform Rules, or plugin:

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' *.googletagmanager.com; ...
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

- [ ] Validate headers at <https://securityheaders.com> — target grade A
- [ ] CSP in report-only mode first; tighten after 2-week observation period

### 5.3 Web Application Firewall (WAF)

- [ ] Enable Cloudflare Free WAF (OWASP ruleset) or Wordfence Web Application Firewall
- [ ] Block `wp-login.php` to all IPs except admin IPs; or implement two-factor auth (2FA) for all admin users
- [ ] Disable XML-RPC if not required by any plugin: add to `.htaccess` or Nginx config
- [ ] Rate-limit login endpoint: 5 failed attempts → 15-minute lockout
- [ ] File-change detection enabled; alert on unexpected modification of core/theme/plugin files

### 5.4 WordPress Hardening Checklist

- [ ] Remove WordPress version from `<head>` and RSS feeds
- [ ] Disable file editing via WP Admin (`DISALLOW_FILE_EDIT` in `wp-config.php`)
- [ ] Set `wp-config.php` permissions to `640` or `600`
- [ ] Database prefix changed from default `wp_`
- [ ] `debug.log` excluded from web root or web-inaccessible
- [ ] Automated daily backups tested for restore; backups stored off-server (S3, Dropbox, etc.)

---

## 6. Performance Targets (Core Web Vitals)

### 6.1 Targets

| Metric                          | Target (Good) | Measurement Tool               |
| ------------------------------- | ------------- | ------------------------------ |
| Largest Contentful Paint (LCP)  | ≤ 2.5 s       | Lighthouse, PageSpeed Insights |
| Interaction to Next Paint (INP) | ≤ 200 ms      | CrUX, Lighthouse               |
| Cumulative Layout Shift (CLS)   | ≤ 0.1         | Lighthouse                     |
| Time to First Byte (TTFB)       | ≤ 800 ms      | WebPageTest                    |
| Total Page Size (homepage)      | ≤ 1.5 MB      | WebPageTest                    |

Measure on a simulated mid-range mobile device (Moto G Power) via Lighthouse.

### 6.2 Implementation Checklist

#### Images

- [ ] Serve WebP (with JPEG/PNG fallback via `<picture>` or CDN auto-convert)
- [ ] Lazy-load all below-the-fold images (`loading="lazy"`)
- [ ] Explicit `width` and `height` attributes on `<img>` to prevent CLS
- [ ] Hero/LCP image preloaded: `<link rel="preload" as="image">`
- [ ] Max image width 1,440 px; srcset provided for 400/800/1200/1440 px

#### Fonts

- [ ] Self-host web fonts (avoid Google Fonts DNS round-trip)
- [ ] `font-display: swap` on all `@font-face` declarations
- [ ] Subset fonts to Latin character set minimum

#### Caching and Delivery

- [ ] Page cache enabled (WP Rocket, W3 Total Cache, or server-level FastCGI cache)
- [ ] Browser cache headers: static assets `Cache-Control: max-age=31536000, immutable`
- [ ] CDN enabled for static assets (Cloudflare, BunnyCDN, or hosting CDN)
- [ ] GZIP or Brotli compression enabled on server

#### JavaScript and CSS

- [ ] Minify and concatenate CSS/JS (WP Rocket, or manual build step)
- [ ] Defer non-critical JS; remove render-blocking scripts from `<head>`
- [ ] Remove unused CSS (PurgeCSS or WP Rocket unused CSS removal)
- [ ] Third-party scripts (chat, analytics, social embeds) loaded async or via façade

#### Hosting

- [ ] PHP 8.2+; OPcache enabled
- [ ] MySQL/MariaDB with query cache; object cache via Redis or Memcached if available
- [ ] Server region matching primary user geography

---

## 7. Content Migration Plan

### 7.1 Pre-Migration Audit

- [ ] Crawl current site with Screaming Frog; export all URLs, titles, meta descriptions, H1s, word counts
- [ ] Tag each URL: **keep**, **redirect**, **consolidate**, or **delete**
- [ ] Identify top-traffic pages via Google Analytics (protect these first)
- [ ] Note all embedded media (images, PDFs, videos) for re-upload or CDN migration

### 7.2 Content Inventory

| Content Type      | Volume Estimate | Migration Method                                      |
| ----------------- | --------------- | ----------------------------------------------------- |
| Pages (static)    | TBD             | Manual copy-edit into new blocks                      |
| Blog / News posts | TBD             | WP Importer (XML export/import) or WP All Import      |
| Events            | TBD             | The Events Calendar CSV import                        |
| PDFs / documents  | TBD             | Re-upload to new Media Library; update links          |
| Images            | TBD             | WP All Import or bulk upload + Media Library          |
| Forms             | TBD             | Rebuild in Gravity Forms / WPForms; re-map CRM fields |

### 7.3 Redirect Matrix

- Export legacy URL list from Screaming Frog
- Map each old URL to new URL in a CSV (`source`, `target`, `http_status`)
- Import into **Redirection** plugin before DNS cutover
- Validate redirects with a post-launch crawl; fix any chains longer than 1 hop

### 7.4 Migration Sequence

1. **Staging build complete** — all templates, plugins, integrations tested
2. **Content freeze** on old site (no new posts/pages during migration window)
3. **Migrate content** to staging; QA by content owner
4. **SEO pre-launch check:** canonical tags, sitemap, robots.txt, redirect matrix
5. **DNS cutover** (low-TTL prep: set TTL to 300 s, 24 h before cutover)
6. **Post-launch crawl** — Screaming Frog on new domain; fix broken links, missing redirects
7. **Search Console** — submit new sitemap; request re-indexing of top pages
8. **Monitor** — CWV, crawl errors, 404 rate for 30 days post-launch

### 7.5 Rollback Plan

- Keep old hosting active for 30 days post-cutover
- DNS rollback possible within TTL window
- Full database + files backup taken immediately before DNS cutover

---

## Appendix: Definition of Done

A feature or page is **done** when:

1. Passes automated axe-core accessibility scan (zero critical/serious errors)
2. Passes Lighthouse audit: Performance ≥ 90, Accessibility ≥ 95, Best Practices ≥ 95, SEO ≥ 95 (mobile)
3. Structured data validates in Google Rich Results Test
4. Security headers grade A on securityheaders.com
5. CRM/email form integration tested end-to-end in staging
6. Reviewed by content owner and signed off
