# caption-resync

[![GitHub stars](https://img.shields.io/github/stars/aamirtechs/caption-resync?style=social)](https://github.com/aamirtechs/caption-resync/stargazers) [![GitHub forks](https://img.shields.io/github/forks/aamirtechs/caption-resync?style=social)](https://github.com/aamirtechs/caption-resync/network/members) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/aamirtechs/caption-resync/blob/main/LICENSE) [![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/aamirtechs/caption-resync/issues)

Batch-fix out-of-sync subtitles from the command line. Shift, stretch, convert
between SRT and WebVTT, strip styling tags, renumber, and validate whole folders
of subtitle files. The free, local alternative to paywalled subtitle sync tools.

Zero dependencies. Python 3.9+.

## Why

Your subtitles lag the audio by a constant offset, or the video was converted
from 23.976 fps to 25 fps and every cue drifts further out of sync as the file
plays. GUI subtitle editors make you fix files one at a time. caption-resync
fixes the whole folder in one command and never touches your originals: outputs
are written next to the inputs with a `.resync` suffix (or into `--out-dir`).

## Install

```bash
pip install caption-resync
# or just run the single file, it has no dependencies:
python3 caption_resync.py --help
```

## Quick example

Say every line of `episode.srt` arrives 2.5 seconds late. Preview the fix, then
apply it:

```bash
$ caption-resync --check episode.srt
1 file(s), 4 cue(s): 0 error(s), 0 warning(s)

$ caption-resync --shift=-2.5s --dry-run episode.srt
episode.srt -> episode.resync.srt [shift -2500 ms, 4 cue(s)]
  first cue now: 00:00:01,000 --> 00:00:04,000

$ caption-resync --shift=-2.5s episode.srt
episode.srt -> episode.resync.srt [4 cue(s)]
done: 1/1 file(s) processed
```

(The outputs above are captured from a real run on
`examples/offset-2.5s.srt`, a sample file whose cues are deliberately 2.5
seconds late.)

After the shift, the file reads:

```
1
00:00:01,000 --> 00:00:04,000
<i>Previously</i> on the show...

2
00:00:05,500 --> 00:00:08,750
We left our hero <b>staring</b> at the timeline.
```

(Captured output.)

Convert SRT to WebVTT while stripping styling tags in the same pass:

```bash
$ caption-resync --to vtt --strip-styles episode.srt -q
$ cat episode.resync.vtt
WEBVTT

1
00:00:03.500 --> 00:00:06.500
Previously on the show...

2
00:00:08.000 --> 00:00:11.250
We left our hero staring at the timeline.
```

(Captured output.)

## Usage

```
caption-resync [files ...] [options]
caption-resync --dir subtitles/ --glob "*.srt" --shift=-2.5s
```

| Option | What it does |
|---|---|
| `--shift OFFSET` | Shift all cues. Forms: `2500` (ms), `2.5s` (seconds), `00:00:02,500` (timestamp). Prefix `-` to move earlier. Negative results clamp at zero with a warning. Tip: use `--shift=-2.5s` (with `=`) so the shell does not treat the value as a flag. |
| `--stretch FACTOR` | Multiply every timestamp, e.g. `1.0427` fixes 23.976 to 25 fps drift. |
| `--to srt\|vtt` | Convert the output format. |
| `--strip-styles` | Remove `<i>`, `<b>`, `<font>`, `{...}` override tags and VTT cue position settings. |
| `--renumber` | Renumber numeric cue identifiers sequentially from 1. |
| `--dir DIR --glob PAT` | Batch process a directory (default patterns: `*.srt,*.vtt`). |
| `--suffix S` | Output filename suffix (default: `.resync`). |
| `--out-dir DIR` | Write outputs here instead of next to the inputs. |
| `--dry-run` | Preview what would change without writing files. |
| `--check` | Validate only: reports malformed cues, zero/negative durations, overlaps, and out-of-order timestamps. Exits 1 on errors. |
| `--force` | Process files even when parsing found errors. |
| `-q, --quiet` | Less output. |

Validator example (captured output):

```bash
$ caption-resync --check bad.srt
[ERROR] bad.srt: cue 2 overlaps the previous cue by 1000 ms
[ERROR] bad.srt: cue 3 starts before the previous cue (out of order)
1 file(s), 3 cue(s): 2 error(s), 0 warning(s)
$ echo $?
1
```

Framerate drift example (captured output):

```bash
$ caption-resync --stretch 1.0427 --dry-run episode.srt
episode.srt -> episode.resync.srt [stretch x1.0427, 4 cue(s)]
  first cue now: 00:00:03,649 --> 00:00:06,778
```

## Format support

- **SRT**: cue numbers (optional), comma milliseconds, multi-line text.
- **WebVTT**: `WEBVTT` header (with optional title text), cue identifiers, cue
  settings (`align:`, `position:`, ...), `NOTE` blocks (skipped), `STYLE` and
  `REGION` blocks (preserved on conversion).
- Robust to UTF-8 BOMs, comma vs dot milliseconds, and 2-digit `mm:ss.mmm`
  timestamps. SRT to VTT to SRT round-trips preserve cue text and timing
  within 1 ms (covered by the test suite).

## Development

```bash
python3 -m unittest discover tests -v   # 51 tests, no dependencies needed
```

See [GOOD_FIRST_ISSUES.md](GOOD_FIRST_ISSUES.md) for contributor-friendly
starting points and [CHANGELOG.md](CHANGELOG.md) for release notes.

## License

MIT. See [LICENSE](LICENSE).
