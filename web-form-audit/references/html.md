# Implementing the patterns in modern HTML/JS

The book's principles predate HTML5; this file maps them onto current platform features. Rule of thumb: **use the platform first** — native inputs, constraint validation, `autocomplete` — then layer JS for timing, messaging, and normalization. Semantic markup gets you keyboard support, screen-reader support, mobile keyboards, and browser/password-manager autofill for free.

## Base markup: label, group, order

```html
<form method="post" action="/signup" novalidate>
  <fieldset>
    <legend>About you</legend>

    <div class="field">
      <label for="email">Email</label>
      <p id="email-help" class="help">We'll only use this to sign you in. No spam, ever.</p>
      <input id="email" name="email" type="email" autocomplete="email"
             required aria-describedby="email-help" />
    </div>
  </fieldset>

  <button type="submit">Create my account</button>
</form>
```

- Every control gets a `<label for>` (or wraps the control) — screen-reader name *and* bigger click target. Radios/checkboxes especially.
- Related fields → `<fieldset>` + `<legend>` (address block, radio groups, multi-part questions). For a multiple-choice question, `<legend>` is the question; `<label>` is each option.
- `novalidate` on the form when you implement your own messaging (browser bubbles are unstylable and show one error at a time); keep the constraint attributes (`required`, `type`, `minlength`, `pattern`) — they still power `checkValidity()` and convey semantics.
- DOM order = visual order = tab order. Single column; don't use positive `tabindex`.
- Top-aligned labels: label block, input block below, `margin-bottom` between fields ≈ 50–75% of input height.
- Never put the primary question in `placeholder`. Placeholder is for format examples only, alongside a visible label — and even then an always-visible hint line is safer.
- Touch sizing: inputs and buttons ≥44px tall (`min-height: 44px`), full-width on small screens, ≥8px gaps between adjacent targets (WCAG 2.2 floors: 24×24px targets; sticky bars must never obscure the focused field).

## Input types, keyboards, autofill

Correct `type` / `inputmode` / `autocomplete` cuts typing effort and error rates dramatically — this is the modern form of "minimize the pain":

| Data | Markup |
|---|---|
| Email | `type="email" autocomplete="email"` |
| Phone | `type="tel" autocomplete="tel"` (accept any format — see normalization) |
| Name | `type="text" autocomplete="name"` (prefer one full-name field over given/family when you don't truly need parts) |
| Postal code | `autocomplete="postal-code" inputmode="numeric"` (text type — codes have letters in many countries), `size`/width matched to local length |
| Address | `autocomplete="street-address"` / `address-line1..2`, `address-level2` (city), `address-level1` (state), `country` |
| Card | `autocomplete="cc-number" inputmode="numeric"`, plus `cc-name`, `cc-exp`, `cc-csc` |
| New password | `type="password" autocomplete="new-password"` (+ let password managers generate) |
| Existing password | `autocomplete="current-password"` |
| One-time code | `autocomplete="one-time-code" inputmode="numeric"` |
| Date of birth | `bday` — or three explicit fields; avoid a date picker for far-past dates |
| Quantity, small int | `type="number" min max` or radios for tiny ranges |
| Search/large enumerable sets | text + `<datalist>` or an ARIA combobox for typeahead suggestions |

Field-length affordances: set width via CSS (`ch` units work well: `input[name=zip]{width:7ch}`), one consistent default width for everything without a natural length.

## Flexible input + normalization (after entry, never during)

```js
// Accept (555) 123-4444, 555.123.4444, 5551234444, +1 555 123 4444 …
phone.addEventListener('blur', () => {
  const digits = phone.value.replace(/\D/g, '').replace(/^1(?=\d{10}$)/, '');
  if (digits.length === 10) {
    phone.value = `(${digits.slice(0,3)}) ${digits.slice(3,6)}-${digits.slice(6)}`;
    clearError(phone);
  }
});
```

Same idea for card numbers (strip spaces/dashes; detect brand from IIN prefix and *show* it instead of asking), dates (accept several separators), and names (never reject characters real names contain). The machine adapts to the human, not vice versa.

## Validation timing: "reward early, punish late"

Validate a field when the user *finishes* it (`blur`), not per keystroke; once a field has been marked invalid, you may re-check on `input` so the error clears the moment they fix it:

```js
function validateField(el) {
  const ok = el.checkValidity() && customChecks(el);
  setFieldError(el, ok ? null : messageFor(el));
  return ok;
}

form.addEventListener('focusout', e => {
  if (e.target.matches('input, select, textarea') && e.target.value !== '')
    validateField(e.target);
});
form.addEventListener('input', e => {
  if (e.target.getAttribute('aria-invalid') === 'true') validateField(e.target);
});
```

Live-as-you-type feedback is reserved for: character-count limits (update every keystroke, zero lag), password strength meters, and suggestion/typeahead. Async uniqueness checks (username, email-already-registered) run on blur, debounced, with a pending indicator.

CSS now encodes the blur-timing rule natively — `:user-invalid` only matches after the user has interacted with the field, unlike `:invalid` which flags empty required fields on page load:

```css
input:user-invalid { border-color: var(--error); }
input:user-valid.needs-confirmation { border-color: var(--ok); } /* only where confirmation earns its keep */
```

Use `:user-invalid` for the visual channel; you still need JS for messages, summaries, and ARIA state.

## Accessible error display (top summary + double emphasis in place)

```html
<!-- Top-level summary: inserted on failed submit, focus moved to it -->
<div id="error-summary" class="error-summary" role="alert" tabindex="-1" hidden>
  <h2>We couldn't create your account</h2>
  <ul><!-- <li><a href="#email">Enter a valid email address</a></li> --></ul>
</div>

<!-- Per-field: red label + adjacent instruction + aria wiring -->
<div class="field field--invalid">
  <label for="email">Email</label>
  <input id="email" name="email" type="email" aria-invalid="true"
         aria-describedby="email-error email-help" />
  <p id="email-error" class="error-text">
    <svg aria-hidden="true" class="icon-error">…</svg>
    Enter an email address like name@example.com.
  </p>
</div>
```

```js
form.addEventListener('submit', e => {
  const bad = [...form.elements].filter(el => el.willValidate && !validateField(el));
  if (bad.length) {
    e.preventDefault();
    renderSummary(bad);                 // list of links; clicking focuses the field
    summary.hidden = false;
    summary.focus();                    // announce + orient keyboard/SR users
  }
});
```

Rules encoded above:
- Summary at top: what happened + linked list of every offending field. `role="alert"` + programmatic focus announces it.
- Each bad field: `aria-invalid="true"`, error text tied via `aria-describedby`, and **two** visual channels (colored label/border *and* icon + text) — never color alone.
- Error text says how to fix, sits where the fix happens, and persists until fixed (no toast, no modal).
- User input is never cleared on a failed submit; server round-trips must re-render every entered value (except passwords).
- Reserve the red/error styling exclusively for errors across the whole app.
- Short forms (1–3 fields) may drop the summary; keep the in-place emphasis.

## Submit progress and duplicate prevention

```js
form.addEventListener('submit', () => {
  if (!form.checkValidity()) return;
  const btn = form.querySelector('button[type=submit]');
  btn.disabled = true;
  btn.setAttribute('aria-disabled', 'true');
  btn.innerHTML = '<span class="spinner" aria-hidden="true"></span> Creating your account…';
  // On async failure: restore label, re-enable, show error summary.
});
```

- Disable *after* the click, never gate the whole form behind disabled-until-valid unless every field is required and checkable — and if you do, the button stays visible (dim, not hidden) with an explanation of what's missing.
- Long operations (uploads): show per-operation progress alongside the button state.
- Idempotency: also guard server-side (token) — users double-submit via history/refresh too.

## Actions block

```html
<div class="actions">
  <button type="submit" class="btn-primary">Place order · $43.20</button>
  <a href="/cart" class="btn-quiet">Back to cart</a>
</div>
```

- Primary: a real `<button type=submit>`, filled/high-contrast, **left-aligned with the fields** (in LTR), first in DOM.
- Secondary (only if unavoidable): link-styled, after the primary. No `type=reset`, ever.
- ToS: `<p class="legal">By clicking "Create account", you agree to the <a>Terms</a>.</p>` directly above the button — no checkbox unless legal insists on explicit action, in which case make it visually unmistakable and not adjacent to marketing opt-ins (which are never pre-checked).

## Authentication fields

Order of preference: passkey → SSO → magic link / OTP → password. Only then does the password markup matter:

```html
<!-- Sign-in: let the platform offer a passkey via conditional UI -->
<input id="user" name="user" type="email"
       autocomplete="username webauthn" />
<!-- navigator.credentials.get({ mediation: 'conditional', publicKey: … }) -->

<!-- Password field done right (NIST 800-63B + WCAG 2.2 Accessible Authentication) -->
<div class="field">
  <label for="pw">Password</label>
  <p id="pw-help" class="help">At least 8 characters. Longer is stronger — a phrase works well.</p>
  <input id="pw" name="pw" type="password" minlength="8"
         autocomplete="new-password" aria-describedby="pw-help" />
  <button type="button" aria-pressed="false" data-toggle-visibility="pw">Show</button>
</div>
```

- Never block paste, never disable autofill, never add `onpaste="return false"` — password managers are the security feature.
- No composition rules (`pattern` demanding uppercase/symbol mixes). Enforce length + a breached-password check (e.g., k-anonymity lookup against Pwned Passwords) on submit.
- If showing a strength meter, score length/entropy/breach status; never show a "must contain…" rule checklist.
- OTP: `<input inputmode="numeric" autocomplete="one-time-code">` gets SMS/email codes autofilled on supporting platforms. One field, not six boxes, unless you handle paste-across-boxes and backspace perfectly.
- Sign-in errors: "email or password is incorrect" (don't leak which), with visible recovery links — recovery is part of the form.

## Express paths and anti-spam

- **Checkout**: render Apple Pay / Google Pay buttons (via Payment Request API or your PSP's elements) *above* the manual card form — they replace card + contact + address entry in one sheet. Manual entry stays as fallback.
- **Address**: progressive-enhancement lookup — typeahead or postcode search that *fills* the conventional, still-editable address block (`autocomplete` tokens intact). Always keep a "enter address manually" path; lookup services miss new and rural addresses.
- **Anti-spam**, cheapest first: server-side rate limiting → honeypot → time-check → invisible risk-scored challenge (Turnstile/reCAPTCHA v3) → visible challenge only as last resort:

```html
<!-- Honeypot: hidden from humans, tempting to bots. Reject any submission that fills it. -->
<div class="hp" aria-hidden="true">
  <label for="website">Website</label>
  <input id="website" name="website" type="text" tabindex="-1" autocomplete="off" />
</div>
<style>.hp{position:absolute;left:-9999px}</style>
```

Also record a server-issued timestamp/token at render and reject sub-second submissions.

## Conditional sections (selection-dependent inputs)

Small follow-up (1–3 fields), expose-within pattern:

```html
<fieldset>
  <legend>Shipping method</legend>
  <label><input type="radio" name="ship" value="standard" checked> Standard — free</label>
  <label><input type="radio" name="ship" value="pickup"> Store pickup</label>
  <div class="follow-up" data-for="pickup" hidden>
    <label for="store">Choose a store</label>
    <select id="store" name="store">…</select>
  </div>
</fieldset>
```

```js
fieldset.addEventListener('change', () => {
  fieldset.querySelectorAll('.follow-up').forEach(d => {
    const active = fieldset.querySelector('input:checked').value === d.dataset.for;
    d.hidden = !active;
    d.querySelectorAll('input,select,textarea').forEach(el => el.disabled = !active);
  });
});
```

- `disabled` on hidden branch controls: they're skipped by validation and not submitted.
- Visually bind the revealed block to its trigger (indent + shared background/border); animate reveal (`transition` on height/opacity, respect `prefers-reduced-motion`).
- Larger branches: vertical-tab layout (radio list + adjacent panel) or a `<select>` + one grouped panel. Very large: separate step. Never render all branches at once, grayed or not.
- `<details>/<summary>` works for user-optional extras ("Add a gift message"); label the summary with exactly what it adds.

## Success

- Confirm in context: message on/near the thing created (`role="status"` so it's announced), highlight-and-fade for in-list additions, dismissible banner otherwise. Never a blocking modal.
- Include next actions (view order, add another, invite) — no dead-end "Thanks" page.
- Post-completion is the right moment for the optional questions you postponed in design step 1.

## Smart defaults

- Preselect majority-serving options (`checked`, `selected`); infer country/locale from request context; derive city/state from postal code (keep the fields editable).
- Persist returning users' prior choices (server-side profile or `localStorage`) as sticky defaults.
- Never pre-check consent/marketing. No default where guessing is presumptuous.

## Accessibility checklist for the implementation

- [ ] Every control has an accessible name from a visible `<label>` (hidden labels only for visually-implied fields like address line 2).
- [ ] Groups use `fieldset/legend`; the question is in the legend for choice groups.
- [ ] Logical DOM/tab order; visible `:focus-visible` styles; no keyboard traps; enter submits.
- [ ] Errors: `aria-invalid`, `aria-describedby`, focused `role="alert"` summary; success uses `role="status"`.
- [ ] Nothing conveyed by color alone; text contrast ≥ 4.5:1; layout survives 200% zoom and long localized labels (top-aligned labels help).
- [ ] Help text programmatically associated (`aria-describedby`), not floating nearby.
- [ ] Custom widgets (comboboxes, date pickers) keyboard-operable with a plain-input fallback; motion respects `prefers-reduced-motion`.
- [ ] Session timeouts warn and extend without losing entered data.
- [ ] WCAG 2.2 form criteria met — targets, redundant entry, accessible authentication, focus not obscured (full list with explanations: patterns.md, accessibility section).
