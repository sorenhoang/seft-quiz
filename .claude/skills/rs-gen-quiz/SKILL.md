---
name: rs-gen-quiz
description: Use when the user runs /rs-gen-quiz or asks to research a topic and turn it into a self-study quiz, mind-map quiz, flashcard/Q&A set, or interactive multiple-choice quiz covering that topic from beginner to deep-dive/internals level, optionally sized small/medium/large/extra-large.
allowed_tools:
  - WebSearch
  - WebFetch
  - Read
  - Write
  - Edit
  - Glob
---

# rs-gen-quiz

Research a topic, then ship a self-contained interactive HTML quiz — same engine as `quizzes/*.html` in this repo. Each topic gets two tabs at the top: **Multiple Choice** (click an option → instantly reveals correct/wrong on the spot, locks that question, scored, localStorage progress, end-of-section review with explanations) and **Q&A** (open question, answer hidden behind a toggle — self-review, not scored).

## When to Use

- `/rs-gen-quiz <topic> [size]` — e.g. `/rs-gen-quiz Kubernetes networking`, `/rs-gen-quiz React Server Components large`, `/rs-gen-quiz distributed consensus extra-large`
- User wants to deep-dive a topic and test recall across skill levels, not just skim it
- User asks for a bigger/smaller question bank, or names a size (small/medium/large/extra-large) directly

## Process

### 1. Scope
Pull the topic from the command args. If it's vague ("databases"), narrow it yourself to something concrete rather than stopping to ask — pick the most likely scope and note the assumption in your final summary.

### 2. Pick the Bank Size
Read the size off the command args (`small` / `medium` / `large` / `extra-large`). **Default: `medium`** if none given. If the user names something that isn't one of the four, map it to the nearest tier and note the assumption in your final report — don't stop to ask.

Size scales **questions per section only** — not section count. Section count is fixed by how the topic actually decomposes (step 3); a topic doesn't grow more sub-topics just because you asked for a bigger bank.

| Size | MC questions / section | QA items / section |
|---|---|---|
| Small | 10 | 10 |
| **Medium (default)** | 20 | 20 |
| Large | 30 | 30 |
| Extra-Large *(hard/broad topics)* | 60 | 60 |

These are exact counts per section, not ranges — every section in the quiz gets the same count for its size tier. Spread that count roughly evenly across the three difficulty tiers from step 4 (e.g. at 30/section: ~10 beginner, ~10 intermediate, ~10 deep-dive) rather than front-loading easy ones. Reach for **Extra-Large** when the topic is genuinely broad or deep (e.g. "distributed systems", "Kubernetes internals") — not just because the user said "large" casually; at 60 questions × 2 formats per section, a thin topic runs out of real distinct angles fast (see Gotchas).

### 3. Research
Web search the topic like a mind-map: find its natural sub-topics (the way `datadog-quiz.html` breaks "Datadog" into Agent, Tags, Metrics, Logs, APM, etc). For each sub-topic pull from primary sources (official docs, RFCs/specs, source code comments) over blog restatements. Let the topic's own structure set the section count — usually 4–12 sections depending on breadth — fewer, deeper sections beat many shallow ones, and this doesn't change with the size tier from step 2.

At Medium and above, one search per section won't surface enough distinct material to fill it honestly — run several searches per section (config reference, common pitfalls, internals/how-it-works, comparisons/trade-offs) so step 4 has 20–60 genuinely different angles to draw from, not one shallow pass stretched thin.

Cluster the sections into **groups** as you go — a group is a coherent sub-area of the topic (e.g. for "Datadog": a core-product group plus a cross-cutting "Shared Knowledge" group for SRE concepts that aren't Datadog-specific). Groups are an **in-file organizing label, not separate files or tabs** — see step 6. Most topics land on 2–4 groups; a narrow topic can legitimately have just one.

### 4. Write Questions (beginner → deep-dive, per section, in BOTH formats)
Within each section, order questions along a difficulty arc — don't randomize:
1. **Beginner** — what it is / why it exists
2. **Intermediate** — how it's configured or used in practice, common gotchas
3. **Deep-dive** — internals, edge cases, "what happens when X fails", trade-offs vs alternatives

Each section needs two parallel question sets covering that same arc, each holding exactly the count from the step 2 table for the chosen size:
- **`mc`** (recognition) — multiple-choice questions. Every question needs a real distractor (a wrong answer someone would plausibly pick), not three throwaway options. `explain` must teach the mechanism, not restate the answer.
- **`qa`** (active recall) — open questions with a free-text `a`. These are harder on purpose: phrase them so skimming the section title doesn't answer them ("what happens when X fails" beats "what is X").

Don't just translate each MC question into a QA one — MC tests recognition, QA tests recall, so the two sets earn their keep by probing the topic differently, not by being the same facts twice. At Large/Extra-Large sizes this matters more, not less: the extra count must come from genuinely new sub-questions (more edge cases, more failure modes, more trade-offs), never from padding with near-duplicate rewordings.

**Format every `a` as more than one flat sentence.** A wall-of-text paragraph is hard to scan on the QA reveal card — write it as a short lead-in sentence (optional) followed by `* ` bullet lines, one distinct point per line, the way you'd actually explain it out loud. The engine's `mdToHtml()` renders `* `-prefixed lines as `<li>`, blank-line-separated text as `<p>`, and inline `**bold**` / `*italic*` / `` `code` `` — use bold for the key term the question is testing, `code` for literal config keys/env vars/commands, italic sparingly for genuine emphasis. Example:
```
a: "Datadog Watchdog is an **AI anomaly-detection engine** built into APM, Infra, and Logs.\n* Runs continuously, no manual monitor setup required\n* Surfaces `Insights` (informational) separately from `Alerts` (actionable)\n* Root-cause analysis follows a 3-part model: *what*, *where*, *why*"
```

### 5. Order Sections (beginner → deep-dive, across the whole topic)
Question-level ordering (step 4) isn't the only arc — the sections themselves need one too. Order the whole `SECTIONS` array — groups included — so foundational/prerequisite sections come first and internals-heavy ones come last (e.g. "Agent" before "APM sampling internals"), within each group and across groups. A reader working top-to-bottom through the one file should be climbing a difficulty ramp, not bouncing between levels.

### 6. Assemble the Quiz — ONE HTML file for the whole topic
**One topic = one file, always** — `quizzes/<topic-slug>.html`, copied from `template.html` (in this skill's folder). Never split a topic across multiple files or add extra top-level tabs beyond the template's built-in Multiple Choice / Q&A ones. Every section from every group goes in the same `SECTIONS` array, in step 5's order; a section's `group` field is just an in-file label the home screen uses to print a heading between clusters of cards — it does not change the file structure. Fill in:
- `{{TITLE}}`, `{{SUBTITLE}}`, `{{ICON}}` (one emoji)
- `{{SLUG}}` — the topic slug (used as the localStorage key; unique per topic, see Gotchas)
- The `SECTIONS` array — each entry: `{id, group, title, icon, mc:[{q, options, answer, explain}], qa:[{q, a}]}`

`answer` (mc only) is the 0-based index into that question's own `options` array, in the order you wrote them — the engine shuffles at render time and remaps the index itself. Never pre-shuffle or hand-compute a shuffled index. `qa` has no `answer`/`options` — just `q` and `a`, and it isn't shuffled or scored.

Don't touch anything below the `ENGINE` comment in the template — it's generic and already handles rendering, scoring, tab switching, and progress storage.

### 7. Register in the Home Index — AND sync every quiz file's sidebar
The left sidebar (categories → topics) is on **every page**, including inside each quiz file itself — not just `index.html`. That means this step touches more than one file:

1. Open `index.html` (project root). Pick the one `MANIFEST` category (`Backend`, `Infrastructure`, `AI`, `System Design`, `Behavioral` — the fixed set from `CLAUDE.md`) that best fits the topic — don't invent a new category. Push one topic entry into that category's `topics` array: `{title, icon, file:"quizzes/<topic-slug>.html", sections:<total count>}`. If the topic already has an entry (a later run adding sections to the same file), update its `sections` count instead of adding a second entry.
2. Copy that same, now-updated `MANIFEST` array into **every** `quizzes/*.html` file's `SIDEBAR_MANIFEST` — the one you just created and every other quiz file that already exists — with one adjustment: strip the `quizzes/` prefix from each `file` path (sidebar links between quiz files are relative to the `quizzes/` folder itself, e.g. `"quizzes/datadog.html"` in `index.html` becomes `"datadog.html"` in every `SIDEBAR_MANIFEST`). A small `node -e` script reading/rewriting the `SIDEBAR_MANIFEST` block across all files is faster and less error-prone than hand-editing each one.

Don't touch anything below the `ENGINE` comment in `index.html`, or below `SIDEBAR_MANIFEST`'s own `renderSidebar()` IIFE in a quiz file.

Note the terminology doesn't nest the way it sounds: `MANIFEST`/`SIDEBAR_MANIFEST` **category** (index-level, e.g. "Infrastructure") is unrelated to a section's own `group` field (in-file, e.g. "Shared Knowledge" vs the core group) from step 3 — one topic, one category, but as many in-file groups as the research naturally produced.

### 8. Report
State: topic-scope assumption from step 1 if you had to narrow it, the size tier used (and whether it was requested, defaulted, or capped down per step 2), the file path, section count per group (in difficulty order), total MC/QA question counts, and which index category it was filed under.

## Gotchas

- **All text fields (`q`, `options`, `explain`, and `qa`'s `a`) go through `innerHTML`, not `textContent`.** Raw `<`, `>`, `&` from research content (generics like `List<T>`, comparisons like `a < b`, `&&`) will break rendering or vanish. Escape them (`&lt;`, `&gt;`, `&amp;`) before writing into the array.
- **`a`'s markdown only applies to `qa`, not `mc`'s `options`/`q`/`explain`.** Only `mdToHtml()` (used on `a`) parses `* `/`**`/`` ` ``; those characters anywhere else render as literal asterisks/backticks — don't write `**bold**` into an MC option or question text expecting it to format.
- **`* ` at the start of a line means "bullet," so genuine content starting with a literal asterisk or a markdown-looking dash-list needs care** — a stray leading `* ` or `- ` on a line you didn't mean as a bullet silently becomes a `<li>`. Newlines inside `a` must be real `\n` characters, not the literal two-character sequence `\n`.
- **Every section needs both `mc` and `qa` arrays populated.** The home screen branches on the active tab and reads `s.mc.length` / `s.qa.length` directly — an empty or missing array renders an empty/broken card in that tab, not a hidden one.
- **`answer` is an index into your own unshuffled `options` array, per-question.** The engine's `shuffledQuestions()` does the shuffling and index remapping at quiz-start time — if you pre-shuffle or reorder options after picking `answer`, the index will point at the wrong option.
- **`id` must be a unique kebab-case slug within the file** — it's the per-section localStorage key inside that topic's progress bucket. Reusing an id across sections (even across different groups in the same file) silently merges their saved progress.
- **`{{SLUG}}` must be unique per topic.** It's the whole file's localStorage namespace (`quiz_progress_{{SLUG}}_v1`). Two different topics sharing a slug share and overwrite one progress bucket.
- **Don't create `quizzes/<topic-slug>/` as a folder with multiple files inside it — one topic is one flat file, `quizzes/<topic-slug>.html`.** If you catch yourself about to split a big topic into several HTML files or add a second top-level tab beyond Multiple Choice/Q&A, stop — that's not this skill's shape; more breadth means more `group` clusters and more sections in the one file, not more files.
- **A section that's just definitions isn't deep-dive.** If every question in a section is "what is X", you stopped at beginner — go back and add the internals/trade-off/failure-mode questions before calling a section done.
- **Distractors from real confusions beat random wrong answers.** Pull wrong options from things practitioners actually mix up (adjacent commands, similar-sounding configs, the previous version's behavior) — found during research, not invented.
- **`index.html`'s `MANIFEST` (and every quiz file's `SIDEBAR_MANIFEST`) is inline JS, not fetched JSON** — same reason as everything else here: the page needs to work when double-clicked (`file://`), where `fetch()` of a local file silently fails. Don't refactor it into a separate `.json` the page fetches, and don't try to have quiz files `fetch()` `index.html`'s copy either — it's the same constraint either direction.
- **Registering a new topic without updating every existing file's `SIDEBAR_MANIFEST` leaves their sidebars stale** — they'll still render, just missing the newest topic (or showing a stale `sections` count) until the next sync. Do the multi-file sync in step 7 every time, not just on the file you're currently generating.
- **Quiz-file sidebar paths have no `quizzes/` prefix; `index.html`'s do.** Copy-pasting `MANIFEST` straight into `SIDEBAR_MANIFEST` without stripping the prefix produces links like `quizzes/datadog.html` from inside `quizzes/`, which resolves to a nonexistent `quizzes/quizzes/datadog.html`.
- **A category not in the fixed 5 (`Backend`/`Infrastructure`/`AI`/`System Design`/`Behavioral`) needs a `CLAUDE.md` update first**, not a new ad-hoc `MANIFEST` entry — keep the two in sync.
- **Don't hit a Large/Extra-Large per-section count by shrinking research depth per question.** If the research doesn't actually support that many distinct, non-redundant angles on a section, that section ships below the tier's count — fewer, real questions beats padding with rewordings. Note any section you had to cap in the report.
- **Size never changes section count.** If a size tier tempts you to split one real section into two just to hit a bigger-sounding total, or merge two into one to hit a smaller total, that's the wrong knob — section count comes only from step 3's research, independent of step 2's size.
