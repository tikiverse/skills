---
name: status-report
description: Generate a polished, self-contained HTML status/health report (editorial serif design with a RAG portfolio matrix, per-initiative milestone timelines, decision callouts, TOC sidebar, and prev/next pager). Use when the user runs /status-report or asks for a status report, health report, portfolio update, or exec-facing progress report. Interviews the user first, then renders template.html.
argument-hint: [topic or source files]
---

# status-report — interview, then generate an HTML status/health report

This skill produces a single self-contained HTML file in the style of an
executive portfolio status report: a serif editorial layout (Newsreader +
Spectral base64-embedded; Inter from Google Fonts with clean system fallback),
a masthead with colophon, a portfolio pulse strip (on-track / at-risk /
off-track counts), a one-paragraph verdict, an overview note, a status matrix,
and one `<status-article>` section per initiative with health pill, trend
arrow, progress bar, optional alert callout, and a milestone timeline. A
table-of-contents sidebar (with an Overview link back to the matrix) and a
prev/next pager build themselves automatically from the articles via web
components — you never write TOC markup.

It is the sibling of the `proposal-report` skill and shares its design
language. Where proposal-report scores *recommendations* (payoff/conviction/
risk/effort), status-report tracks *initiatives* (health/trend/progress) and
separates what needs the reader's sign-off from what the team is handling.

The template lives next to this file: `template.html` (same directory as this
SKILL.md). **Never regenerate its CSS or JavaScript from memory — always copy
the file and edit only the content placeholders.**

If the user passed an argument, treat it as the report topic and/or the source
files to draw content from, and skip any interview questions it already answers.

## Phase 1 — Interview

Goal: gather everything needed to fill the template, in 2–3 batched rounds,
without interrogating the user about things you can infer. Before asking, look
at the repo/conversation for obvious answers and present inferred values for
confirmation rather than asking open-endedly. Use `AskUserQuestion` for
anything with enumerable options.

**Round 1 — identity & framing** (batch up to 4 questions):
- What is the report about / its working title (e.g. "Engineering & Product
  Health: Q3 FY26")?
- Prepared for and prepared by (name + role each)? Offer inferred values.
- Period covered (e.g. "1 Apr – 30 Jun 2026") and issue date (default: today).
- Kicker eyebrow text (default: "Portfolio Status · Confidential").

**Round 2 — content source** (single question):
- Where does the substance come from? Typical options:
  1. Existing files (notes, standup docs, trackers) — ask which, then read them.
  2. Interview me initiative-by-initiative.
  3. You draft from what you know of this project/codebase; I'll correct.

**Round 3 — per-initiative details.** For each initiative you need: title,
track (eyebrow), owner, target (display text + ISO date if dated), health,
trend, progress %, an optional alert (decision or watch + detail), a
one-sentence blurb, milestones, risks, next steps, and any steering decisions
already made this period (for the Decision record). Rather than asking the
user to rate everything, **draft health/trend/progress yourself and present
the full list for confirmation** in one compact message (one line per
initiative: `NN Title — health · trend · NN% · [decision|watch|—]`). Adjust on
feedback.

**Round 4 — finishing touches** (batch):
- Output path (default: `./status-report.html`).
- Footer right text (default: "Confidential · Internal distribution only").

## Phase 2 — Draft the content

Writing style — this matters as much as the markup:
- Succinct but fully-spelled-out prose. Complete sentences; do not pack
  meaning into fragments the reader must slowly deconstruct.
- First-person-owner tone ("I need a decision by 14 Jul…"), concrete dates,
  numbers, and named owners throughout.

### Content guidelines

- **Bold key words for scanability.** Wrap the load-bearing words of each
  body paragraph and bullet in `<strong>`: dates and deadlines, headline
  metrics (before→after numbers), dollar figures, vendor/tool names, and the
  actual asks ("budget sign-off", "no executive input needed"). One or two
  bolded phrases per paragraph or bullet — a reader skimming only the bold
  text should still get the gist. Bolding everything is the same as bolding
  nothing. In `alert-detail` attributes inline `<strong>` is allowed (it is
  injected as HTML); in `title`/`blurb` it is not.
- **No em dashes.** Do not use em dashes (`&mdash;` or `—`) anywhere in the
  report prose, including attribute text. Restructure with a period, comma,
  colon, or parentheses instead.
- **Health and trend are independent axes — vary them honestly.** Do not let
  trend collapse into a restatement of health (everything off-track declining,
  everything on-track improving). The interesting rows are the off-diagonal
  ones: *at risk but improving* (the fix is working), *on track but declining*
  (green that hides an early warning). Every trend arrow must be justified by
  a sentence in that initiative's body copy, not just asserted.
- **Decisions vs. watch items.** `alert-level="decision"` (red) is only for
  items that need the *reader's* sign-off, each with an explicit ask, cost,
  and deadline. `alert-level="watch"` (amber) is for known issues the team is
  already handling. Don't inflate watch items into decisions.
- **Counts must agree.** The lede, the "Decisions needed" colophon links, the
  pulse-chip numbers, the verdict, and the number of decision-level articles
  all state the same facts; check them against each other after drafting.
- **Order by attention priority**, not alphabetically: decisions first, then
  watch items, then healthy initiatives. Say so at the end of the verdict.
- **Time deltas are computed, not written.** The `<time-delta>` component
  renders "in 3 weeks" / "in 3 mo" from the ISO date, pinned to the issue
  date. Months deliberately render as **"mo"** (not "months") so month deltas
  differ from week deltas by *shape*, making them scannable at a glance
  rather than requiring a read; do not "fix" this to the full word, and do
  not hand-write delta text in content.

- **Decision record.** Where a steering decision was made during the period,
  add a "Decision record" list to that initiative (same muted label-head + `<ul>`
  pattern as Next steps, placed between Risks and Next steps): what was chosen,
  over what alternative, and the trade-off accepted. Prefix each bullet with a
  date chip when the date matters:
  `<li><span class="dr-date">28 May</span>Chose <strong>X</strong> over Y: trade-off…</li>`.
  This is a record of decisions *made*, past tense; asks still open belong in a
  decision alert, not here.
- **Paused initiatives.** Use `health="paused"` for work deliberately halted
  (deprioritized, blocked on an external dependency, waiting on another team's
  quarter). It renders a muted gray pill with a media-pause icon (two vertical
  bars) instead of a status orb, in the pill, the TOC, and the legend. Paused
  usually pairs with `trend="flat"`; include the paused pulse chip only when
  the count is non-zero, and say *why* it is paused and what resumes it in the
  body. Do not use paused as a euphemism for off-track.

Attribute vocabulary (the only valid values):
health `on-track | at-risk | off-track | paused` · trend `up | flat | down` ·
alert-level `decision | watch` (or omit) · progress `0–100`.

## Phase 3 — Generate

1. Copy `template.html` from this skill's directory to the output path.
2. Fill every `{{PLACEHOLDER}}`:
   - Head/masthead: `{{TITLE}}`, `{{KICKER}}`, `{{LEDE}}`, `{{PREPARED_FOR}}`,
     `{{PREPARED_BY}}`, `{{PERIOD_COVERED}}`, `{{ISSUED_DATE}}` (display, e.g.
     "7 Jul 2026") and `{{ISSUED_DATE_ISO}}` (YYYY-MM-DD — pins every
     time-delta; appears once in the script and once in the delta tooltip).
   - Decisions-needed colophon field: bare zero-padded links (`01 · 02`), no
     count prefix. Delete the whole field if there are no decisions.
   - Pulse: `{{N_ON_TRACK}}`, `{{N_AT_RISK}}`, `{{N_OFF_TRACK}}`,
     `{{PULSE_VERDICT_HTML}}` (inline `<a href="#sN">`/`<strong>` allowed).
     The `{{N_PAUSED}}` chip is optional — delete it unless some initiative
     is paused.
   - Overview: `{{OVERVIEW_HEADING}}`, `{{OVERVIEW_PARAGRAPH_1}}`,
     `{{OVERVIEW_PARAGRAPH_2}}` (add/remove `<p>`s as needed).
   - Footer: `{{FOOTER_RIGHT}}`.
3. Expand the two repeating blocks (marked `══ REPEATING BLOCK ══`): one
   matrix `<tr>` **and** one `<status-article>` per initiative, numbered 1..N
   in the same order. Rules:
   - `id="sN"` must equal `"s" + num`; the matrix row links `href="#sN"`.
   - health, trend, and progress on each `<status-article>` must exactly
     match its matrix row (pill value, arrow value, bar width & percentage).
   - The matrix bar-fill class is the health's tone: `good` (on-track) /
     `watch` (at-risk) / `bad` (off-track).
   - For undated/continuous targets, omit `target-date` and the matrix cell's
     `&middot; <time-delta …>` fragment.
   - Omit `alert-level`/`alert-detail` entirely for initiatives with no alert.
   - The Decision record block in the article skeleton is optional — delete
     it for initiatives with no steering decision this period.
   - Milestone timeline states: `done` (node ✓ `&#10003;`), `current`,
     `upcoming`, `slipped` (node `&#33;`, old date as
     `<span class="was">30 Jun</span> &rarr; 20 Jul`).
   - Remove the instructional template comments from the final file.
4. Do not touch the `<style>` blocks, `<template>` elements, or `<script>` —
   they are the design system and components, copied verbatim (including the
   embedded fonts).
5. The only external resource is the Inter Google Fonts `<link>` (falls back
   to system sans offline); everything else must stay inline.

## Phase 4 — Verify

- `grep -n '{{' output.html` must return nothing.
- Count matrix `<tr>` rows in the first `<tbody>` vs. `<status-article>`
  elements — equal, with matching numbers, titles, health, trend, progress.
- Confirm every `href="#sN"` has a matching `id="sN"`, and the decisions
  colophon links point at the decision-level articles.
- Cross-check the counts: pulse chips sum to the number of initiatives; the
  lede/verdict decision count equals the number of `alert-level="decision"`
  articles.
- `grep -n '—' output.html` on the content region must return nothing (em
  dashes are banned; the embedded font data is exempt but should not contain
  one anyway).
- Offer to open the file (`open output.html` on macOS). Mention the TOC docks
  open on viewports ≥1400px and the report is print-ready (each initiative
  avoids page breaks).
