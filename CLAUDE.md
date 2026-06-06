# Orabo (JapaConnect) — Claude Code Guide

## Project Overview
Orabo is a platform offering verified pathways to study, work, and migrate legally to the USA, Canada, UK, and Europe. The marketing surface and free tools are African-first (Japa wedge, country-of-origin personalization across 16 African countries); the paid immigration tools serve a global professional audience.

**Tagline:** "Japa is a dream. We help you plan it, verify it, and live it." (kept site-wide — brand DNA, not positioning copy)

**Target users — free tools, homepage, blog, SEO landers:** African aspirants (Nigeria, Ghana, Kenya, South Africa, Ethiopia, Tanzania, Uganda, Cameroon, and other African countries).

**Target users — paid immigration tools (EB-1A, EB-2 NIW, O-1A, UKGT all tiers):** Global professional audience (researchers, engineers, doctors, entrepreneurs).

**Audience framing rule:** When adding copy to paid immigration tool pages, default to global-audience framing. When adding copy to the homepage, free tools, blog, or SEO landers, default to African-first framing. Backend AI prompts personalize output by country-of-origin via `ORIGIN_AUTHENTICATION_CONTEXT` and `ORIGIN_CURRENCY_MAP` — that is product functionality, not brand framing, and is unaffected by the audience split.

---

## Current State (summary)

Live product surface:

- **Free tools (African-first):** Readiness Quiz, Visa Eligibility Checker (14 visas, 11 questions, 3 free uses), Document Checklist, Cost of Living (33 cities), Take-Home Pay (35 roles × 18 countries), Processing Times (18 visas), Compare Destinations, Migration Worth It Calculator, University/Scholarship/Opportunity discovery pages.
- **Paid AI tools (global audience):**
  - CV Review + Cover Letter; SOP Writer (9 tracks).
  - EB-1A & EB-2 NIW suites — Tier 1 assessment / Tier 2 letter / Tier 3 package.
  - O-1A suite — assessment / letter / package.
  - UK Global Talent — Tech Nation ×3 + Peer Review ×3 (all LIVE).
  - Pathway Match Report ($19).
  - Petition Audit (reviewer tool) — O-1A, EB-1A, EB-2 NIW, UKGT ($59 each, single `ukgt_audit` SKU covers both UKGT routes).
  - RFE Response Copilot ($249) — four-persona Claude pipeline for EB-1A / EB-2 NIW / O-1A.
  - Consultation booking (Calendly).
- **Pro subscription:** $12/mo.
- **SEO:** extensionless URLs (Cloudflare 301s `.html`), `_redirects` 301 cutover, sitemap, blog, programmatic `/compare/*` pages, per-product SEO landings (`/pathway-match`, `/sop-writer`, `/cv-review`, etc.), author entity + E-E-A-T `/why-trust-orabo`.
- **Security:** payment-integrity gate on all AI routes (`verifyPayment`), webhook idempotency ledger, RLS on all user-data tables, prompt-injection defense (`ANTI_INJECTION` + `<user_data>` delimiters) on all AI routes.

**Email pipeline — active `[brevo-<context>]` contexts:** `checklist`, `report`, `cv`, `cv-attr`, `sop`, `sop-attr`, `eb1a`, `eb1a-attr`, `eb2`, `eb2-attr`, `eb1a-letter`, `eb1a-letter-attr`, `eb2-letter`, `eb2-letter-attr`, `eb1a-package`, `eb1a-package-attr`, `eb2-package`, `eb2-package-attr`, `o1a`, `o1a-attr`, `o1a-letter`, `o1a-letter-attr`, `o1a-package`, `o1a-package-attr`, `ukgt-tech`, `ukgt-tech-attr`, `ukgt-tech-statement`, `ukgt-tech-statement-attr`, `ukgt-tech-package`, `ukgt-tech-package-attr`, `ukgt-research`, `ukgt-research-attr`, `ukgt-research-statement`, `ukgt-research-statement-attr`, `ukgt-research-package`, `ukgt-research-package-attr`, `preview`, `worthit`, `consultation`, `waitlist-welcome`, `rejection`, `story-approved`, `story-rejected`, `payout`, `opp-ack`, `publish`, `o1a-audit`, `o1a-audit-attr`, `eb1a-audit`, `eb1a-audit-attr`, `eb2-audit`, `eb2-audit-attr`, `ukgt-audit`, `ukgt-audit-attr`, `pathway-match`, `pathway-match-attr`, `rfe`, `rfe-attr`, `admin-waitlist`, `admin-opportunity`, `admin-booking`.

**Deployment:**
- **Frontend:** static site — source in private GitHub repo, deployed to Cloudflare Pages at `https://orabo.app`. Cloudflare builds from `main` on push; `_headers` in repo root sets server-level HTTP headers (including CSP) at the edge.
- **Backend:** Node/Express on Railway — health check `GET /health` returns `{"status":"ok"}`.
- **Database:** Supabase, two projects (Orabo Prod = users/payments/AI cache; Orabo Main = content/CMS). See Integrations.

**Rule E (permanent):** all hand-authored HTML edits via `sed`/Node.js script only — never the file editor on emoji-containing files; `file -i charset=utf-8` mandatory pre-commit check on every modified HTML file.

---

## Architecture — Multi-Page App

| File | Purpose | Auth |
|---|---|---|
| `index.html` | Main marketing site — free tools + marketing cards for paid services | No |
| `signup.html` | Sign-up page | No |
| `login.html` | Login + forgot-password page | No |
| `reset-password.html` | Password-reset — handles Supabase recovery token (PKCE `?code=` and implicit `#access_token`/`PASSWORD_RECOVERY`); all logic in external `js/reset-password.js`; Supabase UMD + `supabaseClient` | No |
| `dashboard.html` | Personal dashboard — sidebar tool launcher (15 entries: 1 journey home + 8 free + 6 Pro) + right-pane iframe tool content. Pro upgrade entry point. CV/SOP/Consultation are NOT here — they live on standalone pages. | Yes — redirects to `login.html` |
| `cv-tool.html` | Standalone CV Review — public, auto-opens modal | No |
| `sop-tool.html` | Standalone SOP Writer — public, auto-opens modal | No |
| `consult-booking.html` | Standalone Consultation booking — public, auto-opens modal | No |
| `compare-cities.html` | Pro: compare cost of living across 4 cities | Pro localStorage gate |
| `all-countries-pay.html` | Pro: take-home pay across all 9 countries × 35 roles | Pro localStorage gate |
| `doc-checklist.html` | Dashboard iframe: document checklist wizard (free preview) | No (self-gates) |
| `readiness-report.html` | Dashboard iframe: readiness quiz | No (self-gates) |
| `full-doc-checklist.html` | Dashboard iframe: document checklist (Pro card) | No (self-gates) |
| `why-trust-orabo.html` | Public methodology/trust page; indexable; WebPage JSON-LD | No |
| `readiness-score.html` | SEO landing — drives to `/#readiness` | No |
| `visa-eligibility.html` | SEO landing + in-place eligibility wizard. Two `.open-eligibility-cta` call `window.openEligibilityModal()`. `#eligibilityOverlay` + `#upgradeModalOverlay` markup duplicated from `index.html`. | No |
| `document-checklist.html` | SEO landing — drives to `/#tools` | No |
| `eb-immigration.html` | SEO landing — EB-1A & EB-2 NIW. Hero: two Tier 1 CTA buttons. Footer waitlist (`source: eb_waitlist`) in `js/eb-immigration.js` | No |
| `eb1a-tool.html` / `eb2-tool.html` | Standalone EB Tier 1 assessments | No |
| `eb1a-letter-tool.html` / `eb2-letter-tool.html` | Standalone EB Tier 2 letters | No |
| `eb1a-package-tool.html` / `eb2-package-tool.html` | Standalone EB Tier 3 packages | No |
| `o1a-visa.html` | O-1A landing — 3-tier grid + audit card; waitlist `source: o1a_waitlist` in `js/o1a-visa-waitlist.js` | No |
| `o1a-tool.html` / `o1a-letter-tool.html` / `o1a-package-tool.html` | Standalone O-1A Tier 1/2/3; Section 0 back → `/o1a-visa.html` | No |
| `uk-global-talent.html` | UKGT landing — all 6 tiers LIVE; waitlist `source: ukgt_waitlist` in `js/uk-global-talent-waitlist.js` | No |
| `ukgt-tech-tool.html` / `-statement-tool.html` / `-package-tool.html` | UKGT Tech Nation T1/T2/T3; back → `/uk-global-talent.html` | No |
| `ukgt-research-tool.html` / `-statement-tool.html` / `-package-tool.html` | UKGT Peer Review T1/T2/T3; back → `/uk-global-talent.html` | No |
| `o1a-audit-tool.html` / `eb1a-audit-tool.html` / `eb2-audit-tool.html` | Petition Audit (reviewer; user uploads draft); 6 screens; $59; 3 MB raw file cap | No |
| `ukgt-audit-tool.html` | UKGT Endorsement Audit; `route` selector (tech_nation/peer_review); single SKU `ukgt_audit` $59; `noindex,nofollow` | No |
| `pathway-match-tool.html` | Pathway Match Report; 5 sections, 19 fields; $19; back on Section 0 → `/pathway-match`; `noindex,nofollow` | No |
| `petition-audit.html` | Petition Audit router/landing — 2×2 grid to 4 per-type tools; static links only | No |
| `rfe-response.html` | RFE Response landing; indexed; Product JSON-LD; canonical `/rfe-response` | No |
| `rfe-response-tool.html` | RFE Response Copilot tool; `noindex,nofollow`; two-file upload | No |
| `universities.html` / `scholarships.html` / `opportunities.html` | Discovery pages — full navbar + footer; self-init module | No |
| `cost-of-living.html` / `take-home-pay.html` / `visa-pathways.html` / `processing-times.html` | Discovery pages extracted from homepage; full navbar + footer; FAQPage JSON-LD | No |
| `pathway-compare.html` | Compare Destinations — free; navbar + footer; no auth, no backend | No |
| `migration-worth-it.html` | Migration Worth It — free, public, SEO-indexed; no auth/payment | No |
| `admin.html` | Admin panel — not linked publicly; direct URL only; loads `css/admin.css` + `js/admin.js` | Admin (Bearer JWT + `user_profiles.role='admin'`) |
| `blog/index.html` + 6 articles | Blog | No |

`index.html` is always logged-out — no auth in navbar. Standalone tool pages (`cv-tool.html`, `sop-tool.html`, `consult-booking.html`) have no navbar/footer and require no auth — they auto-open the tool modal via an external boot script. Name and email captured as the first form section. `compare-cities.html` / `all-countries-pay.html` gate via `localStorage.getItem('orabo_pro_status') === 'true'`; hide header with `?embed=true`. `doc-checklist.html`, `readiness-report.html`, `full-doc-checklist.html` are standalone embedded pages — import tool module + `autoopen-*.js`; Stripe redirects use `window.top.location.href`.

---

## Frontend Structure

```
japaconnect/
├── index.html               # Main marketing site (paid tool buttons navigate to cv-tool.html / sop-tool.html / consult-booking.html)
├── signup.html              # Sign-up (external js/signup.js)
├── login.html               # Login + forgot-password (external js/login.js)
├── reset-password.html      # Password-reset — loads Supabase UMD + js/reset-password.js (external, CSP-required)
├── dashboard.html           # Protected dashboard — sidebar launcher (15 entries) + right-pane iframe + Pro upgrade. All logic external js/dashboard-inline.js.
├── cv-tool.html             # Standalone CV tool — public; autoopen-cv.js; no auth
├── sop-tool.html            # Standalone SOP tool — public; autoopen-sop.js; no auth
├── consult-booking.html     # Standalone consultation — public; autoopen-consult.js; keeps Supabase UMD
├── compare-cities.html      # Pro: compare CoL across 4 cities (Pro localStorage gate)
├── all-countries-pay.html   # Pro: pay comparison all countries (module script + separate non-module Pro gate)
├── doc-checklist.html       # Dashboard iframe: checklist.js + autoopen-checklist.js
├── readiness-report.html    # Dashboard iframe: quiz.js + autoopen-quiz.js
├── full-doc-checklist.html  # Dashboard iframe: Pro doc checklist
├── styles.css               # Global styles (imports all css/ partials)
├── script.js                # Legacy interactivity (pre-module)
├── site.webmanifest         # PWA manifest
├── sw.js / js/register-sw.js # Service worker permanently removed — stub comment files; not loaded
├── icons/icon-192.png, icon-512.png  # PWA icons (navy/gold logo)
├── _headers                 # Cloudflare Pages server-level HTTP headers — authoritative site-wide CSP
├── _redirects               # 46 permanent 301 redirects (.html → extensionless)
├── favicon.ico              # Multi-layer ICO (16×16 + 32×32)
├── sitemap.xml / robots.txt
├── css/
│   ├── base.css             # :root vars, reset, .container (max-width 1180px), .section, embed-mode rules, .nav-logo, .nav-brand-name
│   ├── hero.css             # Hero — grid collapse 1024px; phone mockup hidden 1024px; hero-trust-strip, hero-explore-link, hero-trust-link
│   ├── features.css         # features-grid (3→2→1); steps-row; trust-grid; Plan→Verify→Live band CSS
│   ├── visatracker.css      # vpt-layout grid 320px 1fr; breakpoint 900px
│   ├── auth.css             # Nav login/signup button styles
│   ├── subscription.css     # Upgrade modal + .nav-go-pro; eligibility upgrade modal z-index 10010
│   ├── quiz.css             # Quiz + report modal; .country-select-error/.visible; .payment-cancel-notice; .hidden utility
│   ├── cvtool.css           # CV + SOP tool styles (shared); .tool-back-btn, .tool-next-btn, .tool-review-* , .draft-resume-* , .tool-landing-card .btn-primary scoped rule
│   ├── hamburger.css        # Mobile hamburger — all rules inside @media (max-width:768px)
│   ├── eb-waitlist.css      # EB waitlist banner + blog teaser (index.html)
│   ├── pricing.css          # Rung ladder + attorney anchor + .elig-reveal block
│   ├── nav.css              # Tools dropdown, .nav-paid-badge, badge styles
│   ├── ibtool.css           # EB + O-1A + UKGT tool modal/screen visibility (.active); .ib-hint-icon/.ib-hint-tooltip; .ib-payment-* ; review screen IDs
│   ├── ibtool-audit.css     # Petition Audit modal/screen/upload CSS
│   ├── ibtool-rfe.css       # RFE Response modal (scoped #rfe-response-tool-modal)
│   ├── worthit.css          # Migration Worth It
│   ├── pathway-match.css    # Pathway Match screen visibility + multiselect
│   ├── pathway-compare.css + pathway-compare-overview.css  # Compare Destinations
│   ├── sources.css          # Source citation modal + tags
│   ├── discovery-pages.css  # .dp-page-hero, .dp-cta-block, .opp-optional-label
│   ├── author.css           # Byline component + author page
│   ├── admin.css            # Admin panel — loaded by admin.html only, NOT in styles.css
│   ├── dashboard-page.css / dashboard-preferences.css / dashboard-journey.css  # Dashboard — linked from dashboard.html, NOT in styles.css
│   └── mobile.css           # Mobile responsive fixes — loaded last in styles.css; owns html,body{overflow-x:hidden}
├── js/
│   ├── auth.js              # Supabase session, profile cache, openAuthModal; defensive window.supabase check — safe to import without Supabase UMD; exports readSessionFromStorage()
│   ├── reset-password.js    # Reset-password page logic (external — inline scripts CSP-blocked). Uses supabaseClient.
│   ├── embed.js             # Sets html.embed-mode synchronously
│   ├── stripe.js            # Checkout + payment return; purchaseItem resolves email bookingMeta.customer_email → booking_email → user.email; checkPaymentReturn IIFE preserves URL for payment_cancelled=1; bypass list 33 keys
│   ├── subscription.js      # isPro() + upgrade modal
│   ├── checklist.js         # Document checklist wizard engine (do not modify for payment-return/bypass/origin)
│   ├── quiz.js              # Readiness quiz engine + AI report (do not modify for payment-return)
│   ├── booking.js           # Consultation booking modal + Stripe; IIFE with !modal guard
│   ├── visatracker.js / visatracker-data.js  # Processing times — 18 visas
│   ├── costliving.js / costliving-data.js     # Cost of living — 33 cities; data file holds FX_RATES, FX_BASELINE, DATA_FRESHNESS
│   ├── paycalc.js / paycalc-data.js           # Take-home pay — ROLES, ROLE_GROUPS, SALARY_DATA
│   ├── tool-review.js       # Shared — renderReview({screenId,prefix,sections,answers,onBack,onContinue})
│   ├── draft-store.js       # Shared — saveDraft/loadDraft/clearDraft/hasDraft/listAllDrafts; key prefix orabo_draft_<toolKey>; DRAFT_VERSION=2; 30-day TTL
│   ├── draft-banner.js      # Shared — showResumeBanner({container,toolLabel,savedAt,onResume,onStartFresh})
│   ├── preview-screen.js    # Shared — showPreviewScreen(opts); 5 states A–E
│   ├── cvtool.js / cvtool-questions.js / cv-file-extract.js  # CV Review; CV_SECTIONS (6); window.cvTool={init,start}
│   ├── soptool.js / soptool-questions.js       # SOP Writer; SOP_SECTIONS (7); window.sopTool={init,start}
│   ├── redirect-paid-tools.js  # Non-module: wires CV/SOP/Consult Start buttons on index.html
│   ├── hero-cta.js            # Non-module IIFE: #hero-eligibility-cta one-click modal
│   ├── funnel-track.js        # ES module: first-party funnel analytics on index.html; window.oraboTrack exposed
│   ├── eligibility.js / eligibility-reveal.js  # Visa eligibility checker (14 visas, 11 Qs, 3 free uses) + reveal-at-value upsell; PATHWAY_REVEAL_MAP
│   ├── autoopen-*.js          # CSP-compliant external boot scripts (one per standalone tool page)
│   ├── payment-return-checklist.js / -quiz.js / -booking.js  # Non-module payment return handlers
│   ├── pro-bypass-checklist.js / -quiz.js  # Non-module Pro bypass; read JWT via readSessionFromStorage() (NOT getSession)
│   ├── checklist-origin-saver.js / quiz-origin-saver.js / free-quiz-origin.js  # Capture-phase country-of-origin validation
│   ├── quiz-results-saver.js  # Non-module: auto-save quiz scores for logged-in users; mirrors quiz.js scoring (DEPENDENCY block at top)
│   ├── quiz-retake-handler.js / quiz-email-capture.js  # quiz-email-capture.js is a DISABLED STUB
│   ├── eb1atool.js / eb1atool-questions.js / eb2tool.js / eb2tool-questions.js  # EB Tier 1
│   ├── eb1a-letter-tool.js / -questions.js / eb2-letter-tool.js / -questions.js  # EB Tier 2
│   ├── eb1a-package-tool.js / -questions.js / eb2-package-tool.js / -questions.js  # EB Tier 3
│   ├── o1atool.js + o1a-letter-tool.js + o1a-package-tool.js (+ *-questions.js)  # O-1A T1/T2/T3
│   ├── ukgt-tech-tool.js / -statement-tool.js / -package-tool.js (+ *-questions.js)  # UKGT Tech Nation
│   ├── ukgt-research-tool.js / -statement-tool.js / -package-tool.js (+ *-questions.js)  # UKGT Peer Review
│   ├── o1a-audit-tool.js / eb1a-audit-tool.js / eb2-audit-tool.js / ukgt-audit-tool.js  # Petition Audit reviewer tools
│   ├── pathway-match-tool.js / -questions.js  # Pathway Match Report
│   ├── rfe-response-tool.js / rfe-file-handler.js / rfe-response-questions.js  # RFE Response Copilot
│   ├── pathway-compare-data.js / -constants.js / -render.js / pathway-compare.js  # Compare Destinations (data layer DO NOT MODIFY)
│   ├── worthit.js            # Migration Worth It (imports ROLES from paycalc-data.js, API_URL from config.js)
│   ├── universities.js / scholarships.js / opportunities.js / opp-teaser.js  # Discovery content modules
│   ├── newsletter.js        # subscribeToDigest() + waitlistDigestHook()
│   ├── analytics.js         # GA4 loader; Measurement ID G-W5GY92ZHYS; <script defer> on every page except admin.html
│   ├── sources.js / sources-paycalc.js / sources-costliving.js / sources-visatracker.js / sources-modal.js  # Source citations (window.OraboSources)
│   ├── dashboard-journey.js / dashboard-inline.js  # Dashboard logic
│   ├── config.js            # API_URL export
│   └── utils.js             # escapeHTML and shared helpers
├── pathway-compare.html / readiness-score.html / visa-eligibility.html / document-checklist.html / eb-immigration.html
├── blog/  (index.html + 6 articles; load ../styles.css)
└── CLAUDE.md
```

**Module size rule:** every JS and CSS module must stay under 600 lines. `index.html` is exempt. Extract data/sub-sections into a sibling file when approaching the limit.

**Note:** `js/dashboard.js` still exists on disk but is no longer imported.

---

## Backend Structure (`japaconnect-backend`)

```
japaconnect-backend/
├── server.js            # Express app, route mounts; ALLOWED_ORIGINS hardcoded in cors() (https://orabo.app, https://www.orabo.app); app.set('trust proxy', 1)
├── scheduler.js         # node-cron jobs; hourly publish-notification cron
├── digest.js            # Weekly newsletter via Brevo — sendDigest({force,test}) idempotent, never throws
├── db/client.js         # pg Pool (Orabo Main)
├── routes/
│   ├── admin.js         # Admin CRUD — requireAdmin (Bearer JWT + user_profiles.role='admin'); pool (Orabo Main) for content + Supabase JS client (Orabo Prod) for users
│   ├── stripe.js        # Checkout, portal, webhooks → Orabo Prod; ITEM_PRICES, ITEM_NAMES, ITEM_SUCCESS_PAGES, TOOL_CANCEL_PAGES
│   ├── checklist.js     # AI document checklist (Claude + ai_cache) → Orabo Prod
│   ├── report.js        # AI readiness report (Claude + ai_cache); res/req.setTimeout(180000)
│   ├── cv.js / sop.js   # CV / SOP — AI gen + docgen.js → Orabo Prod
│   ├── docgen.js        # generatePdf() + generateDocx() — both handle HTML + markdown via isHtml(); LEAK_TOKENS guard
│   ├── eb1a.js / eb2.js / o1a.js (+ o1a-prompts.js)  # EB / O-1A AI routes
│   ├── ukgt-tech.js (+ ukgt-tech-prompts.js) / ukgt-research.js (+ ukgt-research-prompts.js)  # UKGT AI routes
│   ├── preview.js (+ preview-prompts-eb/o1a/ukgt.js)  # Free preview — previewLimiter 3/IP/24h
│   ├── migration-worth-it.js (+ -prompts.js) + utils/worthit-data.js  # Worth-it — worthitLimiter 5/IP/24h
│   ├── pathway-match.js (+ -prompts.js)  # Pathway Match — checklistLimiter
│   ├── audit.js (+ audit-prompts.js + audit-prompts-ukgt.js)  # Petition Audit — express.json({limit:'8mb'}); dispatch by family
│   ├── rfe.js (+ rfe-prompts.js + rfe-email-html.js)  # RFE Response — express.json({limit:'8mb'}); four-persona pipeline
│   ├── universities.js / scholarships.js / visas.js / opportunities.js / stories.js / visits.js / visa-tracker.js  # Content → Orabo Main
│   ├── dashboard.js     # Dashboard APIs; feature-flagged DASHBOARD_V2_ENABLED
│   ├── quiz.js          # POST /api/quiz/save-results → Orabo Prod; JWT-gated; rate-limited 10/min
│   └── internal.js      # POST /api/internal/run-digest — x-digest-secret header
├── utils/
│   ├── verify-payment.js     # Payment gate for all AI routes
│   ├── anti-injection.js     # ANTI_INJECTION constant (prompt-injection defense)
│   ├── document-ingest.js    # parseUploadedDocument (mammoth=DOCX, pdf-parse@^2.4.5=PDF, paste=text), sanitizeForPrompt, IngestError, MAX_WORDS=25000
│   ├── checklist-knowledge.js / checklist-pathway-data.js  # Checklist KB (see below)
│   ├── document-norms.js     # SOP_TRACK_SCAFFOLDS (9 tracks) + CV/SOP country rules
│   ├── brevo-email.js        # sendTransactionalEmail + sendAdminNotification (ADMIN_NOTIFY_EMAIL)
│   ├── constants.js          # ORIGIN_CURRENCY_MAP (16 African countries)
│   ├── dashboard-rules.js / tool-value-map.js  # Dashboard recommendation engine + value map
│   └── sources.js
└── .github/workflows/weekly-digest.yml  # GitHub Actions cron — Saturday 08:00 UTC + workflow_dispatch
```

---

## Feature Data Inventory

### Processing Times (`js/visatracker.js` + `js/visatracker-data.js`) — 18 visas
Study: F-1 (USA), Student (UK), German Student, Australia 500, NZ Student, Portugal D4. Work: UK Skilled Worker, UK Global Talent, O-1 (USA), EU Blue Card, Germany Job Seeker, EB-1A (USA), EB-2 NIW (USA). PR: Express Entry (Canada), Canada PNP, Australia 190, Australia 189, NZ Skilled Migrant.
Each visa: `id`, `label`, `country`, `flag`, `category`, `minWeeks`, `typicalWeeks?`, `maxWeeks`, `difficulty`, `difficultyColor`, `govUrl`, `note`, `milestones[]`. Trend in `TREND_DATA` keyed by visa id. Live data from `/api/visa-tracker` (falls back to static).

### Cost of Living (`js/costliving.js`) — 33 cities
Grouped `<select>`. `CITIES[]` + `COST_DATA{}` (monthly USD: rent, groceries, transport, utilities, dining). FX in `FX_RATES` (base USD). Comparison modal `compareSlots[]` (1–4). `buildCityOptions(selectedId)` grouped `<optgroup>`. Countries: USA (5), Canada (4), UK (4), Ireland (2), Netherlands (2), Germany (3), Portugal (2), Australia (3), NZ (2), Nigeria (2), Kenya (1), Ghana (1), South Africa (2).

### Take-Home Pay (`js/paycalc.js` + `js/paycalc-data.js`) — 35 roles × 18 countries
`ROLES[]`, `ROLE_GROUPS`, `SALARY_DATA{}`. 18 countries: Original (USA, Canada, UK, Germany, Netherlands, Ireland, Australia, NZ, Portugal), New (UAE, France, Sweden, Switzerland, Spain), African Origin (Nigeria, Kenya, Ghana, South Africa). Sectors: Technology (9), Healthcare (6), Education (3), Business & Finance (8), Engineering (4), Trades (5). FX_RATES in `paycalc.js` extended with AED (3.67), SEK (10.30), CHF (0.88). `buildCountryOptions()` with `<optgroup>`. `all-countries-pay.html` imports from `paycalc-data.js`; `COUNTRY_DATA` holds 9 countries.

### Currency list (all dropdowns)
USD, GBP, EUR, CAD, NGN, KES, GHS, ZAR, AUD, NZD (this exact order). FX (base USD): GBP 0.79, EUR 0.92, CAD 1.37, NGN 1580, KES 129, GHS 15.5, ZAR 18.5, AUD 1.55, NZD 1.63.

### CV Review + Cover Letter (`js/cvtool.js`)
Output types: `cv_review` ($18), `cv_cover_letter` ($28), `cover_letter_only` ($12). `ALL_SECTIONS = CV_SECTIONS` (6; section 0 captures `user_name` + `user_email`). Stripe returns to `cv-tool.html?payment_success=1&item=cv_review`. Response: `{ success, result, files: { cv_docx?, cl_docx? }, firstName }` — Word only. No auth.

### SOP Writer (`js/soptool.js`)
Output types: `sop_scratch` ($35), `sop_review` ($18), `sop_rewrite` ($30). `ALL_SECTIONS = SOP_SECTIONS` (7; section 0 captures name + email). Route `/api/sop`. Response: `{ success, result, files: { sop_docx }, firstName }`. No auth.

---

## CV/SOP Tool Architecture

Both tools use an identical pattern.

### HTML (standalone pages)
`cv-tool.html` / `sop-tool.html` — no navbar, no footer, no Supabase UMD. Wrapper `#cv-tool-modal` / `#sop-tool-modal` contains five screen divs: `#cv-form-screen`, `#cv-review-screen`, `#cv-payment-screen`, `#cv-loading-screen`, `#cv-download-screen` (and `sop-` prefix). Hidden by CSS in `css/cvtool.css`. External `autoopen-*.js` polls `window.cvTool` / `window.sopTool` (100ms × 50) then calls `init()`.

### Screen visibility
`showModal()` adds `.active` to modal → `#cv-tool-modal.active { display: block }`. `showScreen(id)` removes `.active` from all five then adds to target. Never use `style="display:..."`.

### Form rendering
`ALL_SECTIONS = CV_SECTIONS` (6) / `SOP_SECTIONS` (7). Question files already include "Your details" (user_name, user_email) as section 0 — do NOT prepend a separate DETAILS_SECTION. `renderSection(index)` re-renders the entire form screen on every navigation.

**Back button (`.tool-back-btn`):** above the section title on every section. Section 0: "← Back to Home" → `/`. Section 1+: "← Back" → `saveCurrentAnswers(index)` then `renderSection(index-1)`. No separate nav-area back button.

**Next button (`.tool-next-btn`):** accent blue pill from `css/cvtool.css`. Label "Next →" except last section reads "Review your answers →" (advances to the review screen). Do NOT use `btn-primary`.

Supported question types: `text`, `email`, `select`, `textarea`, `multiselect`, `conditionalSub`, `conditional`, `fileUpload` (.txt,.pdf,.docx → client-side extraction via `js/cv-file-extract.js`), `large`.

**CV file extraction (`js/cv-file-extract.js`):** ES module. `.txt` (FileReader), `.pdf` (pdfjs-dist@4.10.38 ESM, vendored `js/vendor/pdf.min.mjs`; worker `pdf.worker.min.mjs` via `workerSrc = new URL('./vendor/pdf.worker.min.mjs', import.meta.url).href`), `.docx` (mammoth@1.8.0 UMD, vendored `js/vendor/mammoth.browser.min.js`, loaded non-module before module scripts). Guards: 10 MB (`too_large`); MIN_TEXT_CHARS=20 (`empty`, scanned/image PDFs); `unsupported`; `parse_failed`. CSP: `worker-src 'self'` covers same-origin worker — no `blob:`, no `unsafe-eval`. Do NOT add `unsafe-eval` — pdf.js v4 no-eval fallback activates automatically under strict CSP.

### Payment return flow (sessionStorage bridge)
**Success:** review → "Continue to payment →" → payment → "Select & Pay" → `sessionStorage.setItem('orabo_cv_answers', ...)` + `orabo_cv_output_type` → `purchaseItem(pkg.type, btn, { customer_email: answers.user_email })`. Stripe → `cv-tool.html?payment_success=1&item=cv_review`. `init()` → `handlePaymentReturn()` checks `payment_success=1` AND `orabo_cv_answers` present → clears URL, removes keys, shows loading → POSTs `{ outputType, formAnswers }` to `/api/cv` (**field is `formAnswers`, not `answers`**) → `showDownloadScreen(data, userEmail)`.

**Cancel:** Stripe → `?payment_cancelled=1&item=cv_review`. `init()` wipes sessionStorage only when NEITHER `payment_success=1` nor `payment_cancelled=1` is present. `handlePaymentReturn()` detects cancel → restores answers, shows review screen.

No userId for anonymous flow. Download is Word only (DOCX) — `generatePdf()` produces blank output for HTML.

### API response shapes
- `/api/cv` → `{ success, result, files: { cv_docx?, cl_docx? }, firstName }`
- `/api/sop` → `{ success, result, files: { sop_docx }, firstName }`

---

## Document Checklist & Readiness Report Architecture

### Wizard pathway/destination selects
`#checklist-pathway-select` (Stage 1; 13 pathways; snake_case value); `#checklist-destination-select` (Stage 2; 9 standalone countries + `<optgroup label="Western Europe">` 19 countries); `#pathway-select-error` / `#destination-select-error` (`.visible` pattern). wizardNext1/wizardNext2 validation is inline in `checklist.js`.

**Pathway values + counts:** `work_visa` (15), `study` (13), `permanent_residency` (18), `us_green_card` (20), `job_seeker_visa` (11), `eu_blue_card` (14), `family_partner` (13), `critical_skills_visa` (14), `investor_business_visa` (16), `digital_nomad_visa` (11), `global_talent_visa` (16), `self_employment_visa` (12), `working_holiday_visa` (10). Full option text stored as `checklistPathway` (sent to backend); snake_case stored as `checklistPathwayKey`.

**`jc_checklist_state` shape:** `{ pathway: "<full label>", pathwayKey: "<snake_case>", country: "<country name>" }`. `oraboChecklistRestore(state)` sets `pathwaySel.value = state.pathwayKey`, `destSel.value = state.country`.

### Country-of-origin (mandatory)
Checklist: `id="checklist-country-origin"` (only on doc-checklist.html + full-doc-checklist.html). First option `<option value="" disabled selected>`. `jc_checklist_origin = sel.value`. Quiz: `id="report-country-origin"`, validated before `#quizNext`. `checklist-origin-saver.js` / `quiz-origin-saver.js` use capture-phase `addEventListener('click', fn, true)`; empty → `e.stopImmediatePropagation()` + `.country-select-error.visible`. Do NOT modify `checklist.js` / `quiz.js`.

### SessionStorage bridge
Before Stripe: `jc_checklist_state` / `jc_quiz_state` + origin keys. After return, `payment-return-*.js` reads/clears keys, passes `country_of_origin` in API body. Both call `fetch()` directly inside `DOMContentLoaded` — no `getSession()` await first. `userId` from `state.userId || null` (currently null). AbortController: 100s checklist, 180s report. Message rotation: 8s checklist, 15s report. Console prefixes `[checklist-return]` / `[quiz-return]`.

### API response format
`/api/checklist` and `/api/report` return `{ "success": true, "checklist/report": "<html>", "docx": "<base64>" }`.

### DOCX-only download
"Download Checklist (Word)" / "Download Report (Word)" — DOCX only. `payment-return-*.js` wires download in capture phase inside `.then()`, `e.stopImmediatePropagation()` to block html2pdf. Never add PDF.

### Checklist knowledge base (`utils/checklist-knowledge.js` + `utils/checklist-pathway-data.js`)
**`checklist-pathway-data.js`** — exports `PATHWAY_KNOWLEDGE_BASE` (13 entries keyed by snake_case): `{ display_name, what_it_is, eligibility_gate, core_documents[], destination_variants{}, common_pitfalls[], source_url }`. Destination variant keys use exact dropdown values (`'United States'`, `'United Kingdom'`). `working_holiday_visa` states no African passport qualifies. `critical_skills_visa` scoped to South Africa + Ireland. `investor_business_visa` lists US E-2 treaty African countries. `digital_nomad_visa` flags no-DNV destinations.

**`checklist-knowledge.js`** — exports `CHECKLIST_PROMPT_VERSION`, `INFEASIBLE_COMBINATIONS`, `buildKnowledgeBlock`, `buildInfeasibilityGuardrail`. **Current version: `v2.3-2026-05-21`** — bump whenever KB content changes to invalidate cache. `ORIGIN_AUTHENTICATION_CONTEXT` (16 entries): `hague_apostille_member`, `document_authentication`, `credential_evaluation_notes`, `passport_quirks[]`, `common_destinations_quick_notes{}`. Hague members: South Africa, Morocco, Senegal, Rwanda. `INFEASIBLE_COMBINATIONS` — 6 rules, each `{ detect(pathwayKey, dest, origin), explanation, closest_alternative }`. `buildKnowledgeBlock(pathwayKey, destination, origin)` builds Block 1.

**Routing in `routes/checklist.js`:** `PATHWAY_DISPLAY_TO_KEY` (13; 400 on mismatch). `VALID_DESTINATIONS` Set (28 full country names). Infeasible combinations skip cache read AND write. Cache key: `checklist:${CHECKLIST_PROMPT_VERSION}:${pathwayKey}:${country}:${countryOfOrigin}`.

### Brand sanitisation
`routes/checklist.js` + `routes/report.js` run `sanitizeBrand()` on every Claude response (cache hits + fresh): replaces "Japa Readiness"/"JapaConnect"/"japaconnect"/"japa connect" with "Orabo"/"Orabo Readiness". Sanitised content written to `ai_cache`. `checklist.js` strips markdown code fences before `sanitizeBrand()`: `content = content.replace(/^```html\s*/i, '').replace(/```\s*$/i, '')`.

### Currency personalisation
`country_of_origin` → backend resolves currency from `ORIGIN_CURRENCY_MAP` → injects non-cached Block 2. Block 1 cached with `cache_control: { type: 'ephemeral' }`. Infeasible details go into Block 2 only.

### Dynamic scholarship/university generation (report only)
`routes/report.js` does NOT query `scholarships`/`universities` tables — Claude generates 6–8 each from `VISA_KNOWLEDGE_BASE`. `max_tokens` 8000.

### Pro bypass
`js/pro-bypass-checklist.js` + `js/pro-bypass-quiz.js` intercept pay button in capture phase, check Supabase `pro_status`, call API directly if Pro. They run inside iframes — use `readSessionFromStorage()` (synchronous localStorage read), NOT `getSession()` (deadlocks on dual GoTrueClient). Abort + fall through to Stripe if token missing/expired. Do NOT add bypass logic to `checklist.js` / `quiz.js`.

### Free quiz and docgen
Free quiz on `index.html`: validation in `js/free-quiz-origin.js`. `generateDocx()` checks `isHtml(content)` — do NOT pre-process HTML to markdown.

---

## Auth Flow

1. Sign Up → `signup.html` → `supabase.auth.signUp()` → verification email
2. Email link → `login.html?verified=true`
3. Login → `dashboard.html`
4. `dashboard.html` calls `supabase.auth.getUser()` — redirects to `login.html` if no session
5. `openAuthModal()` in `auth.js` redirects to `login.html`

---

## Integrations

### Supabase — Dual-DB Architecture

#### 1. Orabo Prod (operational / transactional)
- **Host:** `rrkkbnoqubfmhrnxgngp.supabase.co` (us-west-2)
- **Env:** `SUPABASE_URL` + `SUPABASE_SERVICE_ROLE_KEY`
- **Client:** Supabase JS client instantiated inline in each route that needs it (`admin.js`, `checklist.js`, `cv.js`, `sop.js`, `eb1a.js`, `eb2.js`, `o1a.js`, `report.js`, `stripe.js`, `ukgt-tech.js`, `quiz.js`, `audit.js`, `verify-payment.js`, etc.)
- **Tables:** `user_profiles`, `purchases`, `one_off_purchases`, `ai_cache`, `processed_webhook_events`, `analytics_events`
- Auth lives here. Email verification redirect: `https://orabo.app/login.html?verified=true`. Password reset redirect: `https://orabo.app/reset-password` (no `.html` in the `redirectTo` string passed to `resetPasswordForEmail`).
- `user_profiles` columns: `id`, `pro_status`, `pro_expires_at`, `stripe_customer_id`, `stripe_subscription_id`, `quiz_results` (jsonb), `user_preferences` (jsonb), `journey_state` (jsonb), `role` (admin gate).
- `one_off_purchases` columns: `id`, `name`, `email`, `tool`, `stripe_session_id` (nullable), `created_at`.

#### 2. Orabo Main (content / CMS / analytics)
- **Host:** `fcogjoxcfmrcuvtreivm.supabase.co` (us-east-1)
- **Env:** `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` (`DB_HOST` = `db.fcogjoxcfmrcuvtreivm.supabase.co`)
- **Client:** raw `pg.Pool` in `db/client.js`, imported as `pool`
- **Tables:** `opportunities`, `scholarships`, `universities`, `visas`, `stories`, `visits`, `scrape_logs`, `post_earn_rewards`, `visa_processing_snapshots`
- Routes: `opportunities.js`, `scholarships.js`, `universities.js`, `visas.js`, `stories.js`, `visits.js`, `visa-tracker.js`. `admin.js` hits both DBs.
- `opportunities` extra columns: `status` (default 'pending'), `submitter_phone`, `rejection_reason`. `post_earn_rewards` columns: `id`, `email` (unique), `phone`, `name`, `approved_count` (default 0), `total_paid` (default 0), `reward_paid` (default false), `reward_paid_at`, `created_at`, `updated_at`.

#### Cross-cutting rules
- Never mix clients in a single route (except `admin.js`). Need both DBs → fetch each then combine in memory; no cross-DB JOIN.
- New tables go where their role belongs: user/auth/payment/AI-state → Orabo Prod; content/CMS/scraper/analytics → Orabo Main.
- Always confirm the project ref in the Supabase URL bar before any destructive SQL.

### Stripe
- Pro subscription price ID: `price_1TO1Z1Rpjtp1DRiMIRMkRvNT` — stored in `STRIPE_PRO_PRICE_ID` Railway env var; backend reads `process.env.STRIPE_PRO_PRICE_ID`, never from frontend body.
- One-time items: `readiness_report` ($15), `document_checklist` ($9), `verified_badge` ($5), `consultation_express` ($55), `consultation_full` ($90), `consultation_doc_review` ($35), `cv_review` ($18), `cv_cover_letter` ($28), `cover_letter_only` ($12), `sop_scratch` ($35), `sop_review` ($18), `sop_rewrite` ($30), `eb1a_assessment` ($39), `eb2_assessment` ($35), `eb1a_letter` ($150), `eb2_letter` ($140), `eb1a_package` ($250), `eb2_package` ($240), `o1a_assessment` ($39), `o1a_letter` ($150), `o1a_package` ($250), `ukgt_tech_assessment` ($39), `ukgt_tech_statement` ($150), `ukgt_tech_package` ($250), `ukgt_research_assessment` ($39), `ukgt_research_statement` ($150), `ukgt_research_package` ($250), `o1a_audit` ($59), `eb1a_audit` ($59), `eb2_audit` ($59), `ukgt_audit` ($59), `pathway_match_report` ($19), `rfe_response` ($249).
- **Subscription return:** `dashboard.html?payment=success` / `?payment=cancelled`.
- **One-time return (cv/sop):** `cv-tool.html?payment_success=1&item=...` / `sop-tool.html?...`.
- **Consultation:** `consult-booking.html?payment_success=1&item=consultation_*` → `stripe.js` shows `#calendlyOverlay`.
- **Other:** `index.html?payment_success=1&item=...` / `?payment=cancelled`.
- **Standalone page returns:** `readiness_report` → `readiness-report.html`; `document_checklist` → `doc-checklist.html`; `full_doc_checklist` → `full-doc-checklist.html`. Handled by `payment-return-*.js`.
- **Cancel return:** managed via `TOOL_CANCEL_PAGES` map in `routes/stripe.js`. Each tool item cancels to its own page with `?payment_cancelled=1&item=<key>`. Dashboard-originated items cancel to `dashboard.html?payment=cancelled`; all others to `/?payment=cancelled`. `js/stripe.js` bypass list and `TOOL_CANCEL_PAGES` must match the canonical Stripe item key set (report as an absolute count, currently 33 keys — grep `js/stripe.js` to verify rather than incrementing from memory).
- Success URLs append `&session_id={CHECKOUT_SESSION_ID}` (Stripe-substituted; client cannot forge). All tool modules + payment-return scripts read `session_id` from `URLSearchParams` and forward as `stripe_session_id` in the POST body.
- Webhook events: `checkout.session.completed`, `customer.subscription.deleted`, `invoice.payment_succeeded`, `invoice.payment_failed`, `charge.refunded`. Idempotent via `processed_webhook_events(event_id PRIMARY KEY, event_type, processed_at)`; duplicate-key (`23505`) returns 200. All internal errors inside the webhook return 200 (only signature verification returns 4xx).
- `pro_expires_at` from Stripe's actual `current_period_end` (`sub.current_period_end ?? sub.items?.data?.[0]?.current_period_end`) — never hardcode duration. `checkout.session.completed` uses `.update().eq('id', userId)`.
- Required Railway env: `STRIPE_SECRET_KEY`, `STRIPE_PRO_PRICE_ID`, `STRIPE_WEBHOOK_SECRET`, `AI_ROUTES_REQUIRE_STRIPE_SESSION`, `DASHBOARD_V2_ENABLED`.
- **Optional rate/reward env (safe code defaults via `parseInt(...) || default`):** `OPP_RATE_LIMIT_MAX` (5), `OPP_RATE_LIMIT_WINDOW_MIN` (10), `POST_EARN_PAYOUT` (6000), `POST_EARN_REWARD_PER_POST` (300), `POST_EARN_THRESHOLD` (20), `PREVIEW_RATE_LIMIT_MAX` (3), `PREVIEW_RATE_LIMIT_WINDOW_HOURS` (24), `WORTHIT_RATE_LIMIT_MAX` (5), `WORTHIT_RATE_LIMIT_WINDOW_HOURS` (24).

### Brevo
- Weekly digest: Saturday 08:00 UTC — triggered by GitHub Actions cron (`.github/workflows/weekly-digest.yml`) via `POST /api/internal/run-digest`. In-process `node-cron` retained as redundant fallback (idempotent).
- Subscribe: `POST /api/newsletter/subscribe` — `email` (required), `destination` (optional), `name`, `source`, `quiz_score`, `quiz_date`. Only adds to list when `destination` provided. `quiz_score` present → sets `QUIZ_SCORE`, `QUIZ_DATE`, `QUIZ_LEAD='true'`.
- Transactional: consultation confirmation (via `routes/stripe.js`); checklist/report/CV/SOP/EB/O-1A/UKGT/audit/pathway-match/rfe delivery fire-and-forget after generation. CV/SOP purchase attribute update sets `PURCHASE_DATE` + `LAST_TOOL`.
- Required Brevo custom attributes: `DESTINATION` (text), `QUIZ_SCORE` (number), `QUIZ_DATE` (text), `QUIZ_LEAD` (text), `PURCHASE_DATE` (date), `LAST_TOOL` (text), `LAST_PREVIEW_TOOL` (text), `LAST_PREVIEW_DATE` (date), `WORTHIT_TARGET` (text), `WORTHIT_VERDICT` (text), `PREF_PATHWAY` (text), `PREF_FIELD` (text).
- `sendDigest({ force, test })`: never throws to caller, idempotent (checks campaign history last 6 days). `test=true` creates draft `"Orabo Weekly Digest [TEST]"`. `force=true` bypasses idempotency.
- `POST /api/internal/run-digest` — `x-digest-secret` header (`DIGEST_TRIGGER_SECRET`); `?force=true` + `?test=true`; never referenced in frontend.
- Env: `BREVO_API_KEY`, `BREVO_LIST_ID`, `BREVO_SENDER_EMAIL`, `BREVO_SENDER_NAME`, `ADMIN_NOTIFY_EMAIL`, `DIGEST_TRIGGER_SECRET`.
- **Email template logo:** all Brevo templates reference `https://orabo.app/orabo-icon-256x256.png` (`width="80" height="80"` for Outlook). Never use `orabo-icon.svg` in email HTML. Inline styles are permitted in email HTML — the CSP rule applies to the web app only.

### Calendly
- Account: `calendly.com/oraboapp`. Express (30 min): `/oraboapp/30min`. Full (1 hr): `/oraboapp/full-consultation`. Doc Review (45 min): `/oraboapp/document-review`.
- Flow: pay → `consult-booking.html?payment_success=1&item=consultation_*` → `stripe.js` shows `#calendlyOverlay` (`_showCalendlyOverlay(url)`).
- `#bookingModal` is 2 steps only (About You → Review & Pay) — never re-add date/time picker.

### Email (EmailJS)
EmailJS fully removed — do NOT add any new EmailJS calls anywhere. Use Brevo for all email functionality (admin notifications via `[brevo-admin-waitlist]`, `[brevo-admin-opportunity]`, `[brevo-admin-booking]`).

### Claude API
- Model: `claude-sonnet-4-6`; all use prompt caching on the knowledge-base system block.
- `/api/checklist`: max_tokens 8000; two-block prompt (Block 1 cached ephemeral; Block 2 per-request currency + infeasibility); cache key includes `CHECKLIST_PROMPT_VERSION`; infeasible combos skip cache.
- `/api/report`: max_tokens 8000; two-block (KB cached + per-request currency); checks `ai_cache`.
- `/api/cv` + `/api/sop`: max_tokens 6000; two-block (cached gatekeeper persona + uncached country/track rules from `utils/document-norms.js`); NO `ai_cache`; no userId.
- `/api/preview`: `claude-haiku-4-5-20251001` (max_tokens 400); free — no verifyPayment, no ai_cache, no prompt caching.
- `/api/migration-worth-it`: `claude-haiku-4-5-20251001` (max_tokens 250, `res.setTimeout(30000)`).
- EB/O-1A/UKGT routes: max_tokens 8000 (Tier 1/2), 10000 (Tier 3); NO `ai_cache`.
- `/api/rfe`: four-persona pipeline (Analyst → Strategist → Drafter → Critic); Critic-revision loop capped at 2; Strategist max_tokens 8000.
- **When changing checklist immigration rules**, bump `CHECKLIST_PROMPT_VERSION` in `utils/checklist-knowledge.js` — changes all cache keys, forces fresh generations.

---

## Dashboard Architecture (`dashboard.html`)

| Function | Purpose |
|---|---|
| `initDashboard()` | Auth check → profile fetch → `updateDashboardForPlan()` → `handleReturnFromStripe()` |
| `updateDashboardForPlan(plan)` | Single source of truth for all plan-state visual changes. Call on load AND after payment. |
| `handleReturnFromStripe()` | Detects `?payment=success/cancelled`, waits 1.5s, re-fetches profile, calls `updateDashboardForPlan()`. No reload. |
| `initiateStripeCheckout()` | POSTs `{ userId, email, mode: 'subscription' }` to `/api/create-checkout-session` — no `priceId` |
| `showProModal()` / `hideProModal()` | Opens/closes `#pro-modal` |
| `showToast(message, type)` | `type`: `'success'`/`'error'`/`'info'` |
| `openPreferencesModal(prefill)` / `closePreferencesModal()` | `#prefs-modal`; toggles `body.preferences-modal-open` |
| `handlePreferencesSubmit(e)` | Validates, reads JWT via localStorage scan, POSTs to `/api/dashboard/preferences`, calls `loadJourneyState()` |
| `handleRetakeQuiz()` | Sets `sessionStorage.orabo_retaking_quiz='1'` then `window.location.href='/#readiness'` (top-frame nav preserves sessionStorage) |
| `wireJourneyCtas()` | Wires `data-cta` buttons + delegated listener on `#dash-journey-panel` for `data-action="open-prefs-edit"` / `"retake-quiz"` |
| `loadJourneyState()` | Alias for `window.loadJourneyState`; refresh widgets after preferences save |
| `renderRecommendedTools(...)` | Branches on `cta_url`: `sidebar:<tool-id>` → `.dash-tool-card[data-tool-id]` `.click()`; plain URL → `window.open(url,'_blank','noopener')`. Mapping in backend `utils/dashboard-rules.js`. Do NOT modify `dashboard-inline.js` for routing. |
| `refreshUsageStats(plan)` | Pro → fetch `/api/dashboard/usage-stats` + render Pro widgets; Free → upgrade CTA only |
| `renderValueThisMonthWidget` / `renderMonthlyRecapWidget` / `renderValueUpgradeCTAWidget` | Phase 6 widgets |

- **Phase 6 widget visibility:** Pro widgets carry `data-plan="pro"`, hidden by `[data-plan="pro"]:not(.unlocked){display:none}` in `dashboard-journey.css`; `refreshUsageStats('pro')` adds `.unlocked`. Free CTA widget hidden by `.dj-widget--hidden`.
- `dashboard-journey.js` **wraps** (not modifies) `window.updateDashboardForPlan` to fire `refreshUsageStats(plan)`.
- `GET /api/dashboard/usage-stats`: JWT-gated, `DASHBOARD_V2_ENABLED`; queries `one_off_purchases` by `email`; `utils/tool-value-map.js` is source of truth for `hours_saved`/`value_usd` (`value_usd` must stay in sync with ITEM_PRICES — stripe.js canonical).
- **Preferences modal:** `#prefs-modal` via `body.preferences-modal-open .prefs-modal { display: flex }` — never inline styles.
- `window._oraboLastJourneyState` set after each `/api/dashboard/state` fetch; used by `openPreferencesModal` prefill.
- **Iframe close:** listens for `'tool-closed'` postMessage → `closeTool()`. **Pro upgrade from iframes:** post `'initiate-pro-upgrade'` to `window.top`.
- **Pro tool cards:** `class="pro-tool"` + `data-plan="pro"`; tooltip `[data-plan="pro"]::after`; hidden when `.unlocked`.
- **`#pro-modal`:** hidden by default via `#pro-modal { display: none }`; shown by adding `.pro-modal-visible`. Does NOT auto-open — only by explicit user action. Dismissed via `#pro-modal-close`, `#pro-modal-dismiss`, backdrop, Escape (all in `js/dashboard-inline.js`).
- **Scroll-lock caveat:** `showProModal()`/`hideProModal()` set `document.body.style.overflow` directly — works while `style-src` retains `'unsafe-inline'`. Refactor to a `body.modal-open` class before tightening `style-src`. Scroll-lock is single-modal-scope — track an open-modal count before adding stacked modals.

---

## Coding Rules

- Always commit and push to `origin main` after every code change — no need to ask.
- **Pro price is $12/mo** — never advertise any other price for the Pro subscription on any surface.
- **Document Checklist and Readiness Report outputs are DOCX-only** — never advertise "PDF" in copy/buttons/emails/FAQ. `generatePdf()` produces blank output for HTML input; intentional, not changing.
- Pure HTML/CSS/JS for the frontend — no framework. No TypeScript. Keep global styles in `styles.css`; feature styles in `css/<feature>.css`. Feature JS for `index.html` in `js/<feature>.js` (ES modules).
- `signup.html`, `login.html`, `dashboard.html` use external non-module script files (`js/signup.js`, `js/login.js`, `js/dashboard-inline.js`) — Cloudflare edge CSP (`script-src` no `'unsafe-inline'`). Do not convert to ES modules and do not re-inline.
- **`reset-password.html` MUST keep ALL logic in `js/reset-password.js`** — inline scripts are CSP-blocked silently. Uses Supabase UMD — always `supabaseClient`, never `const supabase`.
- On `signup.html`, `login.html`, `dashboard.html` — always use `supabaseClient` (not `supabase`); `const supabase` conflicts with the UMD global and crashes silently.
- City/role selection UIs use grouped `<select>` dropdowns (not pill buttons); extend existing data arrays.
- **Cost of Living comparison:** uses `compareSlots[]` (not `compareCities`); `buildCityOptions(selectedId)`; slots 2–4 progressive; slot 0 permanent. Do NOT revert to pills.
- **paycalc-data.js pattern:** `ROLES[]`, `ROLE_GROUPS`, `SALARY_DATA{}` in `js/paycalc-data.js` only. Adding a role: add to all three exports with data for all 18 countries. Adding a country: `SALARY_DATA` key for all 35 roles + `COUNTRIES` entry (fxRate, taxRate, cityId) + `COL_MONTHLY_USD` + `COUNTRY_DATA` in all-countries-pay.html + correct `COUNTRY_GROUPS` array.
- **all-countries-pay.html:** tool logic in `<script type="module">`; Pro gate in separate non-module `<script>` before it. Module top-level `return` is a SyntaxError.
- Follow BEM-like CSS naming. Do not add comments unless logic is non-obvious. Use existing CSS variables — do not introduce new colour values.
- Every public-facing HTML page must include a complete Open Graph block (`og:title`, `og:description`, `og:url`, `og:image` + `og:image:secure_url`, `og:image:type`, `og:image:width=1200`, `og:image:height=630`, `og:image:alt`) referencing `https://orabo.app/og-image.jpg`, plus `twitter:card=summary_large_image` and `twitter:image`. Reference: `uk-global-talent.html`.
- Mobile-first responsive — check existing `@media` breakpoints before adding new ones.
- **New database table (enforced by Supabase from 2026-10-30):** provide `CREATE TABLE` SQL for the SQL editor BEFORE pushing dependent code, with explicit GRANT statements:
  ```sql
  CREATE TABLE public.new_table (...);
  GRANT SELECT, INSERT, UPDATE, DELETE ON public.new_table TO service_role;  -- REQUIRED
  GRANT SELECT ON public.new_table TO authenticated;  -- only if frontend reads via supabase-js with user JWT
  GRANT SELECT ON public.new_table TO anon;           -- only if publicly readable
  ALTER TABLE public.new_table ENABLE ROW LEVEL SECURITY;
  ```
  Grant minimally. Do NOT grant authenticated/anon to OPS tables. Applies to the OPS DB (rrkkbnoqubfmhrnxgngp), not the CONTENT DB pg.Pool writes.
- **RLS is enabled on all user-data tables.** Anon: read-only `opportunities WHERE approved=true` + `stories WHERE approved=true`. User JWT: read/write own `user_profiles` row + read-only own `purchases`. All other writes go through the Express backend with the service role key. Never add a new user-data table without RLS policies in the same migration. Canonical reference: `migrations/2026-05-13-rls-pass-2.sql`.
- **Stripe subscription checkout:** never pass `priceId` from frontend — backend reads `process.env.STRIPE_PRO_PRICE_ID`; frontend sends `{ userId, email, mode: 'subscription' }`.
- **Dashboard plan state:** always route changes through `updateDashboardForPlan(plan)`.
- **`DASHBOARD_V2_ENABLED`** gates all `/api/dashboard/*` routes. When false/unset, every dashboard read route returns 404.
- **CSP is enforced via `_headers` (Cloudflare Pages) — not meta tags.** `_headers` in the repo root is the single authoritative CSP. All per-page `<meta http-equiv="Content-Security-Policy">` tags removed. When adding a new external domain dependency, update `_headers` only. Never re-add meta CSP tags.
- **CSS specificity hide rule:** prefer `#elementId { display: none }` default + `#elementId.visible-class { display: ... }` override (ID specificity on both sides). Avoid `.hidden-class { display: none }` against `#elementId { display: flex }`.
- **CSP inline styles:** `style-src` has `'unsafe-inline'` (required for Chart.js). Never use `style="display:none"` on elements that start hidden via JS; use CSS classes.
- **CSP inline scripts:** `script-src` has no `'unsafe-inline'` — never use `onclick=""`; always wire via `addEventListener` in module init IIFEs.
- **`frame-src` MUST include `'self'`.** Dashboard embeds same-origin iframes. Without `'self'`, loads are blocked and Chrome renders `chrome-error://chromewebdata/`. If a third-party iframe source is added, append its origin — do not replace `'self'`.
- **Standalone tool page boot scripts MUST be external files (CSP).** Every standalone tool page that auto-opens a modal loads `js/autoopen-<tool>.js` — never inline, never `type="module"` unless `import` is needed. Canonical references: `js/autoopen-cv.js` (DOMContentLoaded), `js/autoopen-eb1a.js` (bare polling), `js/autoopen-consult.js` (complex side-effect). Reference as `<script src="js/autoopen-<tool>.js"></script>` at end of `<body>` after module imports. Production CSP blocks inline `<script>` silently; `python3 -m http.server` does NOT apply `_headers`, so inline blocks pass local smoke and fail in production.
- **CSP changes fail silently.** Symptoms: inline handlers do nothing, iframes render `chrome-error://chromewebdata/`, inline scripts never run, inline styles ignored. Local server does NOT apply `_headers`. Every commit touching `_headers` or any CSP directive MUST be followed by an incognito production test on https://orabo.app with DevTools Console open after the Cloudflare deploy (~60–90s) — click through every affected surface and confirm zero `Refused to...` / CSP-violation lines.
- **New tool modal pattern:** CSS `.active` class for visibility (never `style="display:..."`); `showModal()` adds `.active` to wrapper; `showScreen(id)` toggles `.active` on child screens; sessionStorage bridge; two-part URL condition (`payment_success=1` + sessionStorage key) in `init()`.
- **Dashboard iframe pages:** standalone HTML + `autoopen-<tool>.js` + `url:` entry in dashboard TOOLS array. Never use `anchor:` for auto-opening tools.
- **Iframe close button:** include `payment-return-*.js` wiring close to `handleToolClose()` — posts `'tool-closed'` (iframe) or redirects to `dashboard.html` (standalone).
- **userId in AI generation requests:** AI routes derive identity from the `Authorization: Bearer <jwt>` header, never from `req.body.userId`. `utils/verify-payment.js` calls `supabase.auth.getUser(jwt)` to resolve `authUserId`. The anonymous Stripe-session-paid flow (CV, SOP, EB, O-1A) works without a JWT. Do NOT pass `userId` to `/api/cv` or `/api/sop`; recipient email uses `formAnswers.user_email`.
- **Pro bypass scripts read JWT from localStorage, NOT via getSession().** `js/pro-bypass-checklist.js` + `js/pro-bypass-quiz.js` run inside iframes; `getSession()` hangs (dual GoTrueClient). Use `readSessionFromStorage()` from `js/auth.js` (inlined verbatim in each IIFE); read session synchronously from localStorage (`sb-<project-ref>-auth-token`); abort + fall through to Stripe if missing/expired. Do NOT revert to `getSession()`.
- **Stripe redirects in iframes:** `purchaseItem` + `subscribeToPro` use `(window !== window.top ? window.top : window).location.href`.
- **Pro upgrade from embedded pages:** post `'initiate-pro-upgrade'` to `window.top` — never call Stripe from an iframe (applies to `compare-cities.html`, `all-countries-pay.html`, `wireUpgradeModal`, `eligibility.js`).
- **Upgrade button label:** always "Upgrade to Pro".
- **Backend CORS:** `server.js` hardcodes `ALLOWED_ORIGINS` (`https://orabo.app`, `https://www.orabo.app`) — no `FRONTEND_URL` env var.
- **Consultation booking:** `#bookingModal` + `#calendlyOverlay` in `consult-booking.html`. Modal 2 steps only. `_showCalendlyOverlay(url)` in `stripe.js`.
- **booking.js:** IIFE with `if (!modal) return`. Step 1 `.tool-back-btn` "← Back to Home"; step 2 `.tool-back-btn` "← Back" → `goToStep(1)`. Saves `jc_booking_state` to sessionStorage before `purchaseItem()`. `js/payment-return-booking.js` handles cancel return.
- **`.tool-back-btn` in cv/sop forms:** accent text-link (`color: var(--accent)`, no border, `font-size: 0.9rem`) in `css/cvtool.css`. Section 0: "← Back to Home" → `/`; Section 1+: "← Back" → save + `renderSection(index-1)`.
- **`.tool-next-btn`:** self-contained accent blue pill in `css/cvtool.css` (`background: var(--accent)`, `color:#fff`, `border-radius:8px`, `padding:0.75rem 2rem`, `font-weight:600`). Do NOT revert to `btn-primary`.
- **`.tool-landing-card .btn-primary` scoped rule:** in `css/cvtool.css` — required for bare `btn-primary` on index.html CV/SOP cards. Do not remove.
- **one_off_purchases audit log:** `cv.js`, `sop.js`, `eb1a.js`, `eb2.js` (and others) fire-and-forget insert after every successful generation using `formAnswers.user_name` + `formAnswers.user_email`; `stripe_session_id` null for now.
- **CV no-fabrication rule:** CV system prompt (`routes/cv.js`) forbids inventing metrics/teams/revenue/percentages/awards/certs/dates. Inserts `[INSERT METRIC — e.g. team size, $ figure, % improvement]` and flags each in Top 5 Priority Actions. SOP retains Rule 1 (no invented experiences). Both must survive any prompt edit.
- **SOP track branching:** `routes/sop.js` reads `formAnswers.sopTrack`, normalises via `normalizeTrack()`, selects persona + scaffold from `SOP_TRACK_SCAFFOLDS` in `utils/document-norms.js`. Chevening produces four labelled essays (Leadership and Influence, Networking, Studying in the UK, Career Plan). MCF structures around the give-back arc. Unrecognised → `masters_admissions`. `destinationCountry` dropdown: USA, Canada, United Kingdom, Germany, Netherlands, Ireland, Australia, New Zealand, Portugal, Other. Backend lookup: `CV_COUNTRY_RULES[...] || .DEFAULT` and `SOP_COUNTRY_RULES[...] || .DEFAULT`.
- **EB routes (`eb1a.js`, `eb2.js`):** NO ai_cache reads (all tiers fresh). max_tokens 8000 (T1/T2), 10000 (T3). No userId. Branches on `outputType`: `*_assessment` → T1 (plain text); `*_letter` → T2 (HTML, code-fence strip); `*_package` → T3 (HTML, code-fence strip, `res.setTimeout(180000)`). Console prefixes `[eb1a]`, `[eb2]`, `[brevo-eb1a-letter]`, etc.
- **EB frontend (`eb1atool.js`, `eb2tool.js`):** cvtool pattern. sessionStorage `orabo_eb1a_answers`/`orabo_eb1a_output_type` (+ eb2). `init()` preserves sessionStorage on `payment_success=1` or `payment_cancelled=1`; wipes otherwise. Tooltip `.ib-hint-icon` + `.ib-hint-tooltip` (CSS hover, never onclick). Element ID prefix `ib-q-` / `ib-err-`. Exports `window.eb1aTool` / `window.eb2Tool = {init, start}`. EB-1A keys include `evidence_inventory` + `letter_writers` (Section E, optional); EB-2 `evidence_and_letters` (Section D, optional) — backend defaults blank, must not block. SCREENS includes review screen.
- **EB Tier 1/2/3 tools:** standalone page pattern. Poll `window.<tool>` (100ms × 50). sessionStorage keys per tier (`orabo_eb1a_letter_*`, `orabo_eb1a_package_*`, etc.). Download file keys `eb1a_letter_docx` / `eb1a_package_docx` / etc. Question id prefixes isolated per tier (`eb1a_letter_q<n>_<slug>`, `eb1a_package_q<n>_<slug>`). Mandatory email validation at section 0. No caching, no userId, DOCX-only. Cancel return restores answers + shows review screen.
- **EB-2 NIW Tier 2 R-01 Filing Risk Advisory:** `EB2_LETTER_SYSTEM_PROMPT` inserts `<h3>Filing Risk Advisory</h3>` between Prong 2 and Prong 3 when ALL THREE of `eb2_letter_q15_us_collaborations`, `eb2_letter_q12_publications_projects`, `eb2_letter_q14_expert_letters` indicate no evidence. Preserve this block; do NOT weaken/remove. Layer 2 guard `console.warn('[eb2-letter] prong2-advisory-missing', ...)` is log-only, runs on pre-footer `aiOutput` — preserve.
- **O-1A route (`routes/o1a.js`):** mounted `/api/o1a`. Branches: `o1a_assessment` → T1 (plain text, 8-criterion scorecard, non-immigrant framing); `o1a_letter` → T2 (HTML, Petition Support Letter + mandatory Peer Consultation Letter); `o1a_package` → T3 (HTML, 7 sections incl. Petitioner Support Brief). NO ai_cache. Petitioner-voice Rule 1 in T2/T3 bans first-person pronouns referring to the beneficiary. `[INSERT: <description>]` placeholders. max_tokens 8000 (T1/T2), 10000 (T3); `res.setTimeout(180000)`. File keys `o1a_assessment_docx` / `o1a_letter_docx` / `o1a_package_docx`. Console prefixes `[o1a]`, `[brevo-o1a]`, etc.
- **O-1A frontend tools:** EB pattern. Section 0 Back → `/o1a-visa.html`. sessionStorage `orabo_o1a_*` per tier. Question prefixes `o1a_q<n>_<slug>` / `o1a_letter_q<n>_<slug>` / `o1a_package_q<n>_<slug>`. Download `o1a-assessment.docx` / `o1a-petition-letter.docx` / `o1a-full-petition-package.docx`.
- **UKGT Tech Nation route (`routes/ukgt-tech.js`):** mounted `/api/ukgt-tech`. Branches: `ukgt_tech_assessment` → T1; `ukgt_tech_statement` → T2; `ukgt_tech_package` → T3. Prompts in `routes/ukgt-tech-prompts.js`. `sanitizeBrand()` inline. UK English mandatory. max_tokens 8000 (T1/T2), 10000 (T3); `res/req.setTimeout(180000)`. NO ai_cache. verifyPayment + gate before Claude. Word-count guardrail on T2/T3: `_countPersonalStatementWords()`; >1,200 → one retry; logs `[ukgt-tech-statement] word-count retry` / `[ukgt-tech-package] word-count retry`. Code-fence strip before `generateDocx()`. File keys `ukgt_tech_assessment_docx` / `ukgt_tech_statement_docx` / `ukgt_tech_package_docx`. Downloads `ukgt-tech-nation-eligibility-assessment.docx` / `ukgt-tech-personal-statement-package.docx` / `ukgt-tech-full-endorsement-package.docx`.
- **UKGT Peer Review route (`routes/ukgt-research.js`):** mounted `/api/ukgt-research`. Branches: `ukgt_research_assessment` / `ukgt_research_statement` / `ukgt_research_package`. Prompts in `routes/ukgt-research-prompts.js` (3 exports). Endorsing-body routing RS/RAEng/BA. UK English. Eligibility Advisory Header inserted only when Q5 unambiguously states Tier 1 NOT YET ELIGIBLE. Word-count guardrail as above. File keys `ukgt_research_assessment_docx` / `_statement_docx` / `_package_docx`. `const EMAIL_Q_ID = 'ukgt_research_q0_user_email'`.
- **UKGT frontend tools:** O-1A pattern. Poll respective `window.*Tool` (100ms × 50). Section 0 back → `/uk-global-talent.html`. sessionStorage + question prefixes isolated per tool (`ukgt_tech_q<n>_<slug>`, `ukgt_tech_statement_q<n>_<slug>`, etc.). Q5 track uses `{ value, label }` objects — `_renderSelect` handles both formats. Email validation at section 0. No caching, no userId, DOCX-only, cancel return restores + review screen.
- **css/ibtool.css:** EB + O-1A + UKGT modal/screen visibility (`.active`) for all tiers incl. review screen IDs. `.tool-spinner` + `.tool-loading-*`. `.ib-hint-icon` + `.ib-hint-tooltip`. `.ib-payment-*`. Extend ID lists when adding a tool page. Imported in `styles.css` before `mobile.css`.
- **Review screen pattern (14 tool modules):** a `*-review-screen` div sits between the last form section and the payment screen. Rendered by `renderReview()` from `js/tool-review.js` via a thin wrapper. Last form Next label "Review your answers →". Payment back targets review; review back targets last form section; review continue → payment.
- **js/tool-review.js:** `renderReview({ screenId, prefix, sections, answers, onBack, onContinue })`. Builds `.tool-review-wrap` > sections > `.tool-review-item` (`.tool-review-label` + `.tool-review-value`). Empty/null → `"(not provided)"` with `.tool-review-value.empty`. Action row `.tool-review-actions`. All buttons via `addEventListener`.
- **css/cvtool.css review layout classes:** `.tool-review-wrap`, `.tool-review-section`, `.tool-review-item`, `.tool-review-label`, `.tool-review-value` (+ `.empty`), `.tool-review-actions`. Shared by CV/SOP; EB inherits via `ibtool.css` referencing same class names.
- **Stripe cancel URL for tool items:** `TOOL_CANCEL_PAGES` in `routes/stripe.js`. Format `${FRONTEND_URL}/${toolPage}?payment_cancelled=1&item=${itemType}`. Do NOT use `/?payment=cancelled` for these items.
- **payment_cancelled=1 bypass in stripe.js:** `checkPaymentReturn` IIFE preserves the URL when `payment_cancelled=1` + the item is a tool key. Do not clear the URL or show a generic banner.
- **Bypass list count in `js/stripe.js`:** report as an absolute count, not a cumulative delta. Grep `js/stripe.js` against the canonical Stripe item key list to verify rather than incrementing from memory.
- **booking.js cancel return:** saves `jc_booking_state` (plan, price, duration, name, email, country, destination, purpose, itemType) before `purchaseItem()`. `js/payment-return-booking.js` sets `window._bookingCancelReturn = true`, restores fields, advances to Step 2. Inline script in `consult-booking.html` skips auto-open on cancel return.
- **Document Checklist back buttons:** `js/checklist.js` injects `.tool-back-btn` on Stage 2 + Stage 3 only (Stage 1 has only X close). `window.oraboChecklistRestore(state)` restores state + `checklistWizardGoTo(3)`. Do NOT modify checklist.js for payment-return logic beyond these hooks.
- **Readiness Quiz back buttons:** `js/quiz.js` `renderQuestion(index)` re-creates `button#quizTopBack.tool-back-btn` only when `index > 0`. `showResults()` injects `button#quizResultsBack.tool-back-btn`. `window.oraboQuizRestoreScore(state)` restores `quizAnswers` + `showResults()`. Do NOT modify quiz.js beyond these hooks.
- **Readiness quiz options (`js/quiz.js`):** Q7 (`destination`) 6 options (US/Canada/UK/Europe/Australia/NZ, all `score:10`; `value` PascalCase). Q11 (`field`) 6 options (Healthcare/Tech/Finance/Education/Trades/Other; Other last `score:4`; Trades 5th `score:10`). `MAX_SCORE` auto-computed. `destination_value` from Q7 forwarded to report API.
- **init() sessionStorage preservation (14 tools):** `init()` checks both `payment_success=1` AND `payment_cancelled=1` before wiping; wipe only when NEITHER present.
- **Save & Resume Draft (14 paid tools):** `js/draft-store.js` + `js/draft-banner.js`. Each module imports both, defines `TOOL_KEY` (snake_case) + `TOOL_LABEL`. `saveCurrentAnswers(index)` ends with `saveDraft(TOOL_KEY, { answers, sectionIndex: index, outputType: ITEM_TYPE || null })`. `showDownloadScreen()` starts with `clearDraft(TOOL_KEY)`. `init()` shows `showResumeBanner(...)` gated on no payment URL params. CV/SOP: compute `_isPaymentUrl` from `URLSearchParams` BEFORE `handlePaymentReturn()`. EB/O1A/UKGT: gate on `!isSuccess && !isCancelled`. Do NOT apply to Document Checklist, Readiness Quiz, Consultation Booking.
- **localStorage draft keys:** `orabo_draft_cv`, `_sop`, `_eb1a`, `_eb2`, `_eb1a_letter`, `_eb2_letter`, `_eb1a_package`, `_eb2_package`, `_o1a`, `_o1a_letter`, `_o1a_package`, `_ukgt_tech`, `_ukgt_tech_statement`, `_ukgt_tech_package`, `_ukgt_research`, `_ukgt_research_statement`, `_ukgt_research_package`, `_pathway_match`. Distinct from sessionStorage payment-return keys (`orabo_*_answers`, `orabo_*_output_type`).
- **Free Preview Verdict (8 tools $140–$250):** free email-gated preview between review and payment. `routes/preview.js` (`previewLimiter` 3/IP/24h). Prompts in `routes/preview-prompts-eb/o1a/ukgt.js`, each with a `CRITICAL GUARDRAILS` block (4-block output) — never remove/soften. `claude-haiku-4-5-20251001` (max_tokens 400). NO verifyPayment, NO ai_cache. Frontend `js/preview-screen.js` (`showPreviewScreen(opts)`, 5 states A–E). Preview cached in draft (`lastPreview` + `lastPreviewEmail`, DRAFT_VERSION=2), not ai_cache; cached preview skips the Claude call. Skip logs `[preview-skip] <outputType>`.
- **Migration Worth It route (`routes/migration-worth-it.js`):** mounted `/api/migration-worth-it`; own `worthitLimiter` (5/IP/24h). No verifyPayment, no ai_cache. `claude-haiku-4-5-20251001` (max_tokens 250, `res.setTimeout(30000)`). Commentary failure non-fatal (fallback string). Tool recommendation deterministic: USA+HIGH_ACH→EB-1A, USA+TECH→O-1A, USA+other→EB-2, UK+TECH→UKGT Tech Assessment, TRADES→Document Checklist, else→CV Review. Brevo sets `WORTHIT_TARGET` + `WORTHIT_VERDICT`. `utils/worthit-data.js` holds data (SALARY_DATA in USD). 429: `{ error:'rate_limited', retryAfter:86400 }`.
- **Migration Worth It frontend (`migration-worth-it.html` + `js/worthit.js`):** standalone — no auth, no Supabase UMD. ES module importing ROLES/ROLE_GROUPS from `paycalc-data.js`, API_URL from `config.js`. Decodes `?result=<base64>` on load. Share buttons CSP-safe `addEventListener`. `#worthit-result` hidden via `css/worthit.css .hidden`.
- **Worth-it commentary must steer to the recommended Orabo tool** — 4th sentence names the engine-selected tool. System prompt NEVER list forbids external sites/portals/lawyers/consultants. Recommendation engine is the single source of truth.
- **Preview route skips verifyPayment** — never add it. Claude called directly after input validation + rate-limit. No ai_cache.
- **CV/SOP purchase verification:** `verify-payment.js` derives `authUserId` from the JWT; consumes an unconsumed `purchases` row atomically; anonymous flow uses the Stripe-session path. Do NOT add a hard userId requirement to `/api/cv` or `/api/sop`.
- **purchaseItem() guest checkout:** resolves email `bookingMeta.customer_email || bookingMeta.booking_email || user?.email || ''`. CV/SOP pass `{ customer_email: answers.user_email }`; consultation passes `{ booking_email }`. Backend `userId` optional for payment mode; required for subscription mode.
- **index.html paid tool buttons navigate to standalone pages** via `js/redirect-paid-tools.js`. `cvtool.js`/`soptool.js` NOT loaded on `index.html`.
- **Standalone tool pages (cv-tool, sop-tool, consult-booking):** no auth, no navbar/footer. cv/sop: no Supabase UMD — external boot script polls `window.cvTool`/`window.sopTool` (100ms × 50) then `init()`. consult-booking keeps Supabase UMD; auto-clicks `.consult-btn` after 300ms.
- **auth.js safe without Supabase UMD:** `const { createClient } = window.supabase || {}` — if absent, `supabaseClient` null, listeners skipped, `getCurrentUser()` returns null. Required for cvtool/soptool → stripe.js → auth.js chain.
- **Return URL security:** `login.html`/`signup.html` validate `?return=` with `/^[a-z0-9\-]+\.html$/` — only `.html` filenames.
- **stripe.js onDashboard guard:** skips `payment=success/cancelled` when `#dash-main` present.
- **Country-of-origin validation:** capture-phase `addEventListener('click', fn, true)`; `e.stopImmediatePropagation()` if empty; `.country-select-error.visible`. Do NOT add to `quiz.js`/`checklist.js`.
- **DOCX-only downloads for checklist/report:** capture-phase listener inside `.then()`; `e.stopImmediatePropagation()` blocks html2pdf. Never add PDF.
- **Brand sanitisation:** call `sanitizeBrand()` on every Claude response in `checklist.js` + `report.js` (cache hits + fresh). Write sanitised to `ai_cache`.
- **Dynamic Claude scholarship/university generation:** do NOT add DB queries to `routes/report.js` — Claude generates from `VISA_KNOWLEDGE_BASE`.
- **Checklist KB versioning:** when changing any rule in `checklist-pathway-data.js` or `checklist-knowledge.js`, bump `CHECKLIST_PROMPT_VERSION`. **Current: `v2.3-2026-05-21`.**
- **Checklist infeasible combinations:** in `INFEASIBLE_COMBINATIONS`. Never cache infeasible results. Never modify `verifyPayment()`. To add: `{ detect(pathwayKey, dest, origin), explanation, closest_alternative, action }` + bump version.
- **Opportunities submission form:** 8 required fields. Country `<select>`: USA, Canada, UK, Australia, NZ + `<optgroup label="Western Europe">` (19) + "Other". Frontend sends `{ title, organization, country, end_date, type, link, description, email }`; backend maps `organization`→`org`, `end_date`→`deadline`/`expires_at`. Duplicate check `LOWER(TRIM(link))` → 409 `{ error:'duplicate' }`. Rate limit 5/IP/10min (route-scoped POST only). `app.set('trust proxy', 1)` required.
- **Community Feed filter dropdown:** `<select id="oppFilterSelect">`. Three scholarship filters (Undergraduate, Masters, Scholarship) mutually exclusive — each maps to exactly one stored type. Do NOT change `scholarship` to a substring match. `.filter-btn` styles in `css/cards.css` retained (visa tracker + uni finder).
- **Opportunities publish notification:** `scheduler.js` hourly; `approved=true AND email_sent=false AND email != ''`; sets `email_sent=true`. `email_sent` added via SQL editor — do not include in CREATE TABLE.
- **User-supplied URLs:** pass through `safeURL()` before any `<a href>`/`setAttribute('href',...)`/`window.open()`. Defined inside `js/admin.js` IIFE; rejects all but `http:`/`https:`, returns `'#'`. `esc()` does NOT sanitise scheme — both required. `js/opportunities.js` applies the same pattern.
- **Always update CLAUDE.md** after any major feature addition, architecture change, or new integration.
- **CLAUDE.md mirror sync (App-authenticated, two-tier verification):** Mirror (`orabo-claudemd-mirror`) auto-syncs from `japaconnect` via `mirror-claudemd.yml`, authenticated by a GitHub App installation token (secrets `MIRROR_APP_ID` + `MIRROR_APP_PRIVATE_KEY`). Workflow fails RED if the mirror does not end byte-identical (SHA256 via Contents API). Manual re-sync: Actions → `mirror-claudemd.yml` → Run workflow. **Commit-message convention (hard requirement):** sync Action writes `sync: CLAUDE.md from japaconnect@<source_sha>`. **Verification (two-tier):** PRIMARY — mirror HEAD commit subject must contain the source SHA (`git ls-remote` + Commits API). SECONDARY — `curl -s https://raw.githubusercontent.com/Oraboss/orabo-claudemd-mirror/main/CLAUDE.md | grep -iE "<unique-string>"`; a miss within ~5 min of a PRIMARY PASS is raw CDN cache lag, not a failure. `MIRROR_PAT` retired — do not re-add.
- **Data freshness governance:** every static data file displaying numbers exports `DATA_FRESHNESS` `{ source, verified, cadence_days, tier }`. Tiers: `fx` (30d), `fees` (90d), `costliving` (90d), `eligibility` (365d). Run `node scripts/check-data-freshness.mjs`. `js/costliving-data.js` holds `FX_RATES`, `FX_BASELINE`, `DATA_FRESHNESS`. See `FRESHNESS.md`.
- **Source citations (`js/sources.js`):** every numeric figure must have a `SOURCES[key]` entry across the three data files with an opened authoritative URL, ISO `verified` date, and `license_note` (`public_domain`/`fair_use_attribution`/`orabo_estimate`). `sources-modal.js` accesses `window.OraboSources` lazily at click time. Add source entries + a `.source-tag[data-source-key]` button when adding a numeric tool.
- **Footer content must never include provenance/debug metadata.** The `## Sources & Authorities` footer on AI deliverables must contain only legal citations — never authoring metadata, methodology comments, bot-detection notes (`canonical-pattern-confirmed`, `web-fetched`, `Last verified:`). Guard pattern: log-only absence check in `routes/docgen.js generateDocx()` — `LEAK_TOKENS = /Cloudflare|web-fetched|canonical-pattern|Last verified/i` fires `console.error`; never runtime-strip. Extend the regex when new forbidden tokens are found.
- **Email delivery for checklist + report uses trusted email from `verify-payment.js`**, then `user_email` from body. Frontend reads `localStorage.orabo_user_email`. Resolution: `trustedEmail` → `user_email` → skip + log `[brevo-checklist] no recipient email available`. Both routes return `email_queued: boolean`.
- **Every Brevo fire-and-forget call must have `.then(success log)` + `.catch(err => detailed error log)` with a `[brevo-<context>]` prefix.** No silent swallowing.
- **Waitlist welcome email** fired by `routes/newsletter.js` when `destination` present OR `source === 'waitlist'`, AND source not suppressed, AND Brevo contact-create returned HTTP 201. Suppressed: `eb_waitlist`, `eb_waitlist_homepage`, `quiz_score_capture`, `opportunity_submission`. Logs `[brevo-waitlist-welcome] gate: {...}`.
- **Opportunity submission email flow:** `routes/opportunities.js` fires ack email (`[brevo-opp-ack]`) + `sendAdminNotification` (`[brevo-admin-opportunity]`). `scheduler.js` fires publish email (`[brevo-publish]`), LEFT JOIN `post_earn_rewards` for `approved_count`. `js/opportunities.js` newsletter call passes `source: 'opportunity_submission'`.
- **Consultation booking admin-notify (`[brevo-admin-booking]`):** fires in the Stripe webhook `checkout.session.completed` for `consultation_*` items — after confirmed payment, not at form submit.
- **Waitlist admin-notify (`[brevo-admin-waitlist]`):** fires in `routes/newsletter.js` for sources in `WAITLIST_NOTIFY_SOURCES` (`eb_waitlist`, `eb_waitlist_homepage`, `ukgt_waitlist`, `o1a_waitlist`) — regardless of `isNewContact`.
- **Mobile hamburger nav:** all styles in `css/hamburger.css` inside `@media (max-width:768px)`. Toggle IIFE in `script.js`; uses `.open` class. No desktop styles in hamburger.css; no duplicate toggle logic.
- **Mobile overflow fixes (`css/mobile.css`):** loaded last in `styles.css`; all rules inside `@media (max-width:768px)`. Owns `html, body { overflow-x: hidden }`, hero fixes, text scaling, grids. Do NOT touch other CSS files for mobile overflow.
- **Service worker:** permanently removed — `sw.js` + `js/register-sw.js` are stub comment files; tag absent from `index.html`. Do not re-add.
- **PWA icons:** `icons/icon-192.png` + `icons/icon-512.png` are binary copies of `orabo-icon-256x256.png` / `orabo-icon-512x512.png`. Replace both whenever the source logo changes. `site.webmanifest` references these paths.
- **SEO landing pages (`readiness-score.html`, `visa-eligibility.html`, `document-checklist.html`, `eb-immigration.html`):** minimal nav, no footer, load `styles.css` only, page layout in inline `<style>` in `<head>`. Use `.lp-nav`, `.lp-hero`, `.lp-section`, `.lp-cta-band`, `.lp-footer`. CTAs link to app sections. **CSP applies equally** — no inline `<script>` blocks. Interactive JS must be external (`eb-immigration.html` uses `js/eb-immigration.js`).
- **Blog articles (`blog/*.html`):** load `../styles.css`; use `.article-body`, `.callout`, `.data-table`, `.cta-block`, `.blog-nav`. Back link → `/blog/`. Target 900–1,200 words. End with a `.cta-block` linking to the relevant tool. Add a `.blog-card` to `blog/index.html` + a `.blog-teaser-card` to the homepage teaser whenever a new article ships.
- **Admin panel (`admin.html`):** direct URL only. Does NOT load Supabase UMD or `styles.css`. Auth via **Bearer JWT + `user_profiles.role='admin'`** (`requireAdmin`). All admin API routes under `/api/admin/`. `css/admin.css` NOT in `styles.css`. `js/admin.js` is a non-module IIFE.
- **Admin opportunity approval:** sets BOTH `approved = TRUE` AND `status = 'approved'`. Does NOT send the "post is live" email (scheduler handles it). Upserts `post_earn_rewards` (approved_count +1, phone from submitter_phone).
- **Post & Earn payout:** `POST /api/admin/post-earn/:email/mark-paid` resets `approved_count = 0`, sets `reward_paid_at = NOW()`, increments `total_paid`, keeps `reward_paid = FALSE`. Sends Brevo airtime notification. Env: `POST_EARN_THRESHOLD` (20), `POST_EARN_REWARD_PER_POST` (300), `POST_EARN_PAYOUT` (6000).
- **CV/SOP delivery emails:** `_sendCvEmail` / `_sendSopEmail` include the specific output type label. Upsell: `cv_review` + `cover_letter_only` → SOP Writer; all SOP types → Consultation. Fire a separate `POST /v3/contacts` setting `PURCHASE_DATE` + `LAST_TOOL`. `SOP_LABELS` map in `sop.js`.
- **Newsletter subscribe backward compatibility:** `/api/newsletter/subscribe` no longer requires `destination`; validated only when provided; contact added to list only when present.
- **Fire-and-forget writes from public pages:** must (1) skip silently if no session; (2) never block/alter observed UI; (3) sessionStorage flag to dedupe; (4) clear flag on error; (5) never modify the source module observed; (6) attach via MutationObserver/capture-phase. On index.html, JWT scripts use `readSessionFromStorage()` inlined from `js/auth.js`, not `getSession()`.
- **Saver-style scripts mirroring quiz.js (or any sacred module)** MUST document exact line numbers + selectors in a DEPENDENCY comment block, updated in the same commit as the underlying change OR backed by a smoke test (full quiz signed-in → observe `/api/quiz/save-results` POST → confirm `score` matches screen, not 0). Score 0 is mathematically impossible (min ~33%).
- **`.hidden` utility class:** defined in `css/quiz.css` — `display: none`. Do not redefine elsewhere.
- **EmailJS fully removed.** Do NOT add any new EmailJS calls anywhere. Use Brevo.
- **Payment verification on AI routes:** all AI generation routes call `verifyPayment()` before invoking Claude. Recipients derived from trusted source (Stripe session `customer_email` or Supabase auth via `userId`), never from `formAnswers.user_email`. Kill switch `AI_ROUTES_REQUIRE_STRIPE_SESSION`. Frontend modules must forward `stripe_session_id` from URL params. Any new AI-generation route MUST call `verifyPayment()` before any Claude call.
- **Visa Eligibility tool back/next buttons:** `js/eligibility.js` follows the `.tool-back-btn` / `.tool-next-btn` pattern — back on Q2–Q10 only, label `← Back`; final Next "See my matches →"; progress reads `questions.length`. `#eligibilityResultsBack` injected as first child of results; does NOT modify the localStorage free-use counter (`usesIncrementedThisSession` guard; "Check Again" resets).
- **Visa Eligibility upgrade modal z-index:** must stack above results — z-index 10010 (above `.quiz-overlay` 10000) in `css/subscription.css`. Keep both stacked.
- **Eligibility wizard markup duplication:** `#eligibilityOverlay` + `#upgradeModalOverlay` live in both `index.html` and `visa-eligibility.html` — change both together.
- **Petition Audit reviewer pattern (`routes/audit.js` + `audit-prompts.js` + `audit-prompts-ukgt.js` + `utils/document-ingest.js`):** shared reviewer route dispatching by `family` (`o1a`, `eb1a`, `eb2` live; `ukgt` via `route` → `promptKey = 'ukgt_'+route`). Do NOT add petition-review personas to generate prompts. Officer+counsel personas live in `audit-prompts.js`. Untrusted petition text always wrapped in `<uploaded_petition>...</uploaded_petition>`; `sanitizeForPrompt` strips delimiter tags before the prompt. Overall Posture always a named band, never numeric %. **EB-1A audit:** first-person self-petitioner voice is CORRECT — must NOT be flagged. **EB-2 audit:** Dhanasar requires ALL THREE prongs — failure on any single prong is a threshold failure. New family → add a key + extend `AUDIT_HTML_DISCLAIMERS` (do not create a separate route). **DEFERRED:** pre-payment file-readability precheck. **EB-1A tuning lever:** the CALIBRATION REQUIREMENT block in `EB1A_AUDIT_BLOCK1`.
- **Pathway Match route (`routes/pathway-match.js` + `-prompts.js`):** mounted `/api/pathway-match` with `checklistLimiter`. Single branch `pathway_match_report` (400 otherwise). Two-block prompt; max_tokens 6000; NO ai_cache. `verifyPayment` + `gate` (403). `stripe_session_id: null` in one_off_purchases insert (intentional). Brevo `[brevo-pathway-match]` / `[brevo-pathway-match-attr]`. File key `pathway_match_docx`; download `pathway-match-report.docx`. `TOOL_MANIFEST` is a local constant — do NOT import from stripe.js (circular). **Accuracy rails in Block 1 must never be weakened:** INFEASIBLE_COMBINATIONS absolute; DV Nigeria exclusion; EB-2 ROW no multi-decade backlogs (NIW removes PERM not the priority date queue); success bands high/moderate/low only; `[PROFILE GAP]` for blanks; attorney note in Sections 3+7; Section 7 ≥1 real matching tool.
- **Pathway Match frontend (`pathway-match-tool.html` + `js/pathway-match-tool.js`):** standalone pattern. Section 0 back → `/pathway-match`. Poll `window.pathwayMatchTool` via `js/autoopen-pathway-match.js`. sessionStorage `orabo_pathway_match_answers` / `_output_type`. TOOL_KEY `pathway_match`. NO preview screen ($19 below $140). Multiselect `q15_target_countries`: 4-country cap (disable unchecked at 4); pipe-delimited `|`; ≥1 required. POST field `formAnswers`. `.payment-cancel-notice` defined in `css/quiz.css` — not redefined in `css/pathway-match.css`.
- **RFE Response Copilot (`routes/rfe.js` + `rfe-prompts.js` + `rfe-email-html.js`):** $249 SKU `rfe_response`; mounted before global 20kb parser with `express.json({ limit: '8mb' })`. Four-persona pipeline (Analyst → Strategist → Drafter → Critic); Critic-revision loop capped at 2. `kbBlock` is a standalone `cache_control: ephemeral` block; Critic does NOT receive it. Two halt branches: Athletics-flag → referral DOCX (`one_off_purchases.tool = 'rfe_response_referral'`, `[rfe-referral-refund-needed]`); Strategist-blocked → blocking-gaps DOCX (`rfe_response_blocked`, `[rfe-blocked-refund-eligible]`). `one_off_purchases.tool` values category-suffixed (`rfe_response_eb1a` / `_eb2_niw` / `_o1a` / `_referral` / `_blocked`). Brevo via inline HTML (`rfe-email-html.js`) — no template IDs. Log prefixes `[rfe-analyst/strategist/drafter/critic/revision-cycle]`, `[brevo-rfe]`, `[brevo-rfe-attr]`.
- **Prompt-injection defense (mandatory on all AI routes):** every Claude call must prepend the `ANTI_INJECTION` constant (from `utils/anti-injection.js`) to Block 1; Block 2 user-supplied free-text wrapped in `<user_data>...</user_data>` before the `messages` array. Import the constant — do not copy-paste. `audit.js` exempt (uses `<uploaded_petition>` + `sanitizeForPrompt()`). Any new AI route MUST implement both before merge. Grep every AI output for literal `<user_data>` / `</user_data>` before shipping — tag echo into a DOCX is a customer-facing defect.
- **Dual-DB architecture:** OPS data → Orabo Prod (`rrkkbnoqubfmhrnxgngp`, Supabase JS client, `SUPABASE_URL` + `SUPABASE_SERVICE_ROLE_KEY`). Content data → Orabo Main (`fcogjoxcfmrcuvtreivm`, `pg.Pool` in `db/client.js`, `DB_*`). Never JOIN across DBs. Confirm the project ref before migrations.
- **No em dashes (—, U+2014) in published content.** Blog body, meta tags, public marketing copy must not contain em dashes; replace with commas/periods/colons/semicolons/parentheses. En dashes (–) for numeric ranges and hyphens in compounds are correct. Blog titles use ` | Orabo Blog`. Extends to AI-generated user-facing text — include "do not use em dashes" in every AI-route system prompt. Internal docs (this file, audit `.md`, prompts) are exempt.

---

## Design System

**Colours:** `--navy: #1a3c6e`, `--navy-dark: #0f2548`, `--navy-light: #2a5298`, `--accent: #4a90d9`, `--gold: #f5a623`, `--bg: #f4f7fb`, `--bg-alt: #eaf0f9`, `--text-muted: #5a6b85`.

**Fonts:** Inter (body), Playfair Display (decorative).

**Key utility classes:** `.btn-primary`, `.btn-gold`, `.btn-outline`, `.section-tag`, `.gradient-text`, `.filter-btn`.

**Nav logo:** `.nav-logo` wraps `<img src="/orabo-icon.svg">` + `<span class="nav-brand-name">Orabo</span>`. `.nav-brand-name`: Inter 700, 1.2rem, `#fff`. Both in `css/base.css`.

**Footer brand:** `.footer-brand` uses `<img src="/orabo-icon.svg">` + `<span>Orabo</span>` inside `.nav-logo` anchor — same SVG + sizing (36×36).

**Nav buttons (index.html):** `.nav-login-btn` (ghost), `.nav-signup-btn` (gold fill), `.nav-go-pro` (gold pill) — all use `!important` in `css/auth.css`.

**Landing-page hero button roles** (`eb-immigration.html`, `o1a-visa.html`, `uk-global-talent.html`): `.btn-hero-primary` (gold pill; the primary money-CTA in the hero only; defined locally per page); `.btn.btn-primary` (standard blue/navy; all non-hero buttons); ghost/outline (secondary). Never use `.btn.btn-primary` in a hero money-CTA.

**Parent organisation attribution:** `orabo.app` declares `oraboss.com` as parent org in footer text + JSON-LD. Attribution line "A product of Oraboss — building intelligent products at the intersection of Africa and the global economy." appears above the copyright line on landing pages; link uses `target="_blank" rel="noopener"`. `parentOrganization` + `sameAs` merged into the `Organization` JSON-LD block on `index.html`. `.footer-parent-org` + `.oraboss-link` live inline per landing page; on `index.html` in `css/footer.css`.

---

## Repositories

- **Frontend:** https://github.com/Oraboss/japaconnect — deployed at `https://orabo.app`
- **Backend:** https://github.com/Oraboss/japaconnect-backend — deployed on Railway
- **Branch (both):** main
- **Git user:** Abiodun Bello

---

## Pending / Parked Items

Priority tiers: **Blocking** → **High** → **Medium** → **Low**

### High — verification owed (code shipped, not confirmed live)
- **Live $19 Pathway Match production smoke** — real card → `cs_live_` session → `one_off_purchases` row with non-null `stripe_session_id` → 7-section DOCX → Brevo delivery. Do not mark Pathway Match Phase C complete until this passes.
- **`eb-immigration.html` live verification** — hero shows two CTA buttons (EB-1A $39 + EB-2 NIW $35), no email field; footer waitlist POSTs `{email, source:'eb_waitlist'}`; zero `Refused to execute inline script` in console.

### High — sitewide silent-CSP audit
- Static scan of every HTML file for inline `<script>` blocks with bodies (no `src`) that the edge `script-src 'self'` CSP would block. Prioritise lead-capture surfaces (SEO landings, marketing pages, blog). Failure is invisible to users — deliberate audit, not incidental discovery. Standing check: any page with a form/interactive JS gets a one-time DevTools console pass for `Refused to execute inline script` before it is considered shipped.

### Medium — footer template a11y
- `buildFooter()` / shared footer has two a11y issues capping Lighthouse Accessibility at ~93: (1) `color-contrast` — `--text-muted` greys on `--navy-dark` fail 4.5:1 (fix with a footer-context token like `.footer-text-muted { color: rgba(255,255,255,0.6) }`); (2) `heading-order` — footer `<h5>` after `<h1>`→`<h3>` skips H4 (replace with `<p class="footer-col-heading">`). After fixing, regenerate the SEO batch (`node scripts/generate-compare-seo.mjs`). Target ≥95.

### Medium — footer link inconsistency
- Discovery pages link "Submit Opportunity" to `/opportunities` instead of `/opportunities#submit-opportunity`. Footer has three hand-authored variants + the generator — edit `scripts/generate-compare-seo.mjs buildFooter(pfx)` (re-run after), `pathway-compare.html`, and `index.html`. `buildFooter()` absolute-path rule: use `/compare/` not `${pfx}compare/`.

### Low / Parked
- **Programmatic SEO subpages (Phase 2 of decongestion):** `/cost-of-living/<a>-vs-<b>`, `/take-home-pay/<role>-<country>`, `/visa-pathways/<slug>`, `/processing-times/<slug>` — ~120–150 new pages, same pattern as `scripts/generate-compare-seo.mjs`.
- **Navbar transparent-at-top mechanism (PARKED):** navbar defaults to transparent + white at `scrollY=0`; pages needing solid top-nav carry an inline `<style>` override (`pathway-compare.html` + 13 `/compare/` pages). Consolidation = make solid default in `css/nav.css` + init scroll state on `DOMContentLoaded`. **Trigger:** next new public page with a light area behind the top nav, OR a user report of invisible/flashing navbar.
- **`.btn-hero-primary` duplication (PARKED):** defined locally on 3 landing pages and already drifted (o1a `14px 28px`, ukgt `16px 32px`). Fix: pick one canonical definition, promote to `css/buttons.css`, delete the 3 copies. Trigger: next styling pass on these pages.
- **CSP `style=""` violations on landings:** `eb-immigration.html` (34) remaining after pass 1 of 2. Dedicated CSP cleanup pass owed (pass 2 after soak).
- **`js/eb-immigration.js` `.catch` swallows endpoint errors** — shows success message even on error/network-down. Address only if Railway logs show recurring `/api/newsletter/subscribe` errors; give the `.catch` a real inline error state.
- **Post-signup Pro upgrade breadcrumb (PLANTED, NOT HARVESTED):** `sessionStorage.orabo_post_signup_action = 'pro_upgrade'` set by `js/autoopen-visa-eligibility.js`. Add the reader in (1) `signup.html` post-email-confirmation flow and (2) `dashboard.html` `initDashboard()` (Free plan → `initiateStripeCheckout()` + clear the key). Separate scoped commit.

---

Historical context archived at /audit/CLAUDE-history-2026-06-06.md
