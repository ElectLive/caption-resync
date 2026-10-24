# Good First Issues

New to the project? These four are scoped to be finishable in one sitting.
The test suite runs with zero dependencies: `python3 -m unittest discover tests -v`.

---

### 1. Add `--fix-overlaps`: trim overlapping cues automatically

**Context.** `--check` already reports overlapping cues, but the user still has
to fix them by hand. A `--fix-overlaps` flag should trim each cue's end time to
the next cue's start time (only when they actually overlap), so `--check`
comes back clean afterwards.

**Acceptance criteria**
- New `--fix-overlaps` flag in `build_parser()`, applied after `--shift`/`--stretch`.
- Only shortens end times; never moves start times, never touches durations otherwise.
- `--check` reports zero overlap errors on a file processed with the flag.
- At least 3 new tests: trims overlap, leaves non-overlapping cues alone, does
  not create zero-duration cues (clamp the trim at `start + 1` ms).

**Hints.** The overlap check lives in `check_doc()`; mirror its comparison in a
new `op_fix_overlaps(doc)` function next to `op_shift`.

---

### 2. Support MicroDVD (`.sub`) files as input

**Context.** MicroDVD `.sub` files use frame numbers instead of timestamps
(`{0}{72}Hello`), plus an fps value. Supporting them as input (converted to ms
on parse) would let editors batch-convert old DVD rips to SRT.

**Acceptance criteria**
- Parse `{start}{end}text` lines; accept an `--fps` option (default 23.976) to
  convert frames to milliseconds.
- `.sub` files are picked up by the default `--dir` glob patterns.
- Output goes through the normal SRT/VTT rendering path.
- At least 4 new tests: frame to ms math, default fps, custom `--fps`,
  multi-line `{..}` text.

**Hints.** Frame to ms is `frame / fps * 1000`. A new `parse_sub(text, fps)`
function fits beside `parse_srt`/`parse_vtt`; wire format detection into
`parse_text` and the output extension logic in `main`.

---

### 3. Add `--stats`: word count and reading-speed report per file

**Context.** Subtitle editors often check whether cues are readable: words per
minute (WPM) above ~200 is hard to follow. A `--stats` mode should print per
file: cue count, total words, average WPM across cues, and the number of cues
over 200 WPM.

**Acceptance criteria**
- `--stats` prints a small table to stdout and does not write output files.
- WPM per cue = `words / (duration_seconds / 60)`, skipped for zero-duration cues.
- Works alongside `--dir` batch mode with one row per file plus a totals row.
- At least 3 new tests: word counting, WPM math on a known cue, totals row.

**Hints.** Pure function `stats_for_doc(doc)` returning a small dataclass keeps
this easy to test without touching the CLI.

---

### 4. Preserve `NOTE` blocks on WebVTT to WebVTT processing

**Context.** The VTT parser currently drops `NOTE` comment blocks. When the
input and output are both WebVTT and no conversion happens, dropping comments
loses author metadata.

**Acceptance criteria**
- `NOTE` blocks are stored during parsing and re-emitted in `render_vtt()`,
  in their original position relative to cues.
- Dropping still happens on VTT to SRT conversion (SRT has no comment blocks).
- `--strip-styles` does not touch `NOTE` blocks.
- At least 3 new tests: NOTE survives VTT to VTT, NOTE dropped on VTT to SRT,
  multiple NOTEs keep their order.

**Hints.** Store notes as `(cue_index, text)` pairs in `SubtitleFile` so the
renderer can interleave them correctly.
