# Changelog

All notable changes to caption-resync will be documented in this file.

## [1.0.0] - 2026-10-24

Initial release.

### Added
- Robust SRT and WebVTT parsing: UTF-8 BOMs, comma vs dot milliseconds,
  optional cue identifiers, `NOTE` blocks, `STYLE`/`REGION` header blocks,
  VTT cue settings.
- `--shift`: move all cues by milliseconds (`2500`), seconds (`2.5s`), or a
  timestamp (`00:00:02,500`); negative shifts clamp at zero with a warning.
- `--stretch`: multiply timestamps to fix framerate drift (e.g. 23.976 to
  25 fps).
- `--to srt|vtt`: format conversion; SRT to VTT to SRT round-trips preserve
  text and timing within 1 ms.
- `--strip-styles`: remove `<i>`/`<b>`/`<font>` tags, `{...}` override tags,
  and VTT cue position settings.
- `--renumber`: sequential cue identifiers from 1.
- Batch mode: multiple files, `--dir` with comma-separated `--glob` patterns,
  configurable `--suffix`, `--out-dir`, `--dry-run` previews.
- `--check`: validator reporting malformed cues, zero/negative durations,
  overlaps, and out-of-order timestamps; exits 1 on errors.
- 51-test suite, zero dependencies, Python 3.9+.
