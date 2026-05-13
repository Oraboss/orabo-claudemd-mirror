# Orabo (JapaConnect) — Claude Code Guide

## Project Overview
Orabo is a platform empowering Africans with verified pathways to study, work, and migrate legally to the USA, Canada, UK, and Europe.

**Tagline:** "Japa is a dream. We help you plan it, verify it, and live it."

**Target users:** African aspirants (Nigeria, Ghana, Kenya, South Africa, Ethiopia, Tanzania, Uganda, Cameroon, and other African countries)

---

## Current State — Phase 8 + Phase 5A/5B + Phase EB Tier 1 + Phase EB Tier 2 + Phase EB Tier 3 + Phase O-1A + Email Pipeline Hardening + Review Screen Polish + Back Buttons & Cancel Return + Payment Integrity (Live)

> **Email pipeline last reviewed: 2026-05-12.** Active `[brevo-<context>]` contexts: `checklist`, `report`, `cv`, `cv-attr`, `sop`, `sop-attr`, `eb1a`, `eb1a-attr`, `eb2`, `eb2-attr`, `eb1a-letter`, `eb1a-letter-attr`, `eb2-letter`, `eb2-letter-attr`, `eb1a-package`, `eb1a-package-attr`, `eb2-package`, `eb2-package-attr`, `o1a`, `o1a-attr`, `o1a-letter`, `o1a-letter-attr`, `o1a-package`, `o1a-package-attr`, `consultation`, `waitlist-welcome`, `rejection`, `story-approved`, `story-rejected`, `payout`, `opp-ack`, `publish`.

- **Frontend:** Static site on GitHub Pages — `https://orabo.app` (repo is private)
- **Frontend GitHub repo:** https://github.com/Oraboss/japaconnect (private)
- **Backend:** Node/Express on Railway — health check: `GET /health` returns `{"status":"ok"}`
- **Phase 7 (SEO):** 4 SEO landing pages + blog directory with 3 articles added to root. No JS logic changes, no backend.
- **Phase 7C (Homepage discoverability):** Blog teaser section (3 cards) + EB waitlist banner added to `index.html` above footer. Nav Tools dropdown extended with Blog and EB Green Card Tools links. New `css/eb-waitlist.css` + `js/eb-index-waitlist.js`.
- **Phase 8 (Admin panel):** `admin.html` + `css/admin.css` + `js/admin.js` (frontend); `routes/admin.js` (backend). Token-gated at `x-admin-token` header. Manages opportunities, stories, scholarships, universities, Post & Earn rewards, users, and one-off purchases. Opportunity phone field added to form + backend. `post_earn_rewards` table added.
- **Phase 4 (Nav restructure):** Nav restructured with Tools dropdown grouping CV Review, SOP Writer, Visa Tracker, Cost of Living, Pay Calculator. Sticky mobile CTA bar added at bottom of viewport on `index.html`.
- **Phase 5A (Post-purchase email sequences):** CV and SOP delivery emails now include the specific output type purchased, upsell paragraph (CV→SOP Writer; SOP→Consultation), and fire-and-forget Brevo contact attribute update (`PURCHASE_DATE`, `LAST_TOOL`) after every successful generation.
- **Phase 5B (Quiz score email capture):** Free quiz on `index.html` shows an email capture block after score reveal (MutationObserver on `#quizResultsScreen`). `js/quiz-email-capture.js` (non-module) injects the UI, POSTs to `/api/newsletter/subscribe` with `quiz_score`/`quiz_date`/`source`. Backend `/api/newsletter/subscribe` extended: `destination` now optional; quiz attributes (`QUIZ_SCORE`, `QUIZ_DATE`, `QUIZ_LEAD`) set when `quiz_score` is present.
- **Phase EB Tier 1 (EB immigration tools — Tier 1 only):** `eb1a-tool.html` + `eb2-tool.html` live. Routes: `routes/eb1a.js` + `routes/eb2.js` (NO ai_cache reads — every assessment generated fresh). Frontend modules: `js/eb1atool.js` + `js/eb1atool-questions.js` + `js/eb2tool.js` + `js/eb2tool-questions.js`. CSS: `css/ibtool.css` (imported in `styles.css` before `mobile.css`). Stripe items: `eb1a_assessment` ($39), `eb2_assessment` ($35). Tooltip pattern: `.ib-hint-icon` + `.ib-hint-tooltip` — CSS hover only, no JS. Legal disclaimer appended to every EB output before docgen. Nav "Soon" badge removed from EB entry. Tool links added to `eb-immigration.html` above the tools breakdown section. **Payment return fixed:** `js/stripe.js` `checkPaymentReturn` IIFE now bypasses the generic banner for EB items, preserving the URL for the EB tool module to handle — identical to the CV/SOP pattern. Tier 1 now captures a final supporting-evidence and recommendation-letter inventory section (EB-1A Section E, EB-2 NIW Section D), and the AI weighs this when rating criteria/prongs and produces a new 'Evidence Position Summary' output block.
- **Phase EB Tier 2 (EB immigration tools — Tier 2):** `eb1a-letter-tool.html` + `eb2-letter-tool.html` live. Routes: `routes/eb1a.js` + `routes/eb2.js` each extended with an `eb1a_letter` / `eb2_letter` outputType branch (still NO ai_cache reads — every petition letter generated fresh). Frontend modules: `js/eb1a-letter-tool.js` + `js/eb1a-letter-questions.js` + `js/eb2-letter-tool.js` + `js/eb2-letter-questions.js`. Stripe items: `eb1a_letter` ($150), `eb2_letter` ($140). sessionStorage keys: `orabo_eb1a_letter_answers` / `orabo_eb1a_letter_output_type` and `orabo_eb2_letter_answers` / `orabo_eb2_letter_output_type`. Module exports `window.eb1aLetterTool` / `window.eb2LetterTool = { init, start }`. Each generates a 1,500–2,000 word attorney-voice petition letter; no caching; attorney disclaimer appended; DOCX-only download; Brevo delivery + attribute update with `[brevo-eb1a-letter]` / `[brevo-eb2-letter]` / `[brevo-eb1a-letter-attr]` / `[brevo-eb2-letter-attr]` log prefixes; `one_off_purchases` audit row written. Mandatory email validation at section 0 (mirrors the May 11 fix). AI outputs HTML (`<h1>`, `<h2>`, `<h3>`, `<p>`, `<strong>`); code-fence stripping applied before DOCX generation. `css/ibtool.css` extended with Tier 2 modal/screen IDs. Tier 2 cards added to `eb-immigration.html`.
- **Phase EB Tier 3 (EB immigration tools — Tier 3):** `eb1a-package-tool.html` + `eb2-package-tool.html` live. Routes: `routes/eb1a.js` + `routes/eb2.js` each extended with an `eb1a_package` / `eb2_package` outputType branch (NO ai_cache reads — every package generated fresh). Frontend modules: `js/eb1a-package-tool.js` + `js/eb1a-package-questions.js` + `js/eb2-package-tool.js` + `js/eb2-package-questions.js`. Stripe items: `eb1a_package` ($250), `eb2_package` ($240). sessionStorage keys: `orabo_eb1a_package_answers` / `orabo_eb1a_package_output_type` and `orabo_eb2_package_answers` / `orabo_eb2_package_output_type`. Module exports `window.eb1aPackageTool` / `window.eb2PackageTool = { init, start }`. EB-1A package delivers 6 HTML sections: petition letter (1,500–2,000w), criterion scorecard table (all 10 criteria), evidence checklist by exhibit label, expert letter template, filing strategy brief with 5 action items, attorney disclaimer. EB-2 NIW package delivers 7 HTML sections: petition letter, Dhanasar three-prong scorecard, Standalone Proposed Endeavor Statement, evidence checklist by prong (Exhibit A–D), expert letter guidance, filing notes with RFE scenarios, attorney disclaimer. `max_tokens: 10000` for both (up from 8000 for Tier 2). `res.setTimeout(180000)` on both handlers. Code-fence stripping before `generateDocx()`. Brevo delivery + attribute update with `[brevo-eb1a-package]` / `[brevo-eb2-package]` / `[brevo-eb1a-package-attr]` / `[brevo-eb2-package-attr]` log prefixes. `one_off_purchases` audit row written. Question ID prefix isolation: `eb1a_package_q<n>_<slug>` / `eb2_package_q<n>_<slug>` (distinct from Tier 2 prefix). `css/ibtool.css` extended with Tier 3 modal/screen IDs. Tier 3 cards added to `eb-immigration.html`. Customer email safety net in `routes/stripe.js` validates email regex before passing to Stripe (strips invalid, never 500s). `js/stripe.js` `checkPaymentReturn` IIFE bypass list extended to 12 item keys (added `eb1a_package`, `eb2_package`). Subsequent O-1A phase extended the bypass list to 15 item keys.
- **CV + SOP Adjudicator-First Rewrite:** Both tools rewritten with two-pass gatekeeper-then-drafter system prompts, per-country format rules (9 countries + DEFAULT), and SOP-track-specific adjudicator personas + output scaffolds (`utils/document-norms.js`). CV adds an explicit no-fabrication rule: the AI inserts `[INSERT METRIC — e.g. team size, $ figure, % improvement]` placeholders rather than inventing numbers, and flags every placeholder in the Top 5 Priority Actions. SOP retains its existing Rule 1 (no invented experiences) and adds a tightened Orabo Notes scorecard with a verdict line (YES/MAYBE/NO pile). `max_tokens` bumped to 6000 for both routes. `ai_cache` reads and writes **bypassed** in both routes — every CV/SOP is generated fresh. SOP form gains a required `sopTrack` dropdown (nine tracks: Master's admissions, PhD admissions, Chevening, Commonwealth, Fulbright, MasterCard Foundation, DAAD, Other named scholarship, Visa SOP) with a conditional `otherScholarshipName` follow-up textarea. `destinationCountry` dropdown in both CV and SOP forms updated from grouped "Europe" option to nine individual countries (USA, Canada, United Kingdom, Germany, Netherlands, Ireland, Australia, New Zealand, Portugal, Other). Chevening track produces four labelled essays (Leadership and Influence, Networking, Studying in the UK, Career Plan), not one document. MCF track structures the entire essay around the give-back arc. Backend track normalisation maps display strings to internal keys via `normalizeTrack()` in `routes/sop.js`.
- **Email Pipeline Hardening (May 11 2026):** Opportunity submission email flow rebuilt — acknowledgement email fires from `routes/opportunities.js` with submission details table + phone-conditional Post & Earn paragraph; `scheduler.js` publish-notification rewritten with warmer copy and LEFT JOIN to `post_earn_rewards` to surface `approved_count`. Newsletter welcome gate hardened: now suppresses `opportunity_submission` source + only fires on first-time contacts (Brevo 201 status check). Every Brevo fire-and-forget call now logs gate decisions + `[brevo-<context>]` success/error. All transactional email templates updated to use `https://orabo.app/orabo-icon-256x256.png` as hosted logo. PWA icons replaced (`icons/icon-192.png` + `icons/icon-512.png` now correct navy/gold logo). `site.webmanifest` entries corrected to point at `/icons/` paths.
- **Checkout diagnosability (2026-05-11):** `purchaseItem()` catch block in `js/stripe.js` now logs `console.error('[checkout]', err)` before the alert so future failures are immediately identifiable in DevTools. Backend `/api/create-checkout-session` verified working for all standalone tool items (cv_review, cv_cover_letter, cover_letter_only, sop_scratch, sop_review, sop_rewrite, eb1a_assessment, eb2_assessment) — CORS passes, Stripe sessions created, ITEM_PRICES map complete. If "Could not start checkout" reappears: check `[checkout]` in DevTools console → most likely causes are (1) Stripe key rotation needed in Railway, (2) browser ad-blocker blocking fetch to Railway, (3) Railway cold-start timeout.
- **Mandatory email validation at section 0 (2026-05-11):** CV, SOP, EB-1A, EB-2 tool forms now block advancement past section 0 unless `user_email` passes `isValidEmail()` regex (`/^[^\s@]+@[^\s@]+\.[^\s@]+$/` + length bounds 5–254). Inline `.error-text.visible` (existing mechanism) surfaces the message; `.tool-question input.input-error` (added to `css/cvtool.css`) turns the field border red. Error clears on input. Fixes Stripe `email_invalid` 500s previously seen at 18:42 / 18:44 / 18:46 on 11 May where partial values like `"ade"` and `"cc"` bypassed `<input type="email">` soft validation on mobile.
- **Review Screen Polish (all 8 tools):** A review-before-payment screen has been inserted between the last form section and the payment screen on all 8 tools (CV, SOP, EB-1A Tier 1, EB-2 Tier 1, EB-1A Tier 2, EB-2 Tier 2, EB-1A Tier 3, EB-2 Tier 3). Screen IDs: `cv-review-screen`, `sop-review-screen`, `eb1a-review-screen`, `eb2-review-screen`, `eb1a-letter-review-screen`, `eb2-letter-review-screen`, `eb1a-package-review-screen`, `eb2-package-review-screen`. Shared renderer in `js/tool-review.js` (exported `renderReview()`); each module calls a thin wrapper. Last form section's Next button now reads "Review your answers →" (was "Continue to payment →"); payment screen back button returns to review screen (not last form section). Stripe cancel URL for these 12 items (`cv_review`, `cv_cover_letter`, `cover_letter_only`, `sop_scratch`, `sop_review`, `sop_rewrite`, `eb1a_assessment`, `eb2_assessment`, `eb1a_letter`, `eb2_letter`, `eb1a_package`, `eb2_package`) now returns to the tool's own page with `?payment_cancelled=1&item=<key>` — backend `TOOL_CANCEL_PAGES` map in `routes/stripe.js`. `js/stripe.js` `checkPaymentReturn` IIFE extended to preserve URL for `payment_cancelled=1` for those 12 keys. Each tool's `init()` now preserves sessionStorage when either `payment_success=1` or `payment_cancelled=1` is in the URL; on cancel return, answers are restored from sessionStorage and the user lands on the review screen.
- **EB pricing v2 (2026-05-12):** All six EB items repriced ahead of production launch. eb1a_assessment $35→$39, eb2_assessment $29→$35, eb1a_letter $75→$150, eb2_letter $65→$140, eb1a_package $120→$250, eb2_package $99→$240. Tier 1 kept low as funnel-entry; Tier 2 and Tier 3 anchored to attorney-fee-relative value ($4k–$15k market). CV/SOP/consultation pricing unchanged.
- **Phase O-1A (O-1A Extraordinary Ability Visa suite — all 3 tiers):** `o1a-tool.html` + `o1a-letter-tool.html` + `o1a-package-tool.html` live, plus `o1a-visa.html` landing page. Route: `routes/o1a.js` (NEW — mounted at `/api/o1a`); branches on `outputType`: `o1a_assessment` → Tier 1 (plain text/markdown, 8-criterion scorecard), `o1a_letter` → Tier 2 (HTML, petition letter + mandatory peer consultation letter), `o1a_package` → Tier 3 (HTML, 7-section full petition package including Petitioner Support Brief). Petitioner-voice enforcement in Tier 2/3 system prompts bans first-person pronouns referring to the beneficiary. Non-immigrant framing throughout (3-year renewable work visa, not a green card). Frontend modules: `js/o1atool.js` + `js/o1atool-questions.js` + `js/o1a-letter-tool.js` + `js/o1a-letter-questions.js` + `js/o1a-package-tool.js` + `js/o1a-package-questions.js`. Stripe items: `o1a_assessment` ($39), `o1a_letter` ($150), `o1a_package` ($250). sessionStorage keys: `orabo_o1a_answers`/`orabo_o1a_output_type`, `orabo_o1a_letter_answers`/`orabo_o1a_letter_output_type`, `orabo_o1a_package_answers`/`orabo_o1a_package_output_type`. Question ID prefixes: `o1a_q<n>_<slug>`, `o1a_letter_q<n>_<slug>`, `o1a_package_q<n>_<slug>`. Download filenames: `o1a-assessment.docx`, `o1a-petition-letter.docx`, `o1a-full-petition-package.docx`. Brevo delivery + attribute update with `[brevo-o1a]`/`[brevo-o1a-attr]`, `[brevo-o1a-letter]`/`[brevo-o1a-letter-attr]`, `[brevo-o1a-package]`/`[brevo-o1a-package-attr]` prefixes. `css/ibtool.css` extended with all 15 new O-1A screen IDs. `js/stripe.js` `checkPaymentReturn` bypass list extended from 12 to 15 item keys (added `o1a_assessment`, `o1a_letter`, `o1a_package`). `TOOL_CANCEL_PAGES` map in `routes/stripe.js` extended with 3 new O-1A cancel URLs. Nav Tools dropdown + homepage US Immigration tools section updated. Back button on Section 0 of all 3 O-1A tools routes to `/o1a-visa.html`.
- **Back buttons + cancel return for free tools (2026-05-12):** In-tool back buttons added to Document Checklist wizard (Stages 2 and 3 only — Stage 1 has NO back button, only the X close, to avoid blank-page navigation on the bare standalone page) and Readiness Report quiz (dynamically injected `.tool-back-btn#quizTopBack` in `renderQuestion()` on Questions 2–15 only — Question 1 has NO back button, only the X close, same reason). Stage 2 → Stage 1; Stage 3 → Stage 2. Q2–Q15 go to previous question. A `#quizResultsBack` `.tool-back-btn` is also injected as first child of `#quizResultsScreen` by `showResults()` — clicking it hides results, shows `#quizQuestionScreen`, and re-renders Question 15 with the saved answer preserved. `TOOL_CANCEL_PAGES` in `routes/stripe.js` extended with 6 new entries (`document_checklist`, `full_doc_checklist`, `readiness_report`, `consultation_express`, `consultation_full`, `consultation_doc_review`) — now 21 total keys. `js/stripe.js` `checkPaymentReturn` bypass list extended to 21 keys matching. Cancel return state restoration: `payment-return-checklist.js` handles `payment_cancelled=1` (preserves sessionStorage, intercepts autoopen, calls `window.oraboChecklistRestore`, shows amber cancel notice on Stage 3); `payment-return-quiz.js` handles `payment_cancelled=1` (calls `window.oraboQuizRestoreScore`, shows cancel notice above pay button); new `js/payment-return-booking.js` handles consultation cancel return (restores step 1 fields, advances to step 2, shows cancel notice). `js/booking.js` saves `jc_booking_state` to sessionStorage before Stripe redirect. `css/quiz.css` adds `.payment-cancel-notice` (amber/gold, shared by all 3 tools).
- **Payment Integrity (P1-01 + P1-02, 2026-05-12):** New `utils/verify-payment.js` gates all 7 AI generation routes (`cv`, `sop`, `eb1a`, `eb2`, `o1a`, `checklist`, `report`). Verification order: (1) Stripe session — `session.payment_status === 'paid'` AND `metadata.itemType === outputType` AND `one_off_purchases.stripe_session_id` not consumed; (2) Pro subscription (`user_profiles.pro_status === true` AND `pro_expires_at > NOW()`, checklist/report only); (3) Pre-existing `purchases` row with `consumed_at IS NULL` (consumed atomically on first use). New schema: partial unique index `one_off_purchases_session_id_unique` (where `stripe_session_id IS NOT NULL`); new `purchases.consumed_at TIMESTAMPTZ` column. Stripe success URLs now append `&session_id={CHECKOUT_SESSION_ID}` (Stripe-substituted template variable, client cannot forge). All 11 tool modules + 2 payment-return scripts read `session_id` from `URLSearchParams` and forward as `stripe_session_id` in the POST body. Recipient email for Brevo delivery now derived from trusted source (`session.customer_email` or `supabase.auth.admin.getUserById(userId).email`), never from `formAnswers.user_email`. Kill switch: `AI_ROUTES_REQUIRE_STRIPE_SESSION=true` env var on Railway (currently ON; `false` = shadow-mode logging only). Validated end-to-end with real $9 doc_checklist purchase + Stripe sandbox refund.
- **Webhook Hardening (P1-03 + P1-04 + P1-05, 2026-05-13):** `routes/stripe.js` now idempotent via `processed_webhook_events(event_id TEXT PRIMARY KEY, event_type TEXT, processed_at TIMESTAMPTZ)` ledger. After signature verification and before event-type branching, insert `event.id`; on duplicate-key error (`23505`) return 200 immediately. All internal errors inside the webhook handler now return 200 (only signature verification can return 4xx) — prevents Stripe retry storms. `invoice.payment_failed` revokes `pro_status = false` when `attempt_count >= 2` (existing `customer.subscription.deleted` remains as final fallback at end of dunning cycle). New `charge.refunded` handler looks up Checkout Session via `payment_intent`, reads `metadata.userId` + `metadata.itemType`, deletes the matching `purchases` row where `consumed_at IS NULL`; consumed rows left intact. Anonymous flows auto-mitigated by `one_off_purchases.stripe_session_id` single-use ledger from Phase 1. Stripe Dashboard webhook endpoint subscribed to `charge.refunded` (added manually post-deploy). Validated with real Stripe sandbox refund of the $9 doc_checklist purchase — `[stripe-webhook] charge.refunded` log line confirmed. All 5 findings closed — audit recorded in `audit/pass-1-payment-integrity.md` in the backend repo.

---

## Architecture — Multi-Page App

The frontend is a **multi-page app** with 12 core app HTML files + 6 EB tool pages + 4 O-1A pages + 1 admin page + 4 SEO landing pages + a `blog/` directory:

| File | Purpose | Auth required |
|---|---|---|
| `index.html` | Main marketing site — free tools + marketing cards for paid services (CV/SOP/Consult Start buttons redirect to standalone tool pages) | No |
| `signup.html` | Sign-up page | No |
| `login.html` | Login + forgot-password page | No |
| `dashboard.html` | Personal dashboard, Pro upgrade, and one-off paid services (CV Review, SOP Writer, Consultation) | Yes — redirects to `login.html` if unauthenticated |
| `cv-tool.html` | Standalone CV Review tool — public, auto-opens CV modal on load | No — open to all; name + email captured in form (section 1) |
| `sop-tool.html` | Standalone SOP Writer tool — public, auto-opens SOP modal on load | No — open to all; name + email captured in form (section 1) |
| `consult-booking.html` | Standalone Consultation booking — public, auto-opens booking modal on load | No — open to all; name + email captured in booking form |
| `compare-cities.html` | Pro feature: compare cost of living across 4 cities | Pro localStorage gate |
| `all-countries-pay.html` | Pro feature: take-home pay across all 9 countries for all 35 roles | Pro localStorage gate |
| `doc-checklist.html` | Dashboard iframe: auto-opens document checklist wizard (free preview) | No (tool self-gates) |
| `readiness-report.html` | Dashboard iframe: auto-opens readiness quiz | No (tool self-gates) |
| `full-doc-checklist.html` | Dashboard iframe: auto-opens document checklist wizard (Pro card) | No (tool self-gates) |
| `readiness-score.html` | SEO landing page — "migration readiness quiz Africa" — drives to `/#readiness` | No |
| `visa-eligibility.html` | SEO landing page — "visa eligibility checker Africa" — drives to `/#eligibility` | No |
| `document-checklist.html` | SEO landing page — "immigration document checklist" — drives to `/#tools` | No |
| `eb-immigration.html` | SEO landing page — EB-1A & EB-2 NIW early access — email waitlist via `/api/newsletter/subscribe` | No |
| `eb1a-tool.html` | Standalone EB-1A Eligibility Assessment — no navbar/footer, no Supabase UMD; polls `window.eb1aTool` then calls `init()`; anonymous flow; Stripe returns here | No |
| `eb2-tool.html` | Standalone EB-2 NIW Eligibility Assessment — same pattern as `eb1a-tool.html` | No |
| `eb1a-letter-tool.html` | Standalone EB-1A Petition Support Letter (Tier 2) — polls `window.eb1aLetterTool`; 6 sections, 22 petition-content questions; $150; DOCX download | No |
| `eb2-letter-tool.html` | Standalone EB-2 NIW Petition Letter (Tier 2) — polls `window.eb2LetterTool`; 5 sections, 20 petition-content questions; $140; DOCX download | No |
| `eb1a-package-tool.html` | Standalone EB-1A Full Petition Package (Tier 3) — polls `window.eb1aPackageTool`; 7 sections (A–F + Your details), 25 questions; $250; 6-component DOCX download | No |
| `eb2-package-tool.html` | Standalone EB-2 NIW Full Petition Package (Tier 3) — polls `window.eb2PackageTool`; 6 sections (A–E + Your details), 23 questions; $240; 7-component DOCX download | No |
| `o1a-visa.html` | O-1A landing page — 3-tier tool card grid, 8-criteria explainer, petitioner requirements, EB upsell, email waitlist; links to 3 standalone O-1A tool pages | No |
| `o1a-tool.html` | Standalone O-1A Eligibility Assessment (Tier 1) — polls `window.o1aTool`; 5 sections (Section 0 + A–D), 12 questions; $39; DOCX download; back on Section 0 → `/o1a-visa.html` | No |
| `o1a-letter-tool.html` | Standalone O-1A Petition Support Letter + Peer Consultation Letter (Tier 2) — polls `window.o1aLetterTool`; 6 sections, 22 questions; $150; DOCX download | No |
| `o1a-package-tool.html` | Standalone O-1A Full Petition Package (Tier 3) — polls `window.o1aPackageTool`; 7 sections, 25 questions; $250; 7-section DOCX download including Petitioner Support Brief | No |
| `admin.html` | Admin panel — token-gated, not linked from any public page; direct URL only; loads only `css/admin.css` + `js/admin.js` | Token (`ADMIN_SECRET`) |
| `blog/index.html` | Blog index — 5 article cards | No |
| `blog/cost-of-living-lagos-london-toronto.html` | Article: Lagos vs London vs Toronto cost of living 2025 | No |
| `blog/express-entry-document-checklist.html` | Article: Complete Express Entry document checklist (Canada) | No |
| `blog/uk-skilled-worker-eligibility.html` | Article: UK Skilled Worker eligibility guide | No |
| `blog/eb2-niw-vs-eb1a.html` | Article: EB-2 NIW vs EB-1A comparison | No |
| `blog/nurse-salary-uk.html` | Article: UK nurse salary after tax | No |

`index.html` is always logged-out — no auth in navbar. Standalone tool pages (`cv-tool.html`, `sop-tool.html`, `consult-booking.html`) have no navbar/footer and require **no authentication** — they auto-open the tool modal via a DOMContentLoaded inline script. Name and email are captured as the first section of the tool form. Tool type passed via sessionStorage (`jc_cv_tool`, `jc_sop_tool`, `jc_consult_booking`) set by `redirect-paid-tools.js`. `compare-cities.html` / `all-countries-pay.html` gate via `localStorage.getItem('orabo_pro_status') === 'true'`; hide header with `?embed=true`. `doc-checklist.html`, `readiness-report.html`, `full-doc-checklist.html` are standalone embedded pages (no navbar/footer) — import tool module + `autoopen-*.js`; Stripe redirects use `window.top.location.href`.

---

## Frontend Structure

```
japaconnect/
├── index.html               # Main marketing site (public, free tools + marketing cards; paid tool buttons navigate to cv-tool.html / sop-tool.html / consult-booking.html)
├── signup.html              # Sign-up page (standalone, own inline script)
├── login.html               # Login + forgot-password page (standalone, own inline script)
├── dashboard.html           # Protected dashboard + one-off services: CV Review, SOP Writer, Consultation (loads cvtool.js, soptool.js, booking.js as modules)
├── cv-tool.html             # Standalone CV tool — public; DOMContentLoaded inline script auto-opens modal; Stripe returns here; no auth
├── sop-tool.html            # Standalone SOP tool — public; DOMContentLoaded inline script auto-opens modal; Stripe returns here; no auth
├── consult-booking.html     # Standalone consultation — public; DOMContentLoaded auto-clicks .consult-btn; stripe.js shows calendlyOverlay on return; no auth
├── compare-cities.html      # Pro: compare CoL across 4 cities (standalone, Pro localStorage gate)
├── all-countries-pay.html   # Pro: pay comparison all countries (standalone, Pro localStorage gate)
├── doc-checklist.html       # Dashboard iframe: checklist wizard auto-opens; imports checklist.js + autoopen-checklist.js
├── readiness-report.html    # Dashboard iframe: quiz auto-opens; imports quiz.js + autoopen-quiz.js
├── full-doc-checklist.html  # Dashboard iframe: same as doc-checklist.html, shown to Pro users in dashboard
├── styles.css               # Global styles (imports all css/ partials)
├── script.js                # Legacy interactivity (pre-module)
├── site.webmanifest         # PWA web app manifest (name, icons, theme, display mode) — linked from index.html
├── sw.js                    # Service worker permanently removed — stub comment file only
├── icons/
│   ├── icon-192.png         # PWA icon 192×192 (navy background, globe + Orabo text)
│   └── icon-512.png         # PWA icon 512×512
├── css/
│   ├── base.css             # :root CSS variables, reset, .container (max-width 1180px), .section, embed-mode rules
│   ├── hero.css             # Hero section — grid-template-columns:1fr auto collapsed at 1024px; phone mockup hidden at 1024px; float animation
│   ├── features.css         # features-grid (3→2→1 col), steps-row (flex→column at 768px), trust-grid (4→2→1 col)
│   ├── visatracker.css      # vpt-layout: grid-template-columns 320px 1fr — responsive breakpoint is 900px, not 768px
│   ├── auth.css             # Nav login/signup button styles (.nav-login-btn, .nav-signup-btn)
│   ├── subscription.css     # Upgrade modal + .nav-go-pro
│   ├── quiz.css             # Readiness quiz + report modal; .country-select-error/.visible for validation; .quiz-email-capture styles + .hidden utility (Phase 5B)
│   ├── cvtool.css           # CV Review + SOP Writer tool styles (shared); CSS-based initial hide rules for both tool modals
│   ├── hamburger.css        # Mobile hamburger nav — full-screen overlay, X animation, body.menu-open scroll lock; all rules inside @media (max-width: 768px)
│   ├── eb-waitlist.css      # EB waitlist banner + blog teaser section styles for index.html; loaded before mobile.css in styles.css
│   ├── admin.css            # Admin panel styles — only loaded by admin.html, NOT imported into styles.css
│   ├── mobile.css           # Mobile responsive fixes — hero overflow, grid single-column, text scaling. Loaded last in styles.css.
│   ├── dashboard-page.css   # Dashboard page styles (linked directly from dashboard.html, NOT in styles.css)
│   └── dashboard.css        # (legacy — dashboard is now dashboard.html)
├── js/
│   ├── auth.js              # Supabase session, profile cache, openAuthModal (redirects to login.html); defensive window.supabase check — safe to import on pages without Supabase UMD
│   ├── embed.js             # Sets html.embed-mode class synchronously; hides navbar/footer in iframe
│   ├── stripe.js            # Stripe checkout + payment return handling; payment=success/cancelled skipped on dashboard.html; purchaseItem resolves email from bookingMeta.customer_email → booking_email → user.email; checkPaymentReturn IIFE preserves URL for payment_cancelled=1 for the 21 tool items
│   ├── subscription.js      # isPro() check (Supabase + localStorage fallback) + upgrade modal
│   ├── checklist.js         # Document checklist wizard engine (module — do not modify for payment-return/bypass/origin logic)
│   ├── quiz.js              # Readiness quiz engine + AI report (module — do not modify for payment-return logic)
│   ├── booking.js           # Consultation booking modal + Stripe; wrapped in IIFE with `!modal` guard; step 1 has `.tool-back-btn` "← Back to Home" inserted at top; step 2's `.prev-step` button replaced by `.tool-back-btn` "← Back" → goToStep(1)
│   ├── visatracker.js       # Processing times logic — 18 visas, list + detail + Chart.js trend
│   ├── visatracker-data.js  # Exported VISAS[], TREND_LABELS, TREND_DATA (split from visatracker.js)
│   ├── costliving.js        # Cost of living — 33 cities, grouped dropdown, single-card + compare modal (progressive city slots)
│   ├── paycalc.js           # Take-home pay — 35 roles, grouped dropdown, per-country result + compare modal; imports data from paycalc-data.js
│   ├── paycalc-data.js      # Exported ROLES[], ROLE_GROUPS, SALARY_DATA (split from paycalc.js)
│   ├── tool-review.js       # Shared ES module — exports renderReview(opts) used by all 8 tool modules; opts: { screenId, prefix, sections, answers, onBack, onContinue }; renders .tool-review-wrap with section/item/label/value layout; empty answers show "(not provided)"
│   ├── cvtool.js            # CV Review + Cover Letter Generator — loaded on dashboard.html and cv-tool.html; imports stripe.js + config.js + tool-review.js (no auth.js); renders ALL_SECTIONS = CV_SECTIONS directly; screen visibility via classList.add('active'); review screen between last form section and payment; payment return reads URL param + sessionStorage (success→generate, cancel→review screen); exports window.cvTool = { init, start }
│   ├── cvtool-questions.js  # CV_SECTIONS[] — 6 sections: 'Your details' (user_name, user_email) + 5 original sections
│   ├── soptool.js           # SOP Writer — loaded on dashboard.html and sop-tool.html; same pattern as cvtool.js; imports tool-review.js; review screen between last section and payment; ALL_SECTIONS = SOP_SECTIONS; exports window.sopTool = { init, start }
│   ├── soptool-questions.js # SOP_SECTIONS[] — 7 sections: 'Your details' (user_name, user_email) + 6 original sections
│   ├── redirect-paid-tools.js # Non-module: wires CV/SOP/Consult Start buttons on index.html — saves type to sessionStorage and navigates to standalone page
│   ├── autoopen-checklist.js # Triggers #openChecklistWizard.click() — used by doc-checklist.html + full-doc-checklist.html
│   ├── autoopen-quiz.js      # imports openQuiz from quiz.js and calls it — used by readiness-report.html
│   ├── payment-return-checklist.js # Non-module payment return handler for doc-checklist.html + full-doc-checklist.html
│   ├── payment-return-quiz.js      # Non-module payment return handler for readiness-report.html
│   ├── pro-bypass-checklist.js     # Non-module: Pro bypass for checklist — intercepts pay button, checks Supabase pro_status, calls API directly if Pro
│   ├── pro-bypass-quiz.js          # Non-module: Pro bypass for readiness report — same pattern as pro-bypass-checklist.js
│   ├── checklist-origin-saver.js   # Non-module: saves checklist country-of-origin to sessionStorage; validates before wizard Next1 (capture phase)
│   ├── quiz-origin-saver.js        # Non-module: saves quiz country-of-origin to sessionStorage; validates before quiz Next (capture phase)
│   ├── free-quiz-origin.js         # Non-module: country-of-origin capture-phase validation for the free readiness quiz on index.html
│   ├── quiz-email-capture.js       # Non-module: post-score email capture on index.html — MutationObserver on #quizResultsScreen; injects #quiz-email-capture above #lockedReportBox; cleanup-first sessionStorage wipe; POSTs quiz_score+quiz_date to /api/newsletter/subscribe (Phase 5B)
│   ├── register-sw.js       # Service worker permanently removed — stub comment file; not loaded by any HTML page
│   ├── eb-index-waitlist.js # Non-module: wires EB waitlist email form on index.html — POSTs to /api/newsletter/subscribe with source 'eb_waitlist_homepage'; fail-open
│   ├── admin.js             # Non-module IIFE: full admin panel logic — auth, tab navigation, CRUD for all entities; API base URL hardcoded from config.js value
│   ├── eligibility.js       # Visa eligibility checker — 12 visas, 3 free uses (localStorage gate), Pro unlock via postMessage/upgradeModal
│   ├── newsletter.js        # Newsletter subscribe form handling + subscribeToDigest() + waitlistDigestHook()
│   ├── config.js            # API_URL export
│   └── utils.js             # escapeHTML and shared helpers
├── readiness-score.html     # SEO landing page — migration readiness quiz; drives to /#readiness; inline <style> + no JS
├── visa-eligibility.html    # SEO landing page — visa eligibility checker; drives to /#eligibility; inline <style> + no JS
├── document-checklist.html  # SEO landing page — document checklist; drives to /#tools; inline <style> + no JS
├── eb-immigration.html      # SEO landing page — EB-1A/EB-2 NIW early access; email waitlist IIFE posts to /api/newsletter/subscribe
├── blog/
│   ├── index.html           # Blog index — links to 5 articles; uses ../styles.css
│   ├── cost-of-living-lagos-london-toronto.html  # Article: Lagos vs London vs Toronto cost of living 2025 comparison
│   ├── express-entry-document-checklist.html     # Article: Complete Express Entry document checklist (Canada)
│   ├── uk-skilled-worker-eligibility.html  # Article: UK Skilled Worker eligibility guide (2024/25 rules)
│   ├── eb2-niw-vs-eb1a.html                # Article: EB-2 NIW vs EB-1A comparison for African professionals
│   └── nurse-salary-uk.html                # Article: UK nurse salary after tax + Nigeria comparison
├── admin.html               # Admin panel — not linked publicly; loads css/admin.css + js/admin.js; no Supabase UMD; no styles.css
├── sitemap.xml
├── robots.txt
├── stripe-api.js            # Backend Stripe route scaffold (reference only — not imported by frontend)
├── orabo-audit-report.html  # Static audit report (not a live app page)
└── CLAUDE.md
```

**Module size rule:** Every JS and CSS module must stay under 600 lines. `index.html` is exempt. Extract data or sub-sections into a sibling file when approaching the limit.

**Note:** `js/dashboard.js` still exists on disk but is no longer imported.

---

## Backend Structure (`japaconnect-backend`)

```
japaconnect-backend/
├── server.js            # Express app, route mounts
├── scheduler.js         # node-cron jobs with retry logic; includes hourly publish-notification cron (sends Brevo email when approved=true AND email_sent=false)
├── digest.js            # Weekly newsletter via Brevo
├── db/client.js         # pg Pool
├── routes/
│   ├── admin.js         # Admin CRUD routes — all protected by requireAdmin (x-admin-token header); manages opportunities, stories, scholarships, universities, post_earn_rewards, users, purchases
│   ├── stripe.js        # Checkout, portal session, webhooks
│   ├── checklist.js     # AI document checklist (Claude + ai_cache); returns { checklist, docx }
│   ├── report.js        # AI readiness report (Claude + ai_cache); returns { report, docx }; sets res.setTimeout(180000) + req.setTimeout(180000) at route top
│   ├── cv.js            # CV Review + Cover Letter — AI generation + PDF/DOCX via docgen.js
│   ├── sop.js           # SOP Writer — AI generation + PDF/DOCX via docgen.js
│   ├── docgen.js        # generatePdf() + generateDocx() — both handle HTML and markdown input via isHtml() check
│   ├── universities.js  # CRUD + pagination (DB table exists but report.js no longer queries it)
│   ├── scholarships.js  # CRUD + pagination (DB table exists but report.js no longer queries it)
│   ├── visas.js         # CRUD + pagination
│   └── opportunities.js # Community submissions — requires type, link, description, email; duplicate check by normalized link (409 { error:'duplicate' } if found)
├── utils/
│   └── constants.js     # ORIGIN_CURRENCY_MAP — 16 African countries mapped to local currency string (e.g. 'NGN (₦)')
└── scrapers/            # Cheerio/Playwright scrapers
```

---

## Feature Data Inventory

### Processing Times (`js/visatracker.js` + `js/visatracker-data.js`) — 18 visas
| Category | Visas |
|---|---|
| Study | F-1 (USA), Student Visa (UK), German Student, Australia 500, NZ Student, Portugal D4 |
| Work | UK Skilled Worker, UK Global Talent, O-1 (USA), EU Blue Card, Germany Job Seeker, EB-1A (USA), EB-2 NIW (USA) |
| PR | Express Entry (Canada), Canada PNP, Australia 190, Australia 189, NZ Skilled Migrant |

Each visa has: `id`, `label`, `country`, `flag`, `category`, `minWeeks`, `typicalWeeks` (optional), `maxWeeks`, `difficulty`, `difficultyColor`, `govUrl`, `note`, `milestones[]`. Trend data in `TREND_DATA` keyed by visa id (12 months, min/typical/max arrays). Live data from `/api/visa-tracker` (falls back to static).

### Cost of Living (`js/costliving.js`) — 33 cities
Grouped `<select>` dropdown. Data in `CITIES[]` and `COST_DATA{}` (monthly USD: rent, groceries, transport, utilities, dining). FX rates in `FX_RATES` (base USD). Comparison modal uses `compareSlots[]` (1–4 slots). `buildCityOptions(selectedId)` generates grouped `<optgroup>` options using `COUNTRY_ORDER` and `COUNTRY_FLAG_MAP` constants (also used by `buildCityDropdown()`). `compare-cities.html` uses same progressive slot pattern.

Countries: USA (5), Canada (4), UK (4), Ireland (2), Netherlands (2), Germany (3), Portugal (2), Australia (3), New Zealand (2), Nigeria (2), Kenya (1), Ghana (1), South Africa (2).

### Take-Home Pay Calculator (`js/paycalc.js` + `js/paycalc-data.js`) — 35 roles × 9 countries
`js/paycalc-data.js` exports `ROLES[]`, `ROLE_GROUPS`, `SALARY_DATA{}`. Countries: USA, Canada, UK, Germany, Netherlands, Ireland, Australia, New Zealand, Portugal. Sectors: Technology (9), Healthcare (6), Education (3), Business & Finance (8), Engineering (4), Trades (5).

`all-countries-pay.html` imports from `./js/paycalc-data.js` via `<script type="module">`. `COUNTRY_DATA` holds 9 countries with `costOfLiving` values aligned to `paycalc.js`'s `COL_MONTHLY_USD`. All event wiring via `addEventListener`.

### Currency list (standard — all dropdowns)
**USD, GBP, EUR, CAD, NGN, KES, GHS, ZAR, AUD, NZD** (this exact order). FX rates (base USD): GBP 0.79, EUR 0.92, CAD 1.37, NGN 1580, KES 129, GHS 15.5, ZAR 18.5, AUD 1.55, NZD 1.63.

### CV Review + Cover Letter Generator (`js/cvtool.js`) — one-time purchase
Three output types: `cv_review` ($18), `cv_cover_letter` ($28), `cover_letter_only` ($12). Multi-step form — `ALL_SECTIONS = CV_SECTIONS` (6 sections; section 0 "Your details" captures `user_name` + `user_email`). Standalone page `cv-tool.html` polls for `window.cvTool` then calls `init()`. Stripe returns to `cv-tool.html?payment_success=1&item=cv_review`. API response: `{ success, result, files: { cv_docx?, cl_docx? }, firstName }` — Word download only, no PDF. Backend success_url for cv items is `cv-tool.html`. No auth required.

### SOP Writer (`js/soptool.js`) — one-time purchase
Three output types: `sop_scratch` ($35), `sop_review` ($28), `sop_rewrite` ($30). Multi-step form — `ALL_SECTIONS = SOP_SECTIONS` (7 sections; section 0 "Your details" captures `user_name` + `user_email`). Same standalone + poll pattern as cvtool. API response: `{ success, result, files: { sop_docx }, firstName }` — Word download only. Backend route `/api/sop`. Backend success_url for sop items is `sop-tool.html`. No auth required.

---

## CV/SOP Tool Architecture

Both tools use an identical pattern.

### HTML structure (standalone pages)
`cv-tool.html` and `sop-tool.html` are clean standalone pages — no navbar, no footer, no Supabase UMD script. The wrapper `#cv-tool-modal` / `#sop-tool-modal` contains five screen divs: `#cv-form-screen`, `#cv-review-screen`, `#cv-payment-screen`, `#cv-loading-screen`, `#cv-download-screen` (same with `sop-` prefix). All hidden by CSS in `css/cvtool.css`. The inline non-module `<script>` polls `window.cvTool` / `window.sopTool` (100 ms interval, max 50 attempts) then calls `init()`.

### Screen visibility
`showModal()` adds `.active` to `#cv-tool-modal` → CSS rule `#cv-tool-modal.active { display: block; }` shows it. `showScreen(id)` removes `.active` from all five screen divs then adds it to the target → CSS rule `#cv-form-screen.active { display: block; }` etc. Never use `style="display:..."` on these elements.

### Form rendering
`ALL_SECTIONS = CV_SECTIONS` (6 sections) / `ALL_SECTIONS = SOP_SECTIONS` (7 sections). The question files already include "Your details" (user_name, user_email) as section 0 — do NOT prepend a separate DETAILS_SECTION. `renderSection(index)` re-renders the entire `#cv-form-screen` on every navigation, including progress header, Back button, body questions, and Next button.

**Back button (`.tool-back-btn`):** rendered into the form body above the section title on every section. Section 0: label "← Back to Home" → `window.location.href = '/'`. Section 1+: label "← Back" → `saveCurrentAnswers(index)` then `renderSection(index - 1)`. There is no separate nav-area back button — do not add one back.

**Next button (`.tool-next-btn`):** accent blue pill (self-contained styles in `css/cvtool.css`). Label is "Next →" on all sections except the last, where it reads "Review your answers →" (advances to the review screen, not directly to payment). Do NOT use `btn-primary` for this button — bare `btn-primary` lacks the `.btn` base class in this context and renders unstyled.

Supported question types: `text`, `email`, `select`, `textarea`, `multiselect`, `conditionalSub` (sub-textarea shown when select hits trigger value), `conditional` (question omitted unless parent answer includes a keyword), `fileUpload` (.txt → read into textarea), `large` (tall textarea).

### Payment return flow (sessionStorage bridge)
**Success path:**
1. User completes review screen → "Continue to payment →" → payment screen → "Select & Pay" → `sessionStorage.setItem('orabo_cv_answers', JSON.stringify(answers))` + `orabo_cv_output_type` → `purchaseItem(pkg.type, btn, { customer_email: answers.user_email })`
2. Stripe redirects to `cv-tool.html?payment_success=1&item=cv_review`
3. `init()` → `handlePaymentReturn()` checks two-part condition: `payment_success=1` in URL AND `orabo_cv_answers` in sessionStorage
4. Clears URL via `history.replaceState`, removes sessionStorage keys, shows loading screen
5. POSTs `{ outputType: item, formAnswers: parsedAnswers }` to `/api/cv` — **field name is `formAnswers`, not `answers`**
6. On success: `showDownloadScreen(data, userEmail)` renders Word download buttons from `data.files.cv_docx` / `data.files.cl_docx`

**Cancel path:**
1. User abandons Stripe checkout → Stripe redirects to `cv-tool.html?payment_cancelled=1&item=cv_review`
2. `init()` checks: if neither `payment_success=1` nor `payment_cancelled=1`, wipes sessionStorage (clean start). Otherwise preserves it.
3. `handlePaymentReturn()` detects `payment_cancelled=1` + `orabo_cv_answers` in sessionStorage → clears URL, restores answers, shows review screen
4. User can edit answers or proceed to payment again without re-filling the form

No userId passed for anonymous flow — backend skips purchases table check when userId absent. Download is Word only (DOCX) — no PDF. `generatePdf()` produces blank output for HTML input.

### API response shapes
- `/api/cv` → `{ success, result, files: { cv_docx?, cl_docx? }, firstName }` — `cv_docx` for cv_review, `cl_docx` for cover_letter_only, both for cv_cover_letter
- `/api/sop` → `{ success, result, files: { sop_docx }, firstName }`

---

## Document Checklist & Readiness Report Architecture

### Country-of-origin selection (mandatory)
- Checklist: `id="checklist-country-origin"` in `#wizardStage1`, validated before `#wizardNext1`
- Quiz: `id="report-country-origin"` in `#quizQuestionScreen`, validated before `#quizNext`

`checklist-origin-saver.js` and `quiz-origin-saver.js` use capture-phase `addEventListener('click', fn, true)`. If dropdown empty: `e.stopImmediatePropagation()` blocks module handler; `.country-select-error.visible` shows. Do NOT modify `checklist.js` or `quiz.js`.

### SessionStorage bridge
Before Stripe redirect: `checklist.js` saves `jc_checklist_state`; `quiz.js` saves `jc_quiz_state`; origin savers save `jc_checklist_origin` / `jc_quiz_origin`. After return, `payment-return-*.js` reads and clears all keys, passes `country_of_origin` in API request body. Both scripts call `fetch()` directly inside `DOMContentLoaded` — no `supabaseClient.auth.getSession()` await before the request (that await was the root cause of the API call never firing). `userId` is read from `state.userId || null`; checklist.js/quiz.js do not currently save userId in state so it passes as null. AbortController timeouts: 100 s for checklist, 180 s for report. Message rotation interval: 8 s for checklist, 15 s for report. Console prefixes: `[checklist-return]` / `[quiz-return]`.

### API response format
`/api/checklist` and `/api/report` return `{ "success": true, "checklist/report": "<html>", "docx": "<base64 string>" }`. `docx` present on both cache hits and fresh generations.

### DOCX-only download
Buttons say "Download Checklist (Word)" / "Download Report (Word)" — DOCX only, no PDF. `payment-return-*.js` wires download button in capture phase inside `.then()`, calls `e.stopImmediatePropagation()` to block module's html2pdf handler. Uses `atob()` → `Uint8Array` → `Blob` → `URL.createObjectURL` → `<a>` click. Never add PDF — `generatePdf()` produces blank output for HTML input.

### Brand sanitisation
`routes/checklist.js` and `routes/report.js` run `sanitizeBrand()` on every Claude response (cache hits and fresh): replaces "Japa Readiness", "JapaConnect", "japaconnect", "japa connect" with "Orabo"/"Orabo Readiness". Sanitised content is written to `ai_cache`. `routes/checklist.js` also strips markdown code fences from the raw Claude response before `sanitizeBrand()` and before writing to `ai_cache`: `content = content.replace(/^```html\s*/i, '').replace(/```\s*$/i, '')`.

### Currency personalisation
`country_of_origin` in request body → backend resolves local currency from `ORIGIN_CURRENCY_MAP` → injects second non-cached system block. First block (knowledge base) is cached with `cache_control: { type: 'ephemeral' }`.

### Dynamic scholarship and university generation (report only)
`routes/report.js` does NOT query `scholarships` or `universities` tables. Claude generates 6–8 scholarships and 6–8 universities from `VISA_KNOWLEDGE_BASE`. `max_tokens` for `/api/report` is 8000.

### Pro plan bypass
`js/pro-bypass-checklist.js` and `js/pro-bypass-quiz.js` intercept pay button in capture phase, check Supabase `pro_status`, call API directly if Pro. Non-module scripts loaded last. Do NOT add bypass logic to `checklist.js` or `quiz.js`.

**Pro bypass scripts read JWT from localStorage, NOT via getSession().** These scripts run inside iframes embedded in `dashboard.html`. Calling `supabaseClient.auth.getSession()` from the iframe hangs indefinitely because two GoTrueClient instances conflict (one in the parent, one in the iframe), surfacing as "Multiple GoTrueClient instances detected" in the console. Instead, use `readSessionFromStorage()` from `js/auth.js` (also inlined verbatim in each bypass IIFE), which reads the persisted Supabase session synchronously from localStorage (key `sb-<project-ref>-auth-token`). If the token is missing or expired, abort the bypass and fall through to the normal Stripe flow. Do NOT revert to `getSession()` in these scripts.

### Free quiz and docgen
Free quiz on `index.html`: country-of-origin validation in `js/free-quiz-origin.js` (non-module). `generateDocx()` checks `isHtml(content)` — do NOT pre-process checklist/report HTML to markdown before calling.

---

## Auth Flow

1. Sign Up → `signup.html` → `supabase.auth.signUp()` → verification email
2. Email link → `login.html?verified=true`
3. Login → `dashboard.html`
4. `dashboard.html` calls `supabase.auth.getUser()` — redirects to `login.html` if no session
5. `openAuthModal()` in `auth.js` redirects to `login.html`

---

## Integrations

### Supabase
- Auth (email/password)
- Email verification redirect: `https://orabo.app/login.html?verified=true`
- Password reset redirect: `https://orabo.app/login.html`
- Database tables: `user_profiles`, `purchases`, `universities`, `scholarships`, `visas`, `opportunities`, `scrape_logs`, `ai_cache`, `one_off_purchases`, `post_earn_rewards`
- `user_profiles` columns: `id`, `pro_status`, `pro_expires_at`, `stripe_customer_id`, `stripe_subscription_id`
- `one_off_purchases` columns: `id` (uuid pk), `name` (text), `email` (text), `tool` (text), `stripe_session_id` (text, nullable), `created_at` (timestamptz) — audit log for anonymous CV/SOP purchases
- `opportunities` extra columns added by migration: `status` (text, default 'pending'), `submitter_phone` (text), `rejection_reason` (text)
- `post_earn_rewards` columns: `id` (uuid pk), `email` (text unique), `phone` (text), `name` (text), `approved_count` (int, default 0), `total_paid` (int, default 0), `reward_paid` (bool, default false), `reward_paid_at` (timestamptz), `created_at`, `updated_at` — upserted on each opportunity approval; counter resets to 0 after each payout

### Stripe
- Pro subscription price ID: `price_1TO3hu2Nqh05kvDpKAmtEYjs` — stored in `STRIPE_PRO_PRICE_ID` Railway env var; backend reads `process.env.STRIPE_PRO_PRICE_ID` — never from frontend request body
- One-time items: `readiness_report` ($15), `document_checklist` ($9), `verified_badge` ($5), `consultation_express` ($55), `consultation_full` ($90), `consultation_doc_review` ($35), `cv_review` ($18), `cv_cover_letter` ($28), `cover_letter_only` ($12), `sop_scratch` ($35), `sop_review` ($28), `sop_rewrite` ($30), `eb1a_assessment` ($39), `eb2_assessment` ($35), `eb1a_letter` ($150), `eb2_letter` ($140), `eb1a_package` ($250), `eb2_package` ($240), `o1a_assessment` ($39), `o1a_letter` ($150), `o1a_package` ($250)
- **Subscription return URLs:** `dashboard.html?payment=success` / `dashboard.html?payment=cancelled`
- **One-time return URLs (cv/sop):** `cv-tool.html?payment_success=1&item=...` / `sop-tool.html?payment_success=1&item=...`
- **One-time return URLs (consultation):** `consult-booking.html?payment_success=1&item=consultation_*` — stripe.js `checkPaymentReturn` IIFE calls `_showCalendlyOverlay()`
- **One-time return URLs (other):** `index.html?payment_success=1&item=...` / `index.html?payment=cancelled`
- **Cancel return URLs (tool items):** `cv_review`, `cv_cover_letter`, `cover_letter_only` → `cv-tool.html?payment_cancelled=1&item=<key>`; `sop_scratch`, `sop_review`, `sop_rewrite` → `sop-tool.html?payment_cancelled=1&item=<key>`; `eb1a_assessment` → `eb1a-tool.html?payment_cancelled=1&item=eb1a_assessment`; `eb2_assessment` → `eb2-tool.html?payment_cancelled=1&item=eb2_assessment`; `eb1a_letter` → `eb1a-letter-tool.html?payment_cancelled=1&item=eb1a_letter`; `eb2_letter` → `eb2-letter-tool.html?payment_cancelled=1&item=eb2_letter`; `eb1a_package` → `eb1a-package-tool.html?payment_cancelled=1&item=eb1a_package`; `eb2_package` → `eb2-package-tool.html?payment_cancelled=1&item=eb2_package`; `o1a_assessment` → `o1a-tool.html?payment_cancelled=1&item=o1a_assessment`; `o1a_letter` → `o1a-letter-tool.html?payment_cancelled=1&item=o1a_letter`; `o1a_package` → `o1a-package-tool.html?payment_cancelled=1&item=o1a_package`; `document_checklist` → `doc-checklist.html?payment_cancelled=1&item=document_checklist`; `full_doc_checklist` → `full-doc-checklist.html?payment_cancelled=1&item=full_doc_checklist`; `readiness_report` → `readiness-report.html?payment_cancelled=1&item=readiness_report`; `consultation_express` / `consultation_full` / `consultation_doc_review` → `consult-booking.html?payment_cancelled=1&item=<key>`. Dashboard-originated items still cancel to `dashboard.html?payment=cancelled`; all others to `/?payment=cancelled`. Managed via `TOOL_CANCEL_PAGES` map (21 keys total) in `routes/stripe.js`.
- **Standalone page return URLs:** `readiness_report` → `readiness-report.html?payment_success=1&item=readiness_report`; `document_checklist` → `doc-checklist.html?payment_success=1&item=document_checklist`; `full_doc_checklist` → `full-doc-checklist.html?payment_success=1&item=full_doc_checklist`. Handled by `payment-return-*.js`, not `stripe.js`.
- Webhook events: `checkout.session.completed`, `customer.subscription.deleted`, `invoice.payment_succeeded`, `invoice.payment_failed`, `charge.refunded`
- `pro_expires_at` from Stripe's actual `current_period_end` — never hardcode duration
- Required Railway env vars: `STRIPE_SECRET_KEY`, `STRIPE_PRO_PRICE_ID`, `STRIPE_WEBHOOK_SECRET`, `AI_ROUTES_REQUIRE_STRIPE_SESSION`

### Brevo
- Weekly digest: Saturday 08:00 UTC (`digest.js` → `scheduler.js`)
- Subscribe: `POST /api/newsletter/subscribe` — accepts `email` (required), `destination` (optional), `name`, `source`, `quiz_score`, `quiz_date`. Only adds to list when `destination` provided. When `quiz_score` present, sets `QUIZ_SCORE`, `QUIZ_DATE`, `QUIZ_LEAD='true'` on contact. Callers without `destination` (EB waitlist, quiz capture) now succeed.
- Frontend: `js/newsletter.js` — `subscribeToDigest()` + `waitlistDigestHook()`
- Transactional emails: consultation confirmation (via `routes/stripe.js`); checklist/report delivery fire-and-forget after generation + cache hits; CV/SOP delivery fire-and-forget from `routes/cv.js` and `routes/sop.js` — email includes specific output type label + upsell paragraph (CV→SOP Writer for `cv_review`/`cover_letter_only`; SOP→Consultation for all SOP types)
- CV/SOP purchase attribute update (fire-and-forget): after each delivery, `PURCHASE_DATE` (ISO date) + `LAST_TOOL` (outputType) set on the Brevo contact via `POST /v3/contacts` with `updateEnabled: true`
- `userId` passed in request body from `supabaseClient.auth.getSession()` for checklist/report; CV/SOP use `user_email` from formAnswers (no auth required)
- Required Brevo custom attributes (create in dashboard if not present): `DESTINATION` (text), `QUIZ_SCORE` (number), `QUIZ_DATE` (text), `QUIZ_LEAD` (text), `PURCHASE_DATE` (date), `LAST_TOOL` (text)
- Env vars: `BREVO_API_KEY`, `BREVO_LIST_ID`, `BREVO_SENDER_EMAIL`, `BREVO_SENDER_NAME`
- **Email template logo:** all Brevo templates reference `https://orabo.app/orabo-icon-256x256.png` as a hotlinked PNG (`width="80" height="80"` required for Outlook). Never use `orabo-icon.svg` in email HTML — SVGs do not render in Outlook or many other clients. Inline styles are permitted in email HTML — the CSP rule applies to the web app only.

### Calendly
- Account: `calendly.com/oraboapp`
- Express (30 min): `https://calendly.com/oraboapp/30min`
- Full (1 hr): `https://calendly.com/oraboapp/full-consultation`
- Doc Review (45 min): `https://calendly.com/oraboapp/document-review`
- Flow: pay → `consult-booking.html?payment_success=1&item=consultation_*` → `stripe.js` IIFE shows `#calendlyOverlay`
- `#calendlyOverlay` in `dashboard.html`, styled in `css/consultation.css`, controlled by `_showCalendlyOverlay(url)` in `stripe.js`
- `#bookingModal` is 2 steps only (About You → Review & Pay) — never re-add date/time picker

### EmailJS
- Service ID: `service_vcvthda`; Admin template: `template_3oaoxaa`; User auto-reply: `template_yva0uva`
- Admin template (`template_3oaoxaa`) fires on hero waitlist and contact form submissions to notify the Orabo team. The user auto-reply (`template_yva0uva`) is **NOT** fired by the hero waitlist form — the canonical waitlist welcome is sent by `routes/newsletter.js` via Brevo with proper first-time-only gating and correct branding. `template_yva0uva` may still be wired to other forms (e.g. contact form) — grep before assuming it is removable elsewhere.
- Hero waitlist submit handler (`js/waitlist.js`) must NOT call `emailjs.send` with `template_yva0uva` — that path produced duplicate welcome emails through japaconnection@gmail.com Google relay alongside the Brevo send. Diagnosed via Gmail "Show original" headers and removed 2026-05-11.

### Claude API
- Model: `claude-sonnet-4-6`; all use prompt caching on knowledge base system block
- `/api/checklist` and `/api/report`: max_tokens 8000; two-block system prompt — first (knowledge base, cached) + second (per-request currency, not cached)
- `/api/cv` and `/api/sop`: max_tokens 6000; two-block system prompt — first cached (gatekeeper persona + universal rules including no-fabrication rule for CV), second uncached per-request (target-country format/SOP rules + SOP track scaffold from `utils/document-norms.js`); no `ai_cache` reads or writes — every submission is generated fresh; no userId required
- All check `ai_cache` before calling Claude; store sanitised results after

---

## Dashboard Architecture (`dashboard.html`)

| Function | Purpose |
|---|---|
| `initDashboard()` | Auth check → profile fetch → `updateDashboardForPlan()` → `handleReturnFromStripe()` |
| `updateDashboardForPlan(plan)` | Single source of truth for all plan-state visual changes. Call on load AND after payment. |
| `handleReturnFromStripe()` | Detects `?payment=success/cancelled`, waits 1.5 s, re-fetches profile, calls `updateDashboardForPlan()`. No page reload. |
| `initiateStripeCheckout()` | POSTs `{ userId, email, mode: 'subscription' }` to `/api/create-checkout-session` — no `priceId` |
| `showProModal()` / `hideProModal()` | Opens/closes `#pro-modal` |
| `showToast(message, type)` | `type`: `'success'`, `'error'`, or `'info'` |

- **Iframe close:** dashboard listens for `'tool-closed'` postMessage → calls `closeTool()`. `payment-return-*.js` wires close to `handleToolClose()` — posts `'tool-closed'` (iframe) or redirects to `dashboard.html` (standalone).
- **Pro upgrade from iframes:** post `'initiate-pro-upgrade'` to `window.top`; dashboard calls `initiateStripeCheckout()`. `wireUpgradeModal` in `stripe.js` follows same pattern.
- **Pro tool cards:** `class="pro-tool"` + `data-plan="pro"`. Tooltip via `[data-plan="pro"]::after`; hidden when `.unlocked` present.
- **`#upgrade-btn`:** hidden by default; shown by `updateDashboardForPlan('free')`, hidden by `updateDashboardForPlan('pro')`.

---

## Coding Rules

- Always commit and push to `origin main` after every code change — no need to ask
- Pure HTML/CSS/JS for the frontend — no framework
- Keep global styles in `styles.css`; feature-specific styles in `css/<feature>.css`
- Feature JS for `index.html` goes in `js/<feature>.js` (ES modules with `import/export`)
- `signup.html`, `login.html`, and `dashboard.html` use inline `<script>` blocks — do not add module imports to them
- On `signup.html`, `login.html`, and `dashboard.html` — always use `supabaseClient` (not `supabase`); `const supabase` conflicts with the UMD global `var supabase` and crashes silently
- City/role selection UIs use grouped `<select>` dropdowns (not pill buttons); extend existing data arrays — do not revert to pills
- **Cost of Living comparison city selection:** uses `compareSlots[]` state (not `compareCities`). `buildCityOptions(selectedId)` generates grouped options. Slots 2–4 progressive; slot 0 permanent. Do NOT revert to pills.
- **paycalc-data.js pattern:** `ROLES[]`, `ROLE_GROUPS`, `SALARY_DATA{}` in `js/paycalc-data.js` only. Never re-declare in `paycalc.js` or `all-countries-pay.html`. Adding a role: add to all three exports with data for all 9 countries.
- **all-countries-pay.html module script:** tool logic in `<script type="module">`; Pro gate in separate non-module `<script>` before it. Module does NOT duplicate Pro check — `return` at module top level is a SyntaxError.
- Follow existing BEM-like CSS class naming conventions
- No TypeScript — plain JavaScript only
- Do not add comments unless logic is non-obvious
- Mobile-first responsive design — check existing `@media` breakpoints before adding new ones
- Use existing CSS variables (`--navy`, `--accent`, `--gold`, etc.) — do not introduce new colour values
- **When adding a new database table:** provide `CREATE TABLE` SQL for Supabase SQL editor BEFORE pushing dependent code
- **RLS is enabled on all user-data tables.** Anon key has: read-only access to `opportunities WHERE approved = true` and `stories WHERE approved = true`; no access to anything else via direct Supabase REST. User JWT has read/write access only to their own row in `user_profiles` and read-only access to their own `purchases` rows. All other writes (one_off_purchases, post_earn_rewards, ai_cache, processed_webhook_events, opportunities, stories) go through the Express backend with the service role key, which bypasses RLS. Never add a new user-data table without also adding RLS policies in the same migration. Policy SQL canonical reference: `migrations/2026-05-13-rls-pass-2.sql` (backend repo).
- **Stripe subscription checkout:** never pass `priceId` from frontend — backend reads `process.env.STRIPE_PRO_PRICE_ID`; frontend sends `{ userId, email, mode: 'subscription' }`
- **Dashboard plan state:** always route changes through `updateDashboardForPlan(plan)` — never scatter plan-state updates elsewhere
- **CSP inline styles:** `style-src` has no `'unsafe-inline'` — never use `style="display:none"` on elements that need to start hidden; use CSS rules instead (tool modals hidden via `css/cvtool.css`)
- **CSP inline scripts:** `script-src` has no `'unsafe-inline'` — never use `onclick=""` attributes; always wire via `addEventListener` in module init IIFEs
- **New tool modal pattern:** follow cvtool/soptool pattern — CSS `.active` class for visibility (never `style="display:..."`), `showModal()` adds `.active` to modal wrapper, `showScreen(id)` toggles `.active` on child screens, sessionStorage bridge, two-part URL condition (`payment_success=1` + sessionStorage key) in `init()`
- **Dashboard iframe pages:** standalone HTML (no navbar/footer) + `autoopen-<tool>.js` + `url:` entry in dashboard TOOLS array. Never use `anchor:` for auto-opening tools.
- **Iframe close button pattern:** include `payment-return-*.js` wiring close to `handleToolClose()` — posts `'tool-closed'` (iframe) or redirects to `dashboard.html` (standalone)
- **userId in AI generation requests:** AI generation routes derive user identity from the `Authorization: Bearer <jwt>` header, never from `req.body.userId`. `utils/verify-payment.js` calls `supabase.auth.getUser(jwt)` with the service-role client to resolve `authUserId` server-side. The anonymous Stripe-session-paid flow (CV, SOP, EB, O-1A tools) works without a JWT — the Stripe path in `verify-payment.js` is independent. Pro-bypass scripts (`js/pro-bypass-checklist.js`, `js/pro-bypass-quiz.js`) read the JWT via `readSessionFromStorage()` (synchronous localStorage read — see rule below), send `access_token` in the `Authorization: Bearer` header, and abort the bypass if no session is available (letting the normal Stripe flow proceed). Do NOT pass `userId` to `/api/cv` or `/api/sop` — these are anonymous flows; recipient email uses `formAnswers.user_email`. `payment-return-checklist.js` / `payment-return-quiz.js` do NOT send any userId; email resolution is handled inside `verify-payment.js` from the trusted source (Stripe session or JWT).
- **Pro bypass scripts read JWT from localStorage, NOT via getSession().** `js/pro-bypass-checklist.js` and `js/pro-bypass-quiz.js` run inside iframes embedded in `dashboard.html`. Calling `supabaseClient.auth.getSession()` from the iframe hangs indefinitely because two GoTrueClient instances conflict (one in the parent, one in the iframe), surfacing as "Multiple GoTrueClient instances detected" in the console. Instead, use `readSessionFromStorage()` from `js/auth.js` (also inlined verbatim at the top of each bypass IIFE), which reads the persisted Supabase session synchronously from localStorage (key `sb-<project-ref>-auth-token`). If the token is missing or expired, abort the bypass and fall through to the normal Stripe flow. Do NOT revert to `getSession()` in these scripts.
- **Stripe redirects in iframes:** `purchaseItem` and `subscribeToPro` in `stripe.js` use `(window !== window.top ? window.top : window).location.href` — preserve this
- **Pro upgrade from embedded pages:** post `'initiate-pro-upgrade'` to `window.top` — never call Stripe directly from an iframe. Applies to `compare-cities.html`, `all-countries-pay.html`, `wireUpgradeModal` in `stripe.js`, `eligibility.js`.
- **Upgrade button label:** always "Upgrade to Pro" — never "Unlock with Pro" or other variants
- **Backend CORS:** `server.js` uses explicit `ALLOWED_ORIGINS` array (`https://orabo.app`, `https://www.orabo.app`) — do not revert to `FRONTEND_URL` env var
- **Consultation booking flow:** `#bookingModal` and `#calendlyOverlay` in `dashboard.html`. Modal is 2 steps only — never re-add date/time picker. `_showCalendlyOverlay(url)` in `stripe.js` shows correct event link.
- **booking.js guard pattern:** wraps all code in IIFE with `if (!modal) return` — safe to import on any page without `#bookingModal`
- **booking.js back buttons:** step 1 gets a `.tool-back-btn` "← Back to Home" inserted programmatically at the top of `#step1`; step 2's existing `.prev-step` button is removed and replaced with a `.tool-back-btn` "← Back" → `goToStep(1)`. Both wired via `addEventListener` inside the IIFE — never `onclick=""`.
- **`.tool-back-btn` in cv/sop tool forms:** plain accent-coloured text-link (`color: var(--accent)`, no border, no background, `font-size: 0.9rem`) defined in `css/cvtool.css`. Rendered above the section title by `renderSection()` on every section. Section 0: "← Back to Home" → `/`. Section 1+: "← Back" → saves current answers + `renderSection(index - 1)`. No nav-area back button exists — do not add one.
- **`.tool-next-btn` in cv/sop tool forms:** self-contained accent blue pill defined in `css/cvtool.css` (`background: var(--accent)`, `color: #fff`, `border-radius: 8px`, `padding: 0.75rem 2rem`, `font-weight: 600`). Used by `renderSection()` Next/Continue button in both `cvtool.js` and `soptool.js`. Do NOT revert to `btn-primary` for this button.
- **`.tool-landing-card .btn-primary` scoped rule:** defined in `css/cvtool.css` — makes bare `btn-primary` buttons (no `.btn` base class) self-contained with full pill styling (gradient background, 50px radius, shadow) matching `.btn.btn-primary`. Non-featured CV/SOP pricing cards on `index.html` use bare `btn-primary` — this rule is required. Do not remove it.
- **CV/SOP tools on dashboard.html:** payment return uses two-part condition (URL param + sessionStorage answers). `#cv-start-btn` calls `window.cvTool.start('cv_review')`; `#sop-start-btn` calls `window.sopTool.start('sop_scratch')`; consultation uses 3 `.consult-btn` buttons wired by `booking.js`. cvtool.js and soptool.js do NOT import from `auth.js` — they are fully auth-free.
- **one_off_purchases audit log:** `routes/cv.js`, `routes/sop.js`, `routes/eb1a.js`, and `routes/eb2.js` fire-and-forget insert into `one_off_purchases` after every successful generation using `formAnswers.user_name` and `formAnswers.user_email`. `stripe_session_id` is null for now.
- **CV and SOP no-fabrication rule:** the CV system prompt (`routes/cv.js`) forbids inventing metrics, team sizes, revenue figures, percentages, awards, certifications, dates, or job responsibilities the user did not describe. Where a metric would strengthen a bullet but the user did not provide the figure, the AI inserts `[INSERT METRIC — e.g. team size, $ figure, % improvement]` and flags every placeholder in the Top 5 Priority Actions. SOP retains its existing Rule 1 (no invented experiences). Both rules must survive any future prompt edit — do not remove them.
- **SOP track branching:** `routes/sop.js` reads `formAnswers.sopTrack`, normalises display strings to internal keys via `normalizeTrack()`, and selects an adjudicator persona + output scaffold from `SOP_TRACK_SCAFFOLDS` in `utils/document-norms.js`. Chevening produces four labelled essays (Leadership and Influence, Networking, Studying in the UK, Career Plan), not one document. MCF structures the essay around the give-back arc. Other scholarship tracks share a default scaffold with adjudicator-persona variation only. If `sopTrack` is missing or unrecognised, `normalizeTrack()` falls back to `masters_admissions` silently. The `destinationCountry` dropdown in both CV and SOP forms contains nine individual countries (USA, Canada, United Kingdom, Germany, Netherlands, Ireland, Australia, New Zealand, Portugal, Other) — not the old grouped "Europe" option. Backend country lookup: `CV_COUNTRY_RULES[formAnswers.destinationCountry] || CV_COUNTRY_RULES.DEFAULT` and `SOP_COUNTRY_RULES[formAnswers.destinationCountry] || SOP_COUNTRY_RULES.DEFAULT`.
- **EB tool routes (routes/eb1a.js, routes/eb2.js):** NO ai_cache reads — every EB generation is fresh (Tier 1, Tier 2, and Tier 3). max_tokens 8000 for Tier 1 and Tier 2; max_tokens 10000 for Tier 3 packages. Legal disclaimer included in Tier 1 AI output (appended) and in Tier 2/3 system prompt HTML structure. No `userId` check — anonymous flow only. Route branches on `outputType`: `eb1a_assessment` → Tier 1 (plain text/markdown output); `eb1a_letter` → Tier 2 (HTML output, code-fence stripping); `eb1a_package` → Tier 3 (HTML output, code-fence stripping, res.setTimeout(180000)). Same branching pattern for eb2. Brevo delivery email fire-and-forget after generation. Stripe item keys: `eb1a_assessment` ($39), `eb2_assessment` ($35), `eb1a_letter` ($150), `eb2_letter` ($140), `eb1a_package` ($250), `eb2_package` ($240). Success URLs: `eb1a-tool.html`, `eb2-tool.html`, `eb1a-letter-tool.html`, `eb2-letter-tool.html`, `eb1a-package-tool.html`, `eb2-package-tool.html`. Console prefixes: `[eb1a]`, `[eb2]`, `[brevo-eb1a-letter]`, `[brevo-eb2-letter]`, `[brevo-eb1a-letter-attr]`, `[brevo-eb2-letter-attr]`, `[brevo-eb1a-package]`, `[brevo-eb2-package]`, `[brevo-eb1a-package-attr]`, `[brevo-eb2-package-attr]`.
- **EB tool frontend (js/eb1atool.js, js/eb2tool.js):** cvtool pattern. sessionStorage keys: `orabo_eb1a_answers`/`orabo_eb1a_output_type` and `orabo_eb2_answers`/`orabo_eb2_output_type`. `init()` preserves sessionStorage when `payment_success=1` or `payment_cancelled=1` is in the URL; wipes otherwise (clean start). Loading messages rotate every 10s via `startMessageRotation()`. Single payment card per tool (no grid). Tooltip via `.ib-hint-icon` + `.ib-hint-tooltip` CSS-only hover — never onclick. Question type `'large'` renders textarea with `.large` class. Element ID prefix: `ib-q-` and `ib-err-`. Exports `window.eb1aTool` / `window.eb2Tool = { init, start }`. EB-1A answer keys include `evidence_inventory` and `letter_writers` (Section E, optional); EB-2 NIW answer key includes `evidence_and_letters` (Section D, optional). Backend defaults both to empty string if absent — a blank answer must not block submission or crash the assessment. SCREENS constant includes review screen (`eb1a-review-screen` / `eb2-review-screen`). Imports `renderReview` from `./tool-review.js`.
- **EB Tier 1 tools (`eb1a-tool.html`, `eb2-tool.html`):** identical pattern to CV/SOP — sessionStorage bridge with `orabo_eb1a_*` / `orabo_eb2_*` keys, two-part URL condition (`payment_success=1` + sessionStorage answers) in `handlePaymentReturn()`, anonymous flow with no `userId`, `formAnswers` field name in API request, DOCX-only download, no caching, attorney disclaimer appended. `js/stripe.js` `checkPaymentReturn` IIFE bypasses the generic banner for all 15 tool item keys (both `payment_success=1` and `payment_cancelled=1`) so the URL is preserved for the module's `init()` to read. Cancel return (`payment_cancelled=1`) restores answers from sessionStorage and shows review screen.
- **EB Tier 2 tools (`eb1a-letter-tool.html`, `eb2-letter-tool.html`):** same standalone page pattern as Tier 1. Polls `window.eb1aLetterTool` / `window.eb2LetterTool` (100 ms × 50 attempts) then calls `init()`. sessionStorage keys: `orabo_eb1a_letter_answers` / `orabo_eb1a_letter_output_type` and `orabo_eb2_letter_answers` / `orabo_eb2_letter_output_type`. Download file keys: `data.files.eb1a_letter_docx` and `data.files.eb2_letter_docx`. AI system prompts produce HTML output; backend strips code fences before DOCX generation. Question id prefix: `eb1a_letter_q<n>_<slug>` and `eb2_letter_q<n>_<slug>`. Mandatory email validation at section 0. No caching, no userId, DOCX-only download. SCREENS constant includes `eb1a-letter-review-screen` / `eb2-letter-review-screen`. Cancel return (`payment_cancelled=1`) restores answers from sessionStorage and shows review screen — same flow as Tier 1.
- **EB Tier 3 tools (`eb1a-package-tool.html`, `eb2-package-tool.html`):** same standalone page pattern as Tier 1 and Tier 2. Polls `window.eb1aPackageTool` / `window.eb2PackageTool` (100 ms × 50 attempts) then calls `init()`. sessionStorage keys: `orabo_eb1a_package_answers` / `orabo_eb1a_package_output_type` and `orabo_eb2_package_answers` / `orabo_eb2_package_output_type`. Download file keys: `data.files.eb1a_package_docx` and `data.files.eb2_package_docx`. Download filenames: `eb1a-full-petition-package.docx` / `eb2-niw-full-petition-package.docx`. Question id prefix: `eb1a_package_q<n>_<slug>` and `eb2_package_q<n>_<slug>` (isolated from Tier 2 prefix to avoid sessionStorage key collisions). EB-1A package: 7-section array (Section 0 + Sections A–F); 25 questions; last 3 questions (Section F) are optional inventory/timeline fields. EB-2 NIW package: 6-section array (Section 0 + Sections A–E); 23 questions; last 3 questions (Section E) are optional. SCREENS constant includes `eb1a-package-review-screen` / `eb2-package-review-screen`. `res.setTimeout(180000)` on backend handlers. `max_tokens: 10000`. No caching, no userId, DOCX-only download. Cancel return restores answers and shows review screen — same flow as Tier 1 and Tier 2.
- **O-1A route (`routes/o1a.js`):** mounted at `/api/o1a`. Branches on `outputType`: `o1a_assessment` → Tier 1 (plain text/markdown, 8-criterion scorecard, non-immigrant framing, petitioner pathway assessment from Q3 answer); `o1a_letter` → Tier 2 (HTML output, two `<h1>` sections: Petition Support Letter + mandatory Peer Consultation Letter); `o1a_package` → Tier 3 (HTML, seven `<h1>` sections including unique Petitioner Support Brief). NO ai_cache reads — all O-1A generations are fresh. Petitioner-voice Rule 1 in Tier 2/3 system prompts bans "I have", "my work", "my contributions" and all first-person pronouns when referring to the beneficiary; third-person petitioner voice required throughout. Peer organisation library (IEEE, AMA, YPO, sport governing bodies, etc.) included in Tier 2/3 system prompt to ensure credible peer consultation letter. `[INSERT: <description>]` placeholders for missing specifics (no fabrication). `max_tokens: 8000` for Tier 1 and Tier 2; `max_tokens: 10000` for Tier 3. `res.setTimeout(180000)` on all handlers. Brevo delivery + attribute update fire-and-forget after every generation. `one_off_purchases` audit row written. File keys: `o1a_assessment_docx` (Tier 1), `o1a_letter_docx` (Tier 2), `o1a_package_docx` (Tier 3). Console prefixes: `[o1a]`, `[brevo-o1a]`, `[brevo-o1a-attr]`, `[brevo-o1a-letter]`, `[brevo-o1a-letter-attr]`, `[brevo-o1a-package]`, `[brevo-o1a-package-attr]`.
- **O-1A frontend tools (`o1a-tool.html`, `o1a-letter-tool.html`, `o1a-package-tool.html`):** identical pattern to EB Tier 1/2/3 tools. Section 0 Back button on all 3 O-1A tools routes to `/o1a-visa.html` (not `/`). sessionStorage keys: `orabo_o1a_answers`/`orabo_o1a_output_type` (Tier 1), `orabo_o1a_letter_answers`/`orabo_o1a_letter_output_type` (Tier 2), `orabo_o1a_package_answers`/`orabo_o1a_package_output_type` (Tier 3). Question ID prefixes isolated across tiers: `o1a_q<n>_<slug>`, `o1a_letter_q<n>_<slug>`, `o1a_package_q<n>_<slug>`. SCREENS constant on each module includes the review screen. Download filenames: `o1a-assessment.docx`, `o1a-petition-letter.docx`, `o1a-full-petition-package.docx`. Mandatory email validation at section 0. No caching, no userId, DOCX-only download. Cancel return restores answers from sessionStorage and shows review screen.
- **css/ibtool.css:** EB + O-1A tool modal/screen visibility (`.active` pattern) for Tier 1, Tier 2, and Tier 3 ID sets, including review screen IDs (`eb1a-review-screen`, `eb2-review-screen`, `eb1a-letter-review-screen`, `eb2-letter-review-screen`, `eb1a-package-review-screen`, `eb2-package-review-screen`, `o1a-review-screen`, `o1a-letter-review-screen`, `o1a-package-review-screen`). `.tool-spinner` + `.tool-loading-inner` + `.tool-loading-msg` for loading screen. `.ib-hint-icon` + `.ib-hint-tooltip` tooltip. `.ib-payment-card` + `.ib-payment-features` + `.ib-payment-price` + `.ib-disclaimer` for payment card. When adding a new EB or O-1A tool page, extend the modal/screen ID lists in ibtool.css. Imported in `styles.css` before `mobile.css`.
- **Review screen pattern (all 11 tool modules):** A `*-review-screen` div sits between the last form section and the payment screen in each tool's HTML (`cv-review-screen`, `sop-review-screen`, `eb1a-review-screen`, `eb2-review-screen`, `eb1a-letter-review-screen`, `eb2-letter-review-screen`, `eb1a-package-review-screen`, `eb2-package-review-screen`, `o1a-review-screen`, `o1a-letter-review-screen`, `o1a-package-review-screen`). Rendered by `renderReview()` from `js/tool-review.js` via a thin wrapper in each module. Last form section Next button label: "Review your answers →" (not "Continue to payment →"). Payment screen back button targets the review screen (not the last form section). Review screen back button returns to last form section; continue button advances to payment screen.
- **js/tool-review.js (shared):** ES module exporting `renderReview({ screenId, prefix, sections, answers, onBack, onContinue })`. Builds `.tool-review-wrap` > sections > `.tool-review-item` (`.tool-review-label` + `.tool-review-value`). Empty/null/undefined answers display as `"(not provided)"` with `.tool-review-value.empty`. Action row: `.tool-review-actions` flex container with back (`.tool-back-btn`) and continue (`.tool-next-btn`) buttons. All buttons wired via `addEventListener` — never `onclick=""`.
- **css/cvtool.css review layout classes:** `.tool-review-wrap`, `.tool-review-section`, `.tool-review-item`, `.tool-review-label`, `.tool-review-value` (+ `.empty` modifier for not-provided text), `.tool-review-actions`. These are shared by CV and SOP. EB tools inherit the same layout via `ibtool.css` (which references the same class names — classes live in `cvtool.css`, which is imported globally via `styles.css`).
- **Stripe cancel URL for tool items:** `routes/stripe.js` contains a `TOOL_CANCEL_PAGES` map for the 21 tool item keys (15 CV/SOP/EB/O-1A + 6 free tools + consultation). Cancel URL format: `${FRONTEND_URL}/${toolPage}?payment_cancelled=1&item=${itemType}`. Do NOT use `/?payment=cancelled` for these items — that URL has no sessionStorage and loses the user's form answers. Dashboard items still cancel to `dashboard.html?payment=cancelled`; all others to `/?payment=cancelled`.
- **payment_cancelled=1 bypass in stripe.js:** the `checkPaymentReturn` IIFE in `js/stripe.js` preserves the URL (does nothing) when `payment_cancelled=1` is present and the item is one of the 21 tool keys. This allows the tool module's `init()` to read and handle the parameter. Do not clear the URL or show a generic banner for these items.
- **booking.js cancel return pattern:** `js/booking.js` saves `jc_booking_state` (plan, price, duration, name, email, country, destination, purpose, itemType) to sessionStorage before calling `purchaseItem()`. `js/payment-return-booking.js` (non-module, loaded by `consult-booking.html`) detects `payment_cancelled=1`, sets `window._bookingCancelReturn = true`, clears URL, then in DOMContentLoaded restores form fields and advances modal to Step 2. The inline script in `consult-booking.html` checks `window._bookingCancelReturn` and skips the default auto-open on cancel return.
- **Document Checklist back buttons:** `js/checklist.js` injects `.tool-back-btn` on Stage 2 and Stage 3 only (Stage 1 has NO back button — only the X close, to avoid blank-page navigation on the bare standalone page). Stage 2 → `checklistWizardGoTo(1)`; Stage 3 → `checklistWizardGoTo(2)`. `window.oraboChecklistRestore(state)` is a public helper that restores pathway/country state and calls `checklistWizardGoTo(3)` — used by `payment-return-checklist.js` on cancel return. Do NOT modify checklist.js for payment-return logic beyond these UI hooks.
- **Readiness Quiz back buttons:** `js/quiz.js` `renderQuestion(index)` removes any existing `#quizTopBack` and re-creates a `button#quizTopBack.tool-back-btn` at the top of `#quizQuestionScreen` only when `index > 0` (Question 1 has NO back button — only the X close). Q2–Q15 click → `currentQ--; renderQuestion(currentQ)`. `showResults()` injects a `button#quizResultsBack.tool-back-btn` as first child of `#quizResultsScreen`; clicking it hides results, shows `#quizQuestionScreen`, and re-renders Question 15 with the saved answer. `window.oraboQuizRestoreScore(state)` restores `quizAnswers` and calls `showResults()` — used by `payment-return-quiz.js` on cancel return. Do NOT modify quiz.js for payment-return logic beyond these UI hooks.
- **init() sessionStorage preservation rule (all 11 tools):** on page load, `init()` checks for both `payment_success=1` AND `payment_cancelled=1` before deciding whether to wipe sessionStorage. Wipe only when NEITHER is present (clean start). This ensures cancel returns can restore the review screen from the saved answers.
- **CV/SOP purchase verification:** `verify-payment.js` derives `authUserId` from the JWT in the Authorization header; if a JWT is present and the user has an unconsumed `purchases` row for the requested item type, it is consumed atomically. For the anonymous cv-tool.html / sop-tool.html flow (no JWT, has Stripe session), the Stripe-session path is used instead. Do not add a hard userId requirement to `/api/cv` or `/api/sop` — both support anonymous Stripe-paid flow.
- **purchaseItem() guest checkout:** `stripe.js purchaseItem()` resolves email as `bookingMeta.customer_email || bookingMeta.booking_email || user?.email || ''`. CV/SOP tools pass `{ customer_email: answers.user_email }` as the third argument. Consultation (booking.js) passes `{ booking_email }`. Backend `userId` is optional for payment mode; required for subscription mode.
- **index.html paid tool buttons navigate to standalone pages:** via `js/redirect-paid-tools.js` (non-module). `cvtool.js`/`soptool.js` NOT loaded on `index.html`.
- **Standalone tool pages (cv-tool.html, sop-tool.html, consult-booking.html):** no auth, no navbar, no footer. cv-tool.html and sop-tool.html: no Supabase UMD — non-module inline `<script>` polls `window.cvTool`/`window.sopTool` (100 ms × 50 attempts) then calls `init()`. `init()` checks for payment return first; if not a return, calls `showModal()` + `renderSection(0)`. consult-booking.html keeps Supabase UMD (needed by booking.js → stripe.js → auth.js chain) and DOMContentLoaded auto-clicks `.consult-btn` after 300 ms.
- **auth.js is safe to import without Supabase UMD:** `const { createClient } = window.supabase || {}` — if Supabase UMD is not loaded, `supabaseClient` is null, auth state listener and init IIFE are skipped, `getCurrentUser()` returns null. This is required for cvtool.js/soptool.js which import stripe.js which imports auth.js on pages that do not load Supabase.
- **Return URL security:** `login.html`/`signup.html` validate `?return=` with `/^[a-z0-9\-]+\.html$/` — only `.html` filenames; no full URLs or query strings
- **stripe.js onDashboard guard:** IIFE skips `payment=success/cancelled` when `#dash-main` is present (dashboard handles those params itself)
- **Stripe webhook `checkout.session.completed`:** uses `.update().eq('id', userId)` (not upsert) on `user_profiles`. `current_period_end` read via `sub.current_period_end ?? sub.items?.data?.[0]?.current_period_end` (Stripe API `2026-03-25.dahlia` moved this field)
- **Country-of-origin validation:** `quiz-origin-saver.js`/`checklist-origin-saver.js` use capture-phase `addEventListener('click', fn, true)`; `e.stopImmediatePropagation()` if dropdown empty; `.country-select-error.visible` shows. Do NOT add to `quiz.js`/`checklist.js`.
- **DOCX-only downloads for checklist/report pages:** capture-phase listener inside `.then()`; `e.stopImmediatePropagation()` blocks html2pdf. Never add PDF — `generatePdf()` produces blank output for HTML.
- **Brand sanitisation in AI routes:** call `sanitizeBrand()` on every Claude response in `routes/checklist.js` and `routes/report.js` — both cache hits and fresh. Write sanitised content to `ai_cache`.
- **Dynamic Claude scholarship/university generation:** do NOT add DB queries for scholarships/universities to `routes/report.js` — Claude generates from `VISA_KNOWLEDGE_BASE`
- **Opportunities submission form:** 8 required fields (`opp-field-error` inline per field): Link, Title, Organization, Country, End Date, Type, Brief Description, Your Email. Country `<select>`: USA, Canada, UK, Australia, New Zealand as flat options + `<optgroup label="Western Europe">` (19 countries). Type options include Undergraduate Scholarship and Scholarship Award. Frontend sends `{ title, organization, country, end_date, type, link, description, email }`; backend maps `organization`→`org`, `end_date`→`deadline`/`expires_at`. Duplicate check: `LOWER(TRIM(link))`; returns `{ error: 'duplicate' }` HTTP 409. `input[type="date"]` styled in `css/opportunities.css` under `.story-modal`; text/select inputs styled via global `.form-group input, .form-group select` in `css/layout.css`.
- **Community Feed filter dropdown:** The pill-button filter row was replaced with a `<select id="oppFilterSelect" class="opportunity-filter-select">` dropdown in `index.html`. Filter order: All · Undergraduate · Masters · PhD Funding · Fellowship · Scholarship · Research Grant · Postdoc · Jobs · Work Permit. Filter option `value` attributes and their exact type-mappings in `js/opportunities.js` `filterOpportunities()`: `all` → no filter; `undergraduate` → `type === 'Undergraduate Scholarship'` only; `Masters Scholarship` → `type === 'Masters Scholarship'` only (exact via default branch); `PhD Funding` → exact; `Fellowship` → exact; `scholarship` → `type === 'Scholarship Award'` only; `Research Grant` → exact; `Postdoc` → exact; `Job Opportunity` → exact; `Work Permit Program` → exact. The three scholarship-related filters (Undergraduate, Masters, Scholarship) are **mutually exclusive** — each maps to exactly one stored type and never overlaps. Do NOT change `scholarship` to a substring/contains match — that would cause cross-filter contamination. Styles in `css/opportunities.css` (`.opp-filter-row`, `.opportunity-filter-select`). `.filter-btn` styles in `css/cards.css` are retained — they are used by the visa tracker and university finder.
- **Opportunities publish notification:** `scheduler.js` runs hourly; queries `approved=true AND email_sent=false AND email != ''`; sends Brevo email; sets `email_sent=true`. The `email_sent` column was added via SQL editor — do not include in CREATE TABLE.
- **User-supplied URLs:** any URL sourced from a public submission form (opportunities, stories, universities, scholarships, user profiles) must be passed through `safeURL()` before rendering as an `<a href>`, `setAttribute('href', ...)`, or `window.open()` argument. `safeURL()` is defined inside the IIFE in `js/admin.js` and rejects everything except `http:` and `https:` schemes, returning `'#'` for invalid input. `esc()` escapes HTML metachars but does NOT sanitise the URL scheme — both are required. `js/opportunities.js` (public side) already applies the same pattern via its own `safeURL` helper.
- **Always update CLAUDE.md** after any major feature addition, architecture change, or new integration
- **Email delivery for checklist + report uses trusted email from `verify-payment.js`**, then falls back to `user_email` from request body. Frontend reads from `localStorage.orabo_user_email` (set by dashboard.html on auth, cleared on sign-out). Resolution priority: `trustedEmail` (from Stripe session `customer_email` or JWT Supabase lookup inside `verify-payment.js`) → `user_email` body field → skip send + log `[brevo-checklist] no recipient email available`. Both routes return `email_queued: boolean` in the response; frontend email-notice is gated on `json.email_queued`.
- **Every Brevo fire-and-forget call must have `.then(success log)` + `.catch(err => detailed error log)` with a `[brevo-<context>]` prefix.** No silent swallowing — Railway logs are the only way to spot delivery regressions. Contexts: `checklist`, `report`, `cv`, `cv-attr`, `sop`, `sop-attr`, `eb1a`, `eb1a-attr`, `eb2`, `eb2-attr`, `eb1a-letter`, `eb1a-letter-attr`, `eb2-letter`, `eb2-letter-attr`, `eb1a-package`, `eb1a-package-attr`, `eb2-package`, `eb2-package-attr`, `o1a`, `o1a-attr`, `o1a-letter`, `o1a-letter-attr`, `o1a-package`, `o1a-package-attr`, `consultation`, `waitlist-welcome`, `rejection`, `story-approved`, `story-rejected`, `payout`, `opp-ack`, `publish`.
- **Waitlist welcome email** is fired by `routes/newsletter.js` when `destination` is present OR `source === 'waitlist'`, AND the source is not in the suppressed-sources list, AND the Brevo contact-create returned HTTP 201 (new contact). Suppressed sources: `eb_waitlist`, `eb_waitlist_homepage`, `quiz_score_capture`, `opportunity_submission`. Repeat subscribers (existing contacts, HTTP 204) never receive a second welcome email. Gate decisions log to Railway as `[brevo-waitlist-welcome] gate: { destination, source, isNewContact, shouldFireWelcome }`. Contact creation uses `POST /v3/contacts` with `updateEnabled: true` directly. `sendBrevo` in `routes/admin.js` accepts `context` as its first argument — all callers must pass it.
- **Opportunity submission email flow:** `routes/opportunities.js` fires a fire-and-forget acknowledgement email (`[brevo-opp-ack]`) after every successful row insert — includes title, type, country, link details and a phone-conditional Post & Earn paragraph (renders only when `submitter_phone` is non-empty). `scheduler.js` hourly cron fires the "your opportunity is now live" email (`[brevo-publish]`) when `approved=true AND email_sent=false AND email != ''`; the query LEFT JOINs `post_earn_rewards` to include `approved_count` in the email. The newsletter subscribe call in `js/opportunities.js` passes `source: 'opportunity_submission'` (suppressing the welcome template) and maps the opportunity country to a valid `VALID_DESTINATIONS` value for the digest list (`UK`→`United Kingdom`, European countries→`Europe`); destination is omitted when no mapping exists so the contact is still created in Brevo without a list-add.
- **Mobile hamburger nav:** all styles in `css/hamburger.css` inside `@media (max-width: 768px)`. Toggle IIFE in `script.js`. Hamburger uses `.open` class. Do not add desktop styles to `hamburger.css`; do not duplicate toggle logic.
- **Mobile overflow fixes (`css/mobile.css`):** loaded last in `styles.css`. All rules inside `@media (max-width: 768px)`. Owns `html, body { overflow-x: hidden }`, hero container fixes, hero text scaling, stats bar wrapping, grid single-column. Do NOT touch other CSS files for mobile overflow.
- **Service worker:** permanently removed — `sw.js` and `js/register-sw.js` are stub comment files; the `<script src="js/register-sw.js">` tag is absent from `index.html`. Do not re-add a service worker.
- **PWA icons:** `icons/icon-192.png` and `icons/icon-512.png` are binary copies of `orabo-icon-256x256.png` and `orabo-icon-512x512.png` respectively. Source files live in the repo root. Replace both icons folder files whenever the source logo changes. `site.webmanifest` references these exact paths.
- **Email template logo:** all Brevo email templates reference `https://orabo.app/orabo-icon-256x256.png` as a hotlinked PNG (80×80 rendered, `width`/`height` attributes required for Outlook). Never reference `orabo-icon.svg` in emails — SVGs do not render in Outlook or many other clients. Inline styles ARE permitted in email HTML — the CSP rule applies to the web app only.
- **SEO landing pages pattern (`readiness-score.html`, `visa-eligibility.html`, `document-checklist.html`, `eb-immigration.html`):** minimal nav (logo left + "Back to Orabo" right), no footer (just copyright line), load `styles.css` only, all page-specific layout in inline `<style>` in `<head>`, CSP-safe (no `onclick=""` — all event wiring via `addEventListener` in IIFE `<script>` at bottom of `<body>`), mobile responsive below 768px. Use `.lp-nav`, `.lp-hero`, `.lp-section`, `.lp-cta-band`, `.lp-footer` class convention. CTAs link to app sections (e.g. `/#readiness`, `/#eligibility`, `/#tools`) — not to standalone tool pages.
- **`eb-immigration.html` email waitlist:** IIFE posts `{ email, source: 'eb_waitlist' }` to `/api/newsletter/subscribe`. Shows `#form-success` on both resolve and reject (fail-open). No auth required.
- **Blog articles (`blog/*.html`):** load `../styles.css` (relative), use `.article-body`, `.callout`, `.data-table`, `.cta-block`, `.blog-nav` classes. Back link points to `/blog/`. Target 900–1,200 words. End each article with a `.cta-block` linking to the relevant Orabo tool.
- **`blog/index.html`:** card grid using `.blog-card` anchors; `blog-tag`, `blog-card h2`, `blog-meta` classes. Add a new card here whenever a new blog article is created.
- **Homepage blog teaser section (`index.html`):** 3-column `.blog-teaser-grid` of `.blog-teaser-card` anchors above the EB waitlist banner. Styles in `css/eb-waitlist.css`. Update cards here whenever a new article is added to the blog.
- **Homepage EB waitlist banner (`index.html`):** dark navy `.eb-waitlist-section` above the footer. Email form `#eb-index-email` / `#eb-index-btn` / `#eb-index-success` wired by `js/eb-index-waitlist.js` — POSTs `{ email, source: 'eb_waitlist_homepage' }` to `/api/newsletter/subscribe`; fail-open (shows success on both resolve and reject). Styles in `css/eb-waitlist.css`.
- **Nav Tools dropdown links (index.html):** Blog and EB Green Card Tools entries live after the SOP Writer `<li>`, separated by a `.nav-dropdown-divider` `<div>`. EB entry uses `.nav-dropdown-coming-soon` + `.nav-coming-badge` "Soon" badge. Styles for badge in `css/nav.css`.
- **Admin panel (`admin.html`):** accessed by direct URL only — not linked from any public page. Does NOT load Supabase UMD or `styles.css`. Auth via `x-admin-token` header checked against `process.env.ADMIN_SECRET`. Token stored in `sessionStorage`. All admin API routes live under `/api/admin/` and require the token. `css/admin.css` is NOT imported into `styles.css`. `js/admin.js` is a non-module IIFE.
- **Admin opportunity approval:** sets BOTH `approved = TRUE` AND `status = 'approved'` — keeps existing scheduler cron working. Does NOT send the "post is live" email (scheduler handles it). Upserts into `post_earn_rewards` (approved_count +1, phone from submitter_phone).
- **Post & Earn payout:** `POST /api/admin/post-earn/:email/mark-paid` resets `approved_count = 0`, sets `reward_paid_at = NOW()`, increments `total_paid`, keeps `reward_paid = FALSE` so they can earn again. Sends Brevo airtime notification. Railway env vars: `POST_EARN_THRESHOLD` (default 20), `POST_EARN_REWARD_PER_POST` (default 300), `POST_EARN_PAYOUT` (default 6000).
- **Opportunities `submitter_phone` field:** collected via `id="opp_phone"` in index.html modal (optional). Passed in POST body as `phone`. Stored in `submitter_phone` column. Displayed in admin panel Phone column.
- **CV/SOP delivery emails (Phase 5A):** `_sendCvEmail` and `_sendSopEmail` in `routes/cv.js` / `routes/sop.js` include the specific output type label in subject and body. Upsell paragraph: `cv_review` and `cover_letter_only` upsell SOP Writer; all SOP types upsell Consultation. Both fire a separate fire-and-forget `POST /v3/contacts` to Brevo setting `PURCHASE_DATE` (ISO date string `YYYY-MM-DD`) + `LAST_TOOL` (outputType). `SOP_LABELS` map in `sop.js` maps `sop_scratch`→`Statement of Purpose (from scratch)`, `sop_review`→`SOP Review`, `sop_rewrite`→`SOP Rewrite`.
- **Newsletter subscribe backward compatibility:** `/api/newsletter/subscribe` no longer requires `destination`. `destination` is validated only when provided; contact is added to the list only when `destination` is present. Source-only callers (EB waitlist, quiz capture, any future non-destination subscriber) work without `destination`.
- **Quiz email capture pattern (Phase 5B):** `js/quiz-email-capture.js` is a non-module IIFE. DOMContentLoaded must wipe `orabo_quiz_email_captured` + `orabo_quiz_score_capture` from sessionStorage BEFORE reading any state (cleanup-first rule). MutationObserver watches `#quizResultsScreen` for `style` attribute changes; injects `#quiz-email-capture` above `#lockedReportBox` when results become visible. Skips injection if sessionStorage key already set. Submit POSTs `{ email, source: 'quiz_score_capture', quiz_score, quiz_date }` to Railway URL directly (not relative — GitHub Pages has no server proxy). Fail-open on both resolve and reject. "No thanks" adds `.hidden` to the block and sets `orabo_quiz_email_captured = 'skipped'`. Load AFTER `quiz.js` and `free-quiz-origin.js` in `index.html`.
- **`.hidden` utility class:** defined in `css/quiz.css` — `display: none`. Shared by quiz email capture elements. Do not define again elsewhere.
- **EmailJS legacy:** `emailjs.send` calls remain in `js/booking.js:137`, `js/opportunities.js:251`, and `js/waitlist.js:34` — all using `template_3oaoxaa` (admin notification only). `template_yva0uva` (user auto-reply) has been removed from the codebase entirely (removed 2026-05-11 — was causing duplicate welcome emails via Gmail relay alongside Brevo). EmailJS is scheduled for full removal in a future migration session — **do NOT add any new EmailJS calls anywhere in the codebase**. Use Brevo for all new email functionality.
- **Payment verification on AI routes:** All 7 AI generation routes call `verifyPayment()` from `utils/verify-payment.js` before invoking Claude. Recipients for Brevo delivery are derived from the trusted source (Stripe session `customer_email`, or Supabase auth lookup via `userId`), never from `formAnswers.user_email`. Kill switch is `AI_ROUTES_REQUIRE_STRIPE_SESSION` on Railway. Frontend tool modules must forward `stripe_session_id` from URL params in the POST body — never omit. Any new AI-generation route added in the future (additional visa suites, etc.) MUST call `verifyPayment()` before any Claude API call.

---

## Design System

**Colours:**
- `--navy: #1a3c6e`, `--navy-dark: #0f2548`, `--navy-light: #2a5298`
- `--accent: #4a90d9`, `--gold: #f5a623`
- `--bg: #f4f7fb`, `--bg-alt: #eaf0f9`, `--text-muted: #5a6b85`

**Fonts:** Inter (body), Playfair Display (decorative)

**Key utility classes:** `.btn-primary`, `.btn-gold`, `.btn-outline`, `.section-tag`, `.gradient-text`, `.filter-btn`

**Nav logo:** `.nav-logo` (flex, align-items center, gap 8px) wraps `<img src="/orabo-icon.svg">` + `<span class="nav-brand-name">Orabo</span>`. `.nav-brand-name`: Inter 700, 1.2rem, `#fff`, letter-spacing -0.01em. Both classes defined in `css/base.css`.

**Footer brand:** `.footer-brand` uses `<img src="/orabo-icon.svg" alt="Orabo">` + `<span>Orabo</span>` inside `.nav-logo` anchor — same SVG and sizing (36×36px) as the navbar logo, no globe emoji. No separate footer-logo class — inherits `.nav-logo img` styles from `css/nav.css`.

**Nav buttons (index.html):** `.nav-login-btn` (ghost, white border), `.nav-signup-btn` (gold fill), `.nav-go-pro` (gold pill) — all use `!important` in `css/auth.css`.

**Parent organisation attribution:** `orabo.app` declares `oraboss.com` as its parent organisation in footer text and JSON-LD schema. The Oraboss attribution line ("A product of Oraboss — building intelligent products at the intersection of Africa and the global economy.") appears above the copyright line in the footers of `index.html`, and `blog/index.html`. `dashboard.html` has no footer. The link uses `target="_blank" rel="noopener"` and hovers to `var(--accent)`. The `parentOrganization` and `sameAs` fields are merged into the existing `WebApplication` JSON-LD block in `index.html <head>` (not a separate schema block). CSS class `.footer-parent-org` defined in `css/footer.css`; `.oraboss-link` defined in `blog/index.html`'s inline `<style>` block.

---

## Repositories

- **Frontend:** https://github.com/Oraboss/japaconnect — deployed at `https://orabo.app`
- **Backend:** https://github.com/Oraboss/japaconnect-backend — deployed on Railway
- **Branch (both):** main
- **Git user:** Abiodun Bello
