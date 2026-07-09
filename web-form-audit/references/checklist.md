# Form review checklist

Run this against any form before shipping (or when reviewing a PR that adds/changes one). Items are ordered by leverage — the top sections move conversion most.

## Questions & content
- [ ] Every field passed Keep / Cut / Postpone / Explain — no field survives on "the database has a column for it"
- [ ] Nothing asked that can be inferred (card type from number, city/state from postal code, country from locale, data already on file)
- [ ] Field groups replaced by platform features where possible: express payment buttons (Apple Pay / Google Pay), address lookup filling an editable block, passkey/SSO instead of a password
- [ ] Optional/marketing questions moved to after completion
- [ ] No forced registration before checkout; guest path exists
- [ ] Labels are succinct, in user vocabulary; ambiguous ones rephrased as natural questions
- [ ] Form title matches the link/button that led here
- [ ] Start page present only if users need to gather documents / significant time (and it says what & how long)

## Structure & layout
- [ ] Fields grouped into conversation-order topics; minimal visual group separators (heading + space or thin rule)
- [ ] Single column; one unbroken vertical scan line from first field to primary button
- [ ] One label alignment throughout (default top-aligned; right for tight vertical space; left only to force deliberation)
- [ ] Consistent spacing (~50–75% of field height between questions)
- [ ] Tab order follows visual order; tested by tabbing through
- [ ] Checkout/registration page stripped of nav, promos, and exit links
- [ ] Multi-page: progress shows scope/position/status truthfully — or is deliberately vague/absent; save-and-resume for long forms

## Fields
- [ ] Right control per question (radio/checkbox/select/text per the decision rules)
- [ ] Field widths match expected answers where meaningful; otherwise one consistent width
- [ ] Only the minority of required/optional is marked, with text ("optional") preferred; indicator on the label; `*` has a legend
- [ ] Flexible input: all reasonable formats accepted and normalized after entry (phone, dates, card spacing); nothing rejected over formatting
- [ ] Smart defaults serve the user's majority case; no pre-checked marketing/consent (a GDPR violation, not just bad UX); no presumptuous defaults (birth date, gender)
- [ ] No placeholder-as-only-label; placeholders (if any) are format examples next to a real label
- [ ] Correct `type` / `inputmode` / `autocomplete` on every field (mobile keyboards + autofill)

## Actions
- [ ] One visually dominant primary button, left-aligned with fields, labeled with the outcome (not "Submit")
- [ ] No Reset/Clear; secondary actions absent or visually subordinate (link-styled, after primary)
- [ ] Submit disables + shows in-place progress on click; server-side duplicate guard too
- [ ] ToS handled via "By clicking … you agree" near the button, not a checkbox lost among opt-ins
- [ ] If disabled-until-valid is used: all fields required, button stays visible, missing-state explained

## Help
- [ ] Help exists only for: unfamiliar data, why-we-ask, security reassurance, format recommendations, optional flags
- [ ] Help is concise, adjacent, and visible (or focus-revealed / "?"-triggered next to the label, in a popover that doesn't shift the page)
- [ ] No paragraph of instructions above the form doing load-bearing work
- [ ] Sensitive-data fields carry verifiable reassurance

## Validation, errors, success
- [ ] Inline validation only where users can't know validity (uniqueness, strength, limits, typeahead) — not on their own name
- [ ] Validation fires on blur/completion, never on first keystrokes; live feedback only for counters/meters/suggestions
- [ ] Character limits show a live countdown
- [ ] Error state: top summary (role=alert, focused, links to fields) + double visual emphasis per field (never color alone) + fix-it instruction at the field
- [ ] Error and field styles visibly match; red + warning icons used for errors *only*, nowhere else
- [ ] User input preserved through every error round-trip; earlier answers never altered
- [ ] Success confirmed in context, non-blocking, announced (role=status), with next-step actions — no dead end

## Mobile & touch
- [ ] Touch targets ≥44×44 CSS px; full-width fields/buttons on small screens; ≥8px between adjacent targets
- [ ] Primary action reachable and never obscured by sticky bars or the keyboard
- [ ] Nothing hover-only — help, tooltips, and reveals all work on tap
- [ ] Correct keyboard opens for every field (`type`/`inputmode` verified on a device)
- [ ] Layout tested at phone width and 200% zoom — scan line and tab order survive

## Authentication (if the form signs people in/up)
- [ ] Passkey and/or SSO offered before email+password; magic link or OTP considered for infrequent-use products
- [ ] Password rules are NIST-style: length only (8+), breach-list check, no composition requirements, no forced rotation
- [ ] Paste and autofill/password managers work everywhere (`autocomplete="username" / "new-password" / "current-password" / "one-time-code"`)
- [ ] Show-password toggle present; strength meter (if any) scores length/entropy, not character classes
- [ ] Sign-in error doesn't reveal which credential failed; recovery links visible on the form
- [ ] Anti-spam is invisible-first (rate limit, honeypot, time check) — visible CAPTCHA only as last resort

## Conditional sections
- [ ] 1–3 follow-up fields → revealed inline within/below the chosen option, visually bound to it, animated, hidden branches `disabled`
- [ ] Handful of branches → vertical tabs / select + panel; big branches → separate step
- [ ] All branches never shown simultaneously (grayed-out included)
- [ ] Active choice always visible; revealed fields clearly attached to their trigger; minimal page jump

## Accessibility
- [ ] Every control has a visible `<label>`; groups have `fieldset/legend`
- [ ] Fully keyboard-operable, visible focus, no traps
- [ ] Errors/success wired with `aria-invalid` / `aria-describedby` / `role=alert|status`
- [ ] Nothing color-only; contrast ≥ 4.5:1; survives 200% zoom and long labels
- [ ] Reduced-motion respected; no flashing; timeouts extendable without data loss
- [ ] WCAG 2.2 specifics: targets ≥24px minimum, no redundant re-entry of already-given data, authentication requires no transcription or puzzle-solving

## Measure it
- [ ] Analytics on completion rate, drop-off point, per-field error rates, time-to-complete — so the next redesign argues from data
