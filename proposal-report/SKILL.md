---
name: proposal-report
description: Generate a polished, self-contained HTML proposal/recommendations report (editorial serif design with a scored recommendation matrix, TOC sidebar, and prev/next pager). Use when the user runs /proposal-report or asks to create a proposal report, recommendations report, or advisory report. Interviews the user first, then renders template.html.
argument-hint: [topic or source files]
---

# proposal-report — interview, then generate an HTML advisory report

This skill produces a single self-contained HTML file in the style of an
engineering advisory report: a serif editorial layout (Newsreader/Spectral +
Inter/JetBrains Mono via Google Fonts CDN), a masthead with colophon, an
optional disclaimer notice, an executive overview, an at-a-glance scoring
matrix, and one scored `<proposal-article>` section per recommendation. A
table-of-contents sidebar and a prev/next pager build themselves automatically
from the articles via web components — you never write TOC markup.

The template lives next to this file: `template.html` (same directory as this
SKILL.md). **Never regenerate its CSS or JavaScript from memory — always copy
the file and edit only the content placeholders.**

If the user passed an argument, treat it as the report topic and/or the source
files to draw content from, and skip any interview questions it already answers.

## Phase 1 — Interview

Goal: gather everything needed to fill the template, in 2–3 batched rounds,
without interrogating the user about things you can infer. Before asking
anything, look at the current repo/conversation for obvious answers (existing
markdown docs, project name, the user's name/company) and present inferred
values for confirmation rather than asking open-endedly.

Use `AskUserQuestion` for anything with enumerable options; questions that need
free-form answers can rely on its "Other" free-text option, or be asked in
plain conversation if they truly can't be optioned.

**Round 1 — identity & framing** (batch up to 4 questions):
- What is the report about / its working title?
- Prepared for (client) and prepared by (author/firm)? Offer inferred values.
- Scope line (e.g. "Workstream 1: Tooling and Architecture") and status
  (e.g. "v1 — Draft for review" vs. "v1 — Accepted on {date}").
- Kicker eyebrow text (default: "Engineering Advisory · Confidential").

**Round 2 — content source** (single question):
- Where does the substance come from? Typical options:
  1. Existing files (markdown notes, docs) — ask which, then read them.
  2. Interview me topic-by-topic — elicit each recommendation in conversation.
  3. You draft from what you know of this codebase/project; I'll correct.

**Round 3 — per-recommendation details.** For each recommendation, you need:
title, a one-sentence italic blurb, the body content (recommendation, rationale,
optional plan), and four scores. Rather than asking the user to score
everything, **draft the scores yourself and present the full list for
confirmation** in one compact message (one line per recommendation:
`NN Title — payoff X · conviction X · risk X · effort X`). Adjust on feedback.

**Round 4 — finishing touches** (batch):
- Include the standard liability disclaimer? (yes/no; offer to adapt the
  standard text: professional guidance, client responsible for final review,
  provider assumes no liability)
- Output path (default: `./report.html`, or `proposals/report.html` if a
  `proposals/` directory exists).
- Issued date (default: today, formatted like "17 June 2026").

## Phase 2 — Draft the content

Writing style — this matters as much as the markup:
- Succinct but fully-spelled-out prose. Complete sentences; do not pack
  meaning into fragments the reader must slowly deconstruct.
- Editorial, confident, first-person-advisor tone ("I recommend…"), concrete
  rationale with evidence, links to sources as `<sup class="ref">` footnote
  markers where claims are checkable.
- Blurbs and payoff-details are one to two sentences, italic-friendly.
- The overview (executive note) should explain how the recommendations relate
  to each other, cross-linking with `<a href="#pN">(NN)</a>`.

### Content guidelines

- **Bold key words for scanability.** Wrap the load-bearing words of each
  paragraph and bullet in `<strong>` — the decision itself, tool/product
  names, key numbers and dates — so a reader skimming only the bold text
  still gets the gist of the document. Aim for one or two bolded phrases per
  paragraph or bullet; bolding everything is the same as bolding nothing.
- **No em dashes.** Do not use em dashes (`&mdash;` or `—`) in report prose,
  including attribute text like blurbs and payoff-details. Restructure with a
  period, comma, colon, or parentheses instead.

Score vocabulary (the only valid values — they drive the dot scales):
`very low` (1) · `low` (2) · `low-medium` (2.5) · `medium` (3) ·
`medium-high` (3.5) · `high` (4) · `very high` (5).
Payoff & conviction: more is better. Risk & effort: more is greater cost.

## Phase 3 — Generate

1. Copy `template.html` from this skill's directory to the output path.
2. Fill every `{{PLACEHOLDER}}`. The placeholders are:
   - Head/masthead: `{{TITLE}}`, `{{PREPARED_FOR_SHORT}}`, `{{KICKER}}`,
     `{{LEDE}}`, `{{PREPARED_FOR}}`, `{{PREPARED_BY}}`, `{{ISSUED_DATE}}`,
     `{{SCOPE}}`, `{{STATUS}}`, `{{ORB_STATE}}`. Pick `{{ORB_STATE}}` from
     the status automatically: `pending` (pulsing amber orb) for draft /
     in-review / awaiting-feedback statuses, `accepted` (static green orb)
     for accepted / approved / final statuses. Delete the `status-orb` span
     entirely only if no status signal makes sense.
   - Overview: `{{OVERVIEW_HEADING}}`, `{{OVERVIEW_PARAGRAPH_1}}`,
     `{{OVERVIEW_PARAGRAPH_2}}` (add/remove `<p>`s as needed).
   - Notice: `{{DISCLAIMER_TEXT}}` — or delete the whole notice `<section>`
     if the user declined it.
   - Footer: `{{FOOTER_RIGHT}}` (e.g. "Confidential · Firm Name").
3. Expand the two repeating blocks (marked `══ REPEATING BLOCK ══` in
   comments): one matrix `<tr>` **and** one `<proposal-article>` per
   recommendation, numbered 1..N in the same order. Rules:
   - `id="pN"` must equal `"p" + num`; the matrix row links `href="#pN"`.
   - The four score attributes on each `<proposal-article>` must exactly
     match its matrix row's `<scale-dots value="…">` values.
   - `title`, `blurb`, `payoff-detail` are plain-text attributes: escape with
     entities (`&amp;`, `&rsquo;`, `&ldquo;`, `&rdquo;`), never raw `<` or
     unescaped `&`. Remember: no em dashes (see Content guidelines).
   - Body markup: use the building blocks cataloged in the template comment
     (`h3`, `label-head`, tip blockquotes, `pre`/`code` with `.kw/.st/.cm`
     highlight spans, `table.data`, `figure.diagram` with inline SVG,
     `sup.ref` footnote links). Follow the "Rationale" / "Suggested plan"
     label-head pattern by default, but let content dictate structure.
   - Remove the instructional template comments from the final file.
4. Do not touch the `<style>` blocks, `<template>` elements, or `<script>` —
   they are the design system and components, copied verbatim.
5. No external resources other than the Google Fonts `<link>`s: images and
   diagrams must be inline SVG.

## Phase 4 — Verify

- `grep -n '{{' output.html` must return nothing.
- Count matrix `<tr>` rows in `<tbody>` vs. `<proposal-article>` elements —
  they must be equal, with matching numbers, titles, and scores.
- Confirm every `href="#pN"` has a matching `id="pN"`.
- Offer to open the file (`open output.html` on macOS) for the user to review,
  and mention the TOC docks open on viewports ≥1400px wide and that the report
  is print-ready (`⌘P` → each section avoids page breaks).
