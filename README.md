# TrippyJam

An anonymous voicemail line for the Machine Cinema community. People record a
short story about a psychedelic experience, the site masks their voice
**entirely in the browser**, they approve the masked version, and only then does
anything leave their device. Each submission gets a reference code that can be
used to withdraw it later, no questions asked.

The collected audio is the raw material for a series of **online GenJams**:
animators gather for a day, hear a masked story for the first time, and turn it
into a short video.

The site is fully static — no build step, no framework, no server-side code.

## Pages

| File | What it is |
| --- | --- |
| `index.html` | The line itself: record, mask, review, send. Also a text fallback for people who would rather type. |
| `gallery.html` | The line so far: a listening room for already-submitted (masked) recordings, with per-call waveforms, scrub-to-seek, and one-at-a-time playback. |

## Running it

Serve the folder over HTTP (microphone access and `fetch` of the manifest both
need it):

```sh
python3 -m http.server 8000
# then open http://localhost:8000
```

## How the gallery gets its audio

`gallery.html` reads `gallery/manifest.json` and renders one call card per
entry. To publish a new submission:

1. Drop the masked `.wav` (the file the sender approved — never the original
   recording) into `gallery/audio/`.
2. Add an entry to the `calls` array in `gallery/manifest.json`:

```json
{
  "file": "audio/ACD-E47.wav",
  "ref": "ACD-E47",
  "voice": "cellar",
  "added": "2026-08-06",
  "note": "Optional one-line context shown under the waveform."
}
```

- `ref` is the submission's reference code — it's how a sender can ask for
  their story to be pulled (delete the file and the entry, done).
- `voice` is one of `cellar`, `kite`, `switchboard`, `crowd` and controls the
  chip colour on the card.
- Entries render in manifest order; put the newest first.

The four clips currently in `gallery/audio/` are **synthesized demo
placeholders** (marked as such in their notes), not real submissions — replace
them once real ones arrive.

## Brand assets

`brand/` holds the Machine Cinema logo set — white ink on transparent, which is
what the dark field wants:

| File | Lockup | Used on the site |
| --- | --- | --- |
| `machine-cinema-horizontal.png` | Camera + single-line wordmark | Masthead strip, both pages |
| `machine-cinema-stacked.png` | Camera above wordmark | Footer sign-off, both pages |
| `machine-cinema-mark.png` | Camera only | Receipt stamp, gallery empty/error state |
| `machine-cinema-compact.png` | Camera + two-line wordmark | — (spare) |
| `machine-cinema-compact-alt.png` | Two-line wordmark + camera | — (spare) |
| `favicon.png` | Mark on the field colour | `<link rel="icon">`, both pages |

The favicon is the only derived file: the mark composited onto `--field`
(`#170E22`) so it stays visible against light browser chrome, where white ink on
transparent would disappear.

## Collecting submissions

`CONFIG.endpoint` in `index.html` is `null` by default, which runs the whole
flow end-to-end and hands the masked file back to the sender instead of
posting. Point it at a collection endpoint that accepts `multipart/form-data`
(`kind`, `ref`, and either `audio` + `voice` or `text`) to actually receive
submissions.

## Privacy posture

- Masking (WSOLA pitch shift + filtering + drive) runs in an
  `OfflineAudioContext` on the sender's device; the raw recording is never
  uploaded.
- The sender hears the masked version and picks the voice before sending.
- No accounts, no cookies, no analytics.
- Only masked audio ever appears in the gallery.
