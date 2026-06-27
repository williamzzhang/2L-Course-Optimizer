# 2L Ballot Optimizer

A single-page tool for planning a 2L course ballot at Osgoode Hall Law School. You build a ranked ballot from the course list, and the tool estimates your odds of getting a full schedule, draws your weekly timetable, and checks for exam conflicts. Everything runs client-side in one HTML file with no build step, no dependencies, and no server.

## What it does

Course registration runs on a ranked ballot against capped sections with uncertain demand, so whether you actually land a workable schedule isn't obvious from the course list alone. This tool takes the ballot you'd submit and answers three questions: how likely each pick is to clear, whether the picks you'd most want fit together on a weekly grid, and whether their exams collide. The course, timetable, and exam data are baked into the file, reconciled across three source spreadsheets that name courses inconsistently.

## Usage

Open `2L_Ballot_Optimizer.html` in any modern browser. Pick courses into the ranked slots, then run the simulation. Results, both timetables, and the exam check render below. Nothing is uploaded anywhere.

## How it works

The ballot data lives in a `RAW` array, where each course carries its semester, credits, section cap, historical demand, and flags. A Monte Carlo simulation runs the ballot many times against randomized demand to estimate success odds. Two baked-in lookup tables, resolved offline from the timetable and exam spreadsheets, drive the weekly grid and the exam checker. Because the three source files name the same course differently (punctuation, dropped words, and different instructors who set the exam), the matching was done ahead of time and verified by hand, so the in-browser lookups are exact rather than fuzzy.

## Functions

### Simulation

- **`runSim`** — Reads the ranked ballot, runs 8,000 Monte Carlo iterations in chunks (so the progress bar animates), and tallies how often each course clears and how often each semester reaches a full 13–17 credit load. Validates the ballot for emptiness and duplicates first.
- **`randn`** — Box–Muller transform that draws a standard normal value, used to jitter each course's demand on every iteration.
- **`showResults`** — Renders the results table: each pick's queue position, semester, success percentage with a bar, and a Safe / Moderate / High-risk pill based on the simulated clear rate.

### Ballot building

- **`buildSlots`** — Generates the ranked course-selection slots in the left panel.
- **`rebuildAllDropdowns`** / **`onSelect`** — Populate the course dropdowns and react to a selection, keeping the remaining menus in sync.
- **`updateSlotMeta`** / **`updateSelCount`** / **`updateReqs`** — Refresh the per-slot detail line, the selected-count tag, and the running credit and requirement readouts.
- **`passesFilter`** / **`pill`** — Apply the active course filters and toggle filter pills.

### Timetable

- **`buildTimetable`** — Entry point that collects the selected courses, runs conflict detection per semester, and renders the Fall and Winter grids stacked vertically.
- **`ttCollect`** — Splits the ballot by semester and attaches each course's meeting times from the schedule table, separating fixed-time courses from ones with no scheduled slot.
- **`ttConflicts`** — Detects any time overlap between meetings on the same day within a semester, returning the conflicting course set and the specific overlap windows.
- **`ttRenderGrid`** — Draws a Monday–Friday calendar grid, positions each course block by start and end time, and lays overlapping blocks side by side in columns. Conflicting blocks render in red.
- **`ttRenderAsync`** — Lists courses that carry an extra asynchronous hour or have no fixed meeting time below the relevant grid.
- **`ttRenderBanner`** — Summarizes timetable conflicts (or confirms there are none) above the grids.
- **`ttFmt`** — Formats minutes-since-midnight into a readable clock time.

### Exam conflicts

- **`buildExams`** — Entry point that gathers the exam for each selected course, treats every exam as a three-hour interval, and classifies same-day pairs as either a hard overlap or a same-day heads-up. Courses with no exam on the registrar schedule are assumed take-home or paper-based and skipped.
- **`examRenderBanner`** — Summarizes exam overlaps and same-day pairs, or confirms a clean spread.
- **`examRenderList`** — Renders a chronological list of every exam date among the picks, grouping the courses on each day and tagging any day that has a conflict.
- **`examFmt`** — Formats an exam start time for display.

## Data and assumptions

- **Course odds** come from each section's cap and historical demand. Courses with a new instructor have no demand history, so their odds are estimated from the section cap instead and flagged as `New`.
- **Timetable times** were parsed from a spreadsheet that stores times in a 12-hour format with no AM/PM marker. The governing rule: the earliest class starts at 8:30, so any clock hour from 1 to 7 is PM, and an end time that lands before its start is also PM. All courses were checked for sane start, end, and duration.
- **Exam conflicts** assume each exam is three hours long, since the schedule lists only a start time. Two exams whose three-hour windows overlap are a hard conflict (red); two on the same day that don't overlap are a same-day heads-up (amber).
- **Missing data** is treated as non-conflicting. A course with no timetable entry shows as having no fixed meeting time; a course with no exam entry is assumed take-home or paper-based.

## Tech

A single self-contained HTML file: plain HTML, CSS, and vanilla JavaScript, with all course, timetable, and exam data inlined. No frameworks, no build tooling, no network calls.
