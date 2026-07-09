---
name: web-form-audit
description: "Design and build web forms that convert and reduce user fatigue: signup, checkout, registration, settings, data entry. Use when creating or reviewing HTML/JS forms, choosing layout or label alignment, designing validation, error handling, or form accessibility, deciding which fields to ask, or debugging form abandonment/conversion. Covers mobile-first layout, passkeys/SSO, NIST password rules, WCAG 2.2, GDPR consent, autofill, and express payment."
---

# Web Form Design

Forms stand between users and what they want and between businesses and their goals; nobody likes filling them in. Redesigns can routinely lift completion 10–40%; simple changes, like "Continue" instead of a forced "Register" button, have been [worth $300M/year](https://articles.centercentre.com/three_hund_million_button/) in certain applications / user contexts.

Design thoughtfully and identify areas to test experimentation.

**Four principles** (every decision below serves one):

1. **Minimize the pain** — remove questions, smart defaults, flexible input, helpful validation.
2. **Illuminate a path to completion** — one clear vertical scan line from first field to primary action.
3. **Consider the context** — familiar vs. unfamiliar data, one-time vs. repeated use, casual vs. high-stakes.
4. **Ensure consistent communication** — one voice; a coherent, minimal system of help, error, and success messages.

Most form questions end in "it depends", this skill helps guide some decision rules. Rationale and research data: `references/patterns.md` (Most of the source research is from 2008 - though it is evergreen and not much of the base recommendations have changed. For the things that have, see "What changed since 2008" also in `references/patterns.md` for the modernization: mobile, passkeys/SSO, NIST, WCAG 2.2, GDPR, platform features). Implementation markup and JS: `references/html.md`. Pre-ship review: `references/checklist.md`.

## Workflow

Work through these in order when designing; use the checklist when reviewing.

### 1. Question every question

Each field has a cognitive cost: parse → formulate → enter effort; the fastest field is the one you delete. Apply **Keep / Cut / Postpone / Explain** (Caroline Jarrett) to every field:

- **Keep** — user expects it and it's needed now (shipping address for a physical order).
- **Cut** — not needed now, or inferable: card *type* from the number, city/state from postal code (prefill, keep editable), country from locale, anything already on file. The biggest modern cut replaces whole field groups: Apple Pay / Google Pay instead of card+address entry, address lookup instead of a typed block, passkeys/SSO instead of a password (see Authentication). Data minimization is also a GDPR obligation.
- **Postpone** — ask after completion; optional questions asked *after* signup got ~40% more answers. Never gate checkout behind registration — guest first, account offered after.
- **Explain** — if a sensitive question must stay, add one line of why-we-ask with a user benefit.

New-user flows: consider **gradual engagement** — let people use the product first, collect identity only when needed to save/share. Splitting signup fields across pages is not gradual engagement.

### 2. Organize as a conversation, not a database dump

- Group fields into topics ("About you", "Shipping", "Payment"), ordered as a person would tell you.
- **Few short topics → one page. Long distinct sections → multiple pages. Many questions on one topic → one long page.** For high-stakes transactional forms used mostly on phones, GOV.UK's "one thing per page" measurably beats long pages.
- Separate groups with the *minimum* visual means: heading + whitespace, thin rule, or subtle background. No boxes-in-boxes, row stripes, or heavy borders. Non-informative visuals interrupt scanning.
- Known fixed sequence → progress indicator showing **scope**, **position**, **status**. Branching steps → vague high-level indicator or none; a progress bar that lies ("step 2 of 3" becoming 6) is worse than no bar.
- Long forms: save-and-resume or autosave.
- Title the form to match the link that led to it. Start page only when users must gather documents or invest real time (say what to have ready and how long).
- Critical forms (checkout, registration): strip navigation, promos, every link off the path.

### 3. Layout: one clear scan line

- **Default: single column, top-aligned labels, primary button left-aligned under the last field.** Top-aligned was fastest in eye-tracking (~50ms label→input vs 500ms for left-aligned), gained >10% completion in live tests, and tolerates long/localized labels.
- **Right-aligned labels** only for dense desktop tooling with real vertical constraints (~170–240ms; ragged left edge hurts scanning); they are mostly obsolete on mobile.
- **Left-aligned labels** when you *want* deliberation or label scanning: unfamiliar data, advanced settings. Slowest, but sometimes that's an intentional design decision.
- Never mix alignments in one form; prefer one convention per app.
- Avoid multi-column layouts (they wreck scan line and tab order); one informative row (City / State / ZIP) is fine.
- Vertical rhythm: 50–75% of field height between questions.
- DOM order = visual order = tab order; never fix with `tabindex`.

### 4. Fields

- Controls: radio (1 of 2–5, preselect any majority default), checkbox (0+ / yes-no), text (free answers). `<select>` is the control of last resort: radios/segmented controls beat it for ≤5 options, typeahead comboboxes for big lists (states, countries). Avoid multi-select list boxes.
- **Field length as affordance**: size ZIP/phone/CVC to their answers; one consistent width for the rest — arbitrary widths make people second-guess their answers.
- **Required vs optional**: eliminate optional fields first; then mark only the minority, with the word "(optional)"/"(required)" on the *label*. If you use `*`, add a legend. Mark nothing when everything is required and that's obvious.
- **Flexible input**: accept every reasonable format (phone, dates, card spacing) and normalize *after* entry — never bounce a human for formatting a machine can fix; never reformat mid-typing.
- **Smart defaults**: preselect the majority-serving choice (country from locale, standard shipping). Defaults are highly sticky. Use sparingluy, also noting that pre-ticked consent is illegal under GDPR in some cases. No default where guessing is presumptuous (birth date, gender). Returning users should get their prior choices.
- Labels: succinct, user-oriented vocabulary; natural language when terse is ambiguous ("Which bank issued the card?" beats "Issuing Bank"). Do not use an input's placeholder attribute to serve as a field's label. (Placeholders should only be used extremely sparingly, e.g. to show format expectations like MM/DD/YYYY - although helper text below the field might be a better and stricter option since it persists).

### 5. Actions

- **One primary action**, visually dominant, left-aligned with the fields — centered/right-floated buttons interrupt flow and cost users ~6s; a Cancel styled like Submit and placed first was mis-clicked by 26% of testers.
- **Avoid secondary actions** (Reset above all). If unavoidable, style them subordinate (link vs button). Wizards: prominent Continue, quiet Back.
- Button label = present the outcome ("Pay $43.20"), not a generic "Submit".
- On submit: disable the button and show in-place progress; never a "don't click twice" warning.
- Terms: "By clicking Create account, you agree to the Terms" above the button — not a checkbox lost among opt-ins.
- Disabled-until-valid only when every field is required and checkable; the button stays visible.

### 6. Help text

- Needing lots of help text means the questions are wrong — fix those first.
- Concise help only for: unfamiliar data, why-we-ask, security reassurance, recommended formats, optional flags. Best: always visible, adjacent to the field.
- Many fields needing help → reveal on focus, or a "?" *next to the label* opening a popover that doesn't shift the page. Heavy help content or expert-reused forms → toggleable help panel that follows focus.
- Sensitive data: verifiable reassurance, not vague promises.

### 7. Validation and errors

- **Inline-validate only what users can't know is right**: username availability, password strength, coupon codes, character limits (live countdown). Green-checking someone's own name is noise.
- **Validate on `blur`/completion, never on first keystroke.** Typeahead suggestions are the exception — they help *while* typing.
- An error is the most important thing on the page. Structure: (1) high-contrast top message — red + icon — listing linked offending fields; (2) **double visual emphasis** on each bad field, never color alone, styled to match the top message; (3) fix-it instruction at the field — never in a dismissable modal. Short forms may deliberately drop (1) or (2); long forms need both (bad fields sit off-screen).
- **Red and warning icons are reserved for errors**, app-wide.
- Never erase or alter the user's entries in an error round-trip.
- Only two legitimate errors: invalid input blocking progress, or system failure (apologize, offer an alternate path).

### 8. Success and next steps

- Confirm success in the context of what was done; never block further action (dismissible or auto-fading).
- **No dead ends**: pair confirmation with relevant next actions (view order, invite another, continue).

### 9. Conditional and expanding sections

An initial choice reveals follow-up fields (payment method, new vs returning customer). Decision ladder (full study data: patterns.md):

- **1–3 follow-up fields** → expose inline within/below the selected option (fastest tested).
- **A handful of branches** → vertical or horizontal tabs; make the active selection unmistakable (users fear the other tabs also submit).
- **>4–5 initial options** → drop-down + one grouped panel below it.
- **Large branches** → separate page/step.
- **Never** show all branches at once, grayed-out or not — worst on every metric tested.

Always: keep the active choice visible, visually bind revealed fields to their trigger, hide irrelevant controls, minimize page jump. Optional extras → clearly-worded "Add …" links with a remove option; large pickers → overlays that don't cover the field they fill and show chosen values after closing.

### 10. Accessibility is not optional

Every input gets a real `<label>`; related fields in `<fieldset>/<legend>`; full keyboard operability with visible focus and logical order; never color as the sole signal; errors announced (`aria-invalid`, `aria-describedby`, focus management); zoom-safe; no flashing; extendable timeouts. Target WCAG 2.2 — its new form criteria: 24px minimum targets, no redundant re-entry, accessible authentication (details in patterns.md; code in html.md).

## Authentication: the form you shouldn't build

- Offer **passkeys** (WebAuthn) and/or **SSO** before any email+password form; for infrequent-use products, **magic links** or **one-time codes** (`autocomplete="one-time-code"`) beat passwords outright.
- Password fields follow NIST 800-63B, which reversed the old rules: length (8+) not composition classes, breached-password checks, paste allowed, show-password toggle, no forced rotation. Strength meters score length/entropy, never "1 uppercase + 1 symbol". Markup and code: html.md.
- Sign-in: one identifier field, `autocomplete="username"`/`"current-password"` so password managers work; always a visible recovery path.
- The $300M lesson generalizes: never gate a task behind account creation it doesn't need.

## Mobile and touch (assume most users are on a phone)

- Touch targets ≥44×44 CSS px (WCAG 2.2's 24px is a floor); full-width fields and buttons; primary action reachable by thumb.
- Single column, top-aligned labels — the mobile viewport makes the desktop-era alternatives moot.
- Nothing hover-dependent: help, tooltips, and reveals must work on tap.
- Correct `type`/`inputmode` for the right keyboard; `autocomplete` so autofill types instead of the user; consider camera card-scan and address lookup.
- Anti-spam: honeypots, time checks, and invisible risk-scored challenges before any visible CAPTCHA — every visible puzzle costs conversions.

## Anti-pattern quick list

Reject these on sight: Reset/Clear buttons · forced registration before checkout · placeholder-as-only-label · progress bars that misstate step counts · red text for anything but errors · validation firing on first keystroke · rejecting valid data over formatting · pre-checked marketing opt-ins · asterisks on every field of an all-required form (or asterisks meaning "optional") · "do not click submit twice" · error text in a dismissable alert/modal · multi-column inputs with jumping tab order · alternating row background stripes · mixing label alignments in one form · success page with nothing to do next · secondary action styled identically to (or placed before) the primary · blocking paste in password/OTP fields · password composition rules ("1 uppercase, 1 symbol…") · hover-only help on a touch product · a visible CAPTCHA where a honeypot would do · binary-gender radio as a required field · positive `tabindex`.
