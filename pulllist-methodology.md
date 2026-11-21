# PullList — how this works

> **Revised 2026-11-21.** Changes since the August version: series naming is now `Name (LaunchYear)` for every publisher (no more `Vol. X`); the monthly solicitations task exists; the importer accepts both field-name styles; the chat-based research workflow below is unchanged.

If you're Claude reading this at the start of a new chat: **this file is the
instruction set for a standing workflow called PullList.** The person has
uploaded it so you can pick up the project without re-explaining it. Read
this in full before doing anything else, then proceed with whatever they ask.

## What PullList is

PullList is a research/compilation workflow run in Claude chat. It has no
code of its own — it's just this methodology plus the Research feature. The
person separately runs **ComicsTracker**, a self-hosted comics-tracking web
app (GitHub Pages + their own Google Drive for storage). PullList's entire
job is: research comics data, then hand back a `.json` file shaped exactly
the way ComicsTracker's Import tab expects, so it can be uploaded there
directly with zero reformatting.

Never confuse the two names:
- **ComicsTracker** = the app (GitHub Pages + Drive).
- **PullList** = this chat-side research workflow.

## Output format (required)

Every PullList deliverable is a downloadable `.json` file: a flat array of
issue-row objects. Exact field names, always:

```json
[
  {
    "seriesName": "Uncanny X-Men",
    "number": "492",
    "title": "",
    "publisher": "Marvel",
    "releaseDate": "2007-11-07",
    "events": ["X-Men: Messiah Complex"],
    "readOrder": 2
  }
]
```

- `seriesName` — see naming convention below.
- `number` — string, not int (handles values like `"3.1"`, `"0"`, `"12.1"`, `"27.NOW"`).
- `title` — leave `""` if the issue has no distinct story title (true for
  most modern single-issue superhero comics — don't invent one).
- `publisher` — the actual publisher name (`"Marvel"`, `"DC"`, `"BOOM! Studios"`, etc).
- `releaseDate` — `YYYY-MM-DD` on-sale/street date, **not cover date**
  (cover dates run ~2 months ahead). Leave `""` if genuinely unverified —
  never invent a date.
- `events` — array of event-name strings (an issue can belong to more than
  one event — e.g. a tie-in that bridges two crossovers).
- `readOrder` — integer, **scoped per event**, not global. When an issue
  belongs to multiple events, this field holds the order for whichever event
  was tagged first; that's an accepted limitation, not a bug to fix.

The importer also accepts these alternates, so either style imports cleanly:
`series`/`series_name`, `issue`/`issue_number`, `date`/`release_date`,
`reading_order`/`read_order`, `event_tags`. Extra fields (`writer`, `price`)
are ignored. The canonical names above are preferred for chat-built files.

ComicsTracker's Import tab **merges on `(seriesName, number)`** — matching
existing issues get their `events` unioned in and blank fields filled, but
existing `read`/`purchased`/`dateHistory` are never touched. This means
re-pulling a series you've already partly catalogued is always safe.

## Series naming convention

- **Every publisher, Marvel included:** `Series Name (LaunchYear)`, where
  LaunchYear is the year that volume's #1 went on sale. Examples:
  `"Avengers (2013)"` (Hickman), `"Avengers (2010)"` (Bendis),
  `"Thunderbolts (1997)"`, `"Nightwing (2016)"`,
  `"Mighty Morphin Power Rangers (2016)"`. **Never use `Vol. X`.**
  (Changed 2026-10-22; all Marvel series in Drive were migrated 2026-10-23.)
- A title with one continuous run still gets its launch year, e.g.
  `"Uncanny X-Men (1963)"`.
- If two genuinely different runs would produce the same name (same-year
  relaunch), flag it to the user rather than guessing; they'd merge on import.
- One-shots and minis get their own `seriesName` (don't fold them into a
  parent series' issue numbering).

## The build methodology (publisher-by-publisher)

This is the standing plan, run one publisher at a time:

1. **Main continuity only** — skip alternate-universe lines (Ultimate
   Marvel, DC Elseworlds, etc.) unless the person asks for them separately.
2. **Start points**: Marvel starts at *Avengers Disassembled* (2004); DC
   starts at *Blackest Night* (2009).
3. **Start with major crossover events.** Use comicbookreadingorders.com's
   event-timeline pages (`comicbookreadingorders.com/marvel/event-timeline/`,
   `.../dc/event-timeline/`) as the master list of events for that era.
4. **Group events into eras** of roughly 14–16 events each (this keeps each
   Research task scoped well) and confirm the scope/grouping with the person
   before launching each one.
5. **Every event gets tagged**, including ones that turn out not to be a
   real interleaved crossover — see classification below. Tag first, ask
   questions never; the person's standing instruction is "tag everything
   with reading order," full stop.
6. **After the events pass**, flesh out full runs of the ongoing series that
   participated (e.g. one tie-in issue in Iron Man Vol. 4 → pull that
   volume's full run). This is the step that backfills release dates for
   issues the event-timeline research didn't date precisely.
7. **Then review major character-group lines** for gaps (Marvel: X-Men,
   Spider-Man, Avengers, etc.) not caught by the events pass.
8. **Final cleanup pass** for stray/unrelated issues.

### Event classification (apply consistently)

Not every named "event" is a real multi-title crossover. Classify each one
in the research and say so plainly, but **tag all of them regardless**:

- **Genuine crossover** — real interleaved reading order across ≥2 titles.
  Tag with a curated `readOrder`, not date-sort (parallel titles ship the
  same week and date-sort alone loses the intended sequence).
- **Branding banner** — a line-wide status-quo label (e.g. "The Initiative,"
  "Dark Reign," "Heroic Age") with no single core story. Tag every listed
  issue with the event name; `readOrder` sequencing is best-effort, not
  load-bearing, and these are usually the least-complete pulls (hundreds of
  issues can wear the banner — expect the events pass to only catch a
  representative slice, backfilled later by the full-series-runs step).
- **Single-title arc** — an "event" that's really just a story arc inside
  one ongoing series (e.g. "Planet Hulk," "Dying Wish"). Still tag it, still
  give it a `readOrder`, just note in your reply that it's not a real
  crossover.
- **One-shot anthology** — a set of loosely-linked standalones with no
  interleaving (e.g. "Revolutionary War"). Reading order = publication
  order, nothing fancier.

### Reading order & same-week judgment calls

When two tie-ins ship the same week, curated fan reading order (via
comicbookreadingorders.com / League of Comic Geeks) usually has an opinion
on which comes first — follow it, and flag the judgment call in your reply
rather than silently picking one. Don't just fall back to date-sort for
interleaved events; that defeats the purpose of the `readOrder` field.

### Sourcing priority

1. League of Comic Geeks event checklists and individual issue pages (best
   for exact on-sale dates).
2. comicbookreadingorders.com (best for curated reading order and event
   scope/tie-in lists).
3. Comic Vine, Marvel Database / Marvel Fandom, Wikipedia — cross-reference,
   don't rely on solo.
4. Never invent a date. If a source only gives month precision or the date
   is genuinely unconfirmed, leave `releaseDate: ""` and say so.

## Solicitations (automated, monthly)

A scheduled task, **"PullList monthly solicitations"**, runs on the 24th at
08:46 SGT. It finds the latest on-sale month published on comicreleases.com
(usually 3 months ahead), pulls Marvel, DC and BOOM! (Power Rangers only)
single issues, and saves `solicitations_YYYY-MM.json` to the Drive folder
**PullList**, with a push + email summary. Exclusions: TPBs, HCs, omnibuses,
facsimiles, reprints, variant-only listings, and licensed lines (Star Wars,
Alien, Predator, Godzilla, Planet of the Apes, Disney Duck, Indiana Jones, Sonic).

Ingest by hand: ComicsTracker → Import → Upload PullList .json → review →
Import selected. Newly solicited series arrive **unticked**; series already
tracked stay ticked. If a month's Marvel and DC solicits aren't out yet, the
task stops and says so rather than writing a partial file.

**Titles:** solicitations only carry a story/chapter title for some issues
(mostly minis, one-shots and arc-named issues). The rule is never to invent
one, so the rest arrive with `title: ""`. Titles can be backfilled later from
League of Comic Geeks once an issue is out; the importer fills blank titles
only, never overwrites.

## Workflow mechanics (how a pull actually runs)

1. Person names a publisher/era, or asks to continue.
2. If scope is large or ambiguous (which events belong in this era, how
   many events to batch together), present the event list and confirm
   before launching Research — don't assume.
3. Use the **Research** feature (`suggest_research` renders the button; if
   `launch_extended_search_task` is available that turn, call it directly)
   with a detailed prompt: list every event, ask for full issue lists with
   series/number/date/reading-order, ask for explicit classification of
   which events are genuine crossovers vs banners vs arcs vs anthologies,
   specify sourcing priority as above.
4. When the research artifact comes back, build the JSON file **with a
   Python script**, not by hand-typing hundreds of rows — use a `add()` /
   `seq()` helper pattern keyed by `(seriesName, number)` so cross-event
   issues (an issue tagged in two events) merge their `events` arrays
   automatically rather than creating duplicate rows.
5. Validate: row count, JSON parses, spot-check any issues tagged with
   multiple events actually merged correctly (this is the main
   correctness signal that the build script worked).
6. Present the file, and in your reply: confirm counts, name which events
   are genuine vs banner/arc/anthology, flag what's *not* fully dated yet
   and why (e.g. "banner events are always partial — full series runs will
   backfill this").

## Progress log

*(Keep this section updated as new pulls complete, so a fresh chat can see
what's done and what's next without re-deriving it.)*

**Completed:**
- Generation X-23 (2026) — full run, Marvel
- Dark Nights: Metal (2017) — core + tie-ins, DC
- BOOM! Studios Power Rangers main-line catalog (2016–2026) — 11 series,
  197 rows, 6 events fully tagged (Shattered Grid, Necessary Evil, Unlimited
  Power, The Eltarian War, Charge to 100, Darkest Hour)
- Marvel Era 1 (2004–2008) — 16 events, 502 rows
- Marvel Era 2 (2007–2010, Dark Reign era) — 16 events, 619 rows
- Marvel Era 3 (2010–2012, Heroic Age → early Marvel NOW!) — 15 events, 389 rows
- Marvel Era 4 (2013–2015, Marvel NOW!) — 14 events, 321 rows

**Next up:**
- Marvel: Secret Wars (2015) — the universe-reset event closing out the 616
  line pre-relaunch; flagged as deserving its own dedicated pull given scale
  and continuity consequences, not bundled into Era 4.
- Marvel: post-Secret Wars eras (2015 relaunch onward) — not yet scoped.
- DC: hasn't been started yet. Starts at *Blackest Night* (2009) per the
  standing methodology above. Pull the DC event-timeline list first
  (`comicbookreadingorders.com/dc/event-timeline/`) and confirm era
  groupings with the person before launching Research.
- Step 6 (full series runs) and step 7 (character-group review) haven't
  been run for Marvel yet — all four Marvel eras so far are events-only
  passes. Series like Uncanny X-Men, Avengers, Iron Man Vol. 4 etc. that
  recur across multiple events still have gaps and unconfirmed dates that
  full-run pulls would fill in.
