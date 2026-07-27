# TMAY Interview Playbook

**Full-Time MBA Program · David Eccles School of Business**

A self-guided, browser-based coaching tool for mastering the "Tell Me About Yourself" (TMAY) question that opens most corporate, consulting, tech, and leadership development interviews. Students learn the 4-Part Blueprint, practice aloud with live speech analytics, get a 0–100 score from an AI coach against a weighted rubric, and track their progress over time on a saved-history chart.

**Single HTML file. No build step, no server, no accounts.** One external dependency: Chart.js via pinned CDN (progress chart only — the tool degrades gracefully without it).

---

## **▶ Live tool:** **[Intv Playbook – TMAY (vC)](https://coryjburk.github.io/tmay/)**

---

## Part 1 — User Manual (Students)

### The four-step workflow (= the four tabs)

1. **TMAY Guide** — learn the 4-Part Blueprint the AI coach scores against, delivery pro-tips, and three model answers (Career Switcher, Fast Tracker/LDP, Tech PM Explorer). References, not scripts to recite.
2. **Record & Prompt** — optionally draft in the Script Builder, pick your **Interview Track** (colors your chart), then press **▶ Start Audio Practice** and deliver your answer aloud. Natural pauses are fine — the recorder runs through silence until you stop it. Then **Assemble AI Evaluation Prompt** → **Copy Coaching Prompt**.
3. **Paste Feedback** — paste the prompt into **one** AI tool (Claude, ChatGPT, or Gemini — pick one and stick with it; different tools score differently and consistency is what makes your chart meaningful). Copy the AI's full response and paste it back. The score auto-fills from the `SCORE: NN/100` line; edit if needed; **Save Feedback**. The "Saving against" chip shows exactly which run (date · track · words) your feedback attaches to.
4. **Saved History** — the progress chart appears after two scored runs, with points and segments colored by track. Below it, every entry: score badge, transcript, full feedback, delivery metrics.

### How scoring works

| Blueprint part | Points |
|---|---|
| 1. Opening Gratitude & Target Hook | 10 |
| 2. The Strong Headline (Through-line) | 30 |
| 3. The Highlight Reel | 30 |
| 4. Future-Focused Alignment | 30 |

Delivery mechanics (duration, WPM, fillers) are **coached but not scored** — they come back as unscored "Delivery Notes," so scores stay comparable whether you recorded aloud or typed. The rubric instructs the AI to grade strictly: **most first attempts land 55–75, and 90+ is reserved for near-flawless answers.** A 65 is a normal starting point, not a failure. The trend line matters more than any single number.

### Delivery targets (live metrics)

Duration **60–90 seconds** · Pace **130–150 WPM** (red >160, gold <110) · Fillers **0** (*um, uh, like, so, you know, basically* — flags every occurrence including legitimate uses; a review signal, not an absolute score).

### Managing your history

| Action | Effect |
|---|---|
| Delete this entry | Removes one transcript + feedback. **Chart point stays.** |
| Clear Progress Chart | Removes chart points. **Saved feedback stays.** |
| Clear All History | Removes everything. |
| Export / Import (JSON) | Backup and merge with automatic duplicate-skipping. |

History lives **only in this browser on this device** (~5 MB; a storage meter warns as it fills). No cloud backup by design — export regularly.

### No microphone / unsupported browser

The transcript area becomes editable — type or paste your script. Duration/WPM are marked "not recorded" so the AI evaluates content only. (Firefox always; Safari sometimes.)

### Privacy

The page transmits nothing — no server, no analytics. Speech-to-text uses the browser's built-in recognition service, which in some browsers (notably Chrome) processes audio on the vendor's servers; use typed input for anything confidential. Content pasted into an external AI assistant is governed by that assistant's terms.

### Troubleshooting

| Problem | Fix |
|---|---|
| "Speech API Unavailable" | Use Chrome/Edge, or type your script directly. |
| Mic permission denied | Allow via address-bar site settings, reload. |
| Chart not showing | Needs 2+ scored runs and an internet connection for the chart library. Scores still listed below. |
| Score dropped despite a better run | Cross-AI variance + strict rubric. Same AI every time; read the feedback, not just the number. |
| "Could not save — storage is full" | Export a backup, delete old entries, retry. |
| History gone on a new computer | Per-browser storage. Use Export/Import. |
| Copy button fails | Clipboard is often blocked on `file://` pages. Copy manually or use the hosted URL. |

---

## Part 2 — Operational Guide (Administrators / Maintainers)

### Deployment

The tool is one file. Publish via GitHub Pages: commit as `index.html` on the default branch of `tmay-playbook_vc`, enable **Settings → Pages → Deploy from a branch** (root), and share `https://<org>.github.io/tmay-playbook_vc/`. Edits via GitHub's web editor go live on the next Pages build. Distribute the hosted URL, not the raw file — clipboard and microphone permissions behave more reliably over HTTPS.

### Dependency

Chart.js is loaded from a **pinned** cdnjs URL (`Chart.js/4.4.1/chart.umd.min.js`). If the CDN is unreachable, `renderChart` detects `typeof Chart === 'undefined'` and shows a plain-text notice; all other functionality is unaffected. To upgrade, change the version in the `<script src>` tag and re-test the chart.

### Data model

Two independent localStorage stores (namespaced to avoid collisions with other suite tools):

- **`tmayHistory`** — full records, newest first: `{ dateISO, date, track, transcript, feedback, score, duration, wpm, fillers, measured, rubricVersion }`
- **`tmayScoreLog`** — lightweight chart points, oldest first: `{ dateISO, date, score, track, rubricVersion }`

Separate stores are what make "delete an entry but keep its chart point" and "clear the chart but keep feedback" possible. Every record is stamped with `RUBRIC_VERSION` (`TMAY-rubric-v1.0`) so scores remain interpretable if the rubric changes later. Export bundles are `{ tool: "eccles-tmay-playbook", version: 1, exportedAt, history, scoreLog }`; import accepts bundles (or, defensively, a plain entry array), merges with dedup keys built from `dateISO` + transcript prefix + score, and re-sorts both stores.

### Where to edit what (search strings in `index.html`)

| To change… | Find… |
|---|---|
| Rubric weights, anchors, output format | `const macroPrompt` inside `generateAiPrompt` — also bump `RUBRIC_VERSION` |
| Score auto-extraction pattern | `autoExtractScore` (regex expects `SCORE: NN/100`) |
| Interview tracks & chart colors | `const trackColors` and the `#track-select` options (keep them in sync) |
| Model answers / add an archetype | `id="tpl-*"` cards; new button `onclick="switchArchetype('<name>', this)"` |
| Filler word list | `const fillerWordsList` (also the regex in `pushToSandbox`) |
| WPM / duration thresholds | `updateTimerAndMetrics`, `evaluatePacingAndMetrics` (160/110, 90/45) |
| Storage warning thresholds | `updateStorageMeter` (60% / 85%) and `STORAGE_BUDGET_BYTES` |
| Branding / titles | `<title>`, `.logo-text`, prompt persona line in `macroPrompt`, `.site-footer` |
| Theme colors | CSS custom properties in `:root` (`--red: #CC0000`, etc.) |

### Architecture notes

- **Stale-snapshot prevention:** `pendingRun` freezes date/track/transcript/metrics at prompt-assembly time; `saveFeedback` saves that snapshot, not the live sandbox state. (Lesson imported from the Networking Pitch Coach build, where save-time snapshots were a bug.)
- **Silence handling:** browsers auto-stop speech recognition after a few seconds of silence; `onend` auto-restarts unless `userStopped` is true. The session timer initializes in `toggleRecording`, deliberately not in `onstart`, so restarts don't reset the clock.
- **Metric honesty:** `metricsAreEstimated` (builder push) and `typedMode` (no speech API) annotate the prompt; the rubric explicitly tells the AI to skip delivery analysis rather than invent it.
- **XSS:** all user-originated content rendered into history is passed through `escapeHTML`.
- **Quota:** all writes go through `trySetItem`, which catches quota errors and directs the student to export + prune.

### Rubric governance

Weights (10/30/30/30) were set by program staff in July 2026. Delivery & Mechanics is deliberately excluded from the score so results are comparable across recorded, estimated, and typed input modes. The scoring anchors and the "most first attempts land 55–75" instruction exist to counter AI grade inflation; they are **uncalibrated against a real cohort** — expect to tune after pilot use. Cross-model score variance is real: coaches should tell students to use one AI tool consistently and treat the trend, not the level, as the signal.

### Known limitations

- Filler detection false-positives on "so" and "like" (every occurrence counts).
- Speech recognition is `en-US` only; Safari support is inconsistent (falls back to typed input).
- Word Count includes interim recognition results, so it can flicker during recording.
- Chart requires network access once per session (pinned CDN).
- Score comparability across different AI tools is not guaranteed (see Rubric governance).

### Changelog

**v2.0 — July 2026**
- Restructured into the four-tab workflow (Guide / Record & Prompt / Paste Feedback / Saved History), matching the Networking Pitch Coach experience; sidebar and mobile drawer replaced by the top tab bar.
- Added 0–100 scoring: weighted rubric (Hook 10 / Headline 30 / Highlight Reel 30 / Future Focus 30, `TMAY-rubric-v1.0`) with per-dimension anchors and anti-inflation instruction; `SCORE: NN/100` output contract with auto-extraction on paste-back.
- Added saved history: dual localStorage stores (`tmayHistory` / `tmayScoreLog`), per-entry delete that preserves chart points, separate chart-only and full clears, JSON export/import with dedup, storage meter with tiered warnings, quota-safe writes, XSS-escaped rendering, rubric version stamping.
- Added Chart.js progress chart (pinned CDN, graceful degradation), track selector (General / Career Switcher / Fast Tracker LDP / Tech PM) coloring points and segments, custom legend, tooltips with score · track · date.
- Added `pendingRun` snapshotting so saved records match exactly what the AI evaluated; track changes regenerate the prompt without re-recording.
- Delivery & Mechanics moved from scored dimension to unscored "Delivery Notes."

**v1.1 — July 2026**
- Fixed: Tech PM tab button missing its parameter; invalid `gap: 1fr` CSS; recording silently ending on natural pauses; fabricated Push-to-Sandbox metrics (now labeled estimates); false "100% local" privacy claim; "asset assets" typo.
- Added: mobile nav drawer (superseded in v2.0), standard footer, unified program branding, typed-input polish, clipboard-failure fallback.

**v1.0 — July 2026**
- Initial build: framework sections, model answer bank, script builder, speech analytics sandbox, AI prompt generator.

---

Developed by Cory Burk, Senior Manager, Program Management · Full-Time MBA Program · David Eccles School of Business.
© 2026 University of Utah, David Eccles School of Business. All rights reserved.
