# Technical notes

How an episode gets from a Google Meet to YouTube, and the traps found along the way.
Episode 1 is the worked example; the numbers below are from it.

## Recording

Two recordings run at once and we use both:

- **Jimmy's recording** (Vibe Conferencing). Per-track audio for every participant
  plus the bot's own voice, wall-clock stamped, in the call folder under
  `~/Library/Application Support/Vibeconferencing/profiles/<profile>/agent/calls/`.
  This is the good audio: every voice arrives at Meet's normalised level. It stops
  whenever the bot drops out of the call, and each rejoin starts a new call folder.
- **A QuickTime screen recording** on Stan's Mac, with the menu-bar clock (seconds
  on) in the corner. This is the video, and the fallback audio for any stretch the
  bot missed. Its mic channel lands on the left only, so treat it as mono.

Keep both. Neither is complete on its own.

## Building the episode audio

1. **Find each bot segment's offset** by cross-correlating its audio against the
   QuickTime audio. Use a loudness envelope at ~100 Hz, not raw samples: the two
   recordings heard the room through different paths, so raw correlation scatters by
   a few hundred ms while the envelope lands within 20 ms. Check several windows
   along the segment; the offset should not drift.
2. **Mind the QuickTime audio start.** The audio stream in a QuickTime screen
   recording starts about 0.13 s after the video (`ffprobe` shows it as the audio
   `start_time`). Offsets measured against a plain audio dump are early by that
   much relative to the picture. Add it back, or measure against an audio dump that
   has been delayed by the same amount.
3. **Remix the bot segments from the per-track files, not from `call-recording.mp4`.**
   The bot's own merge mixes in its `share-audio` track on top of Meet's return of the
   same audio on a remote slot, so anything played through a tab share (the theme
   song) comes out doubled, about a quarter second apart. Mix `bot` plus the
   `remote-participant-*` tracks, aligned by `startWallClock` in `manifest.json`,
   and leave `share-audio` out. Tracked as
   [vibeconf-app#862](https://github.com/wanderingstan/vibeconf-app/issues/862).
   Decode each `.webm` to WAV before `amix`; feeding the webm files straight into
   the filter graph silently dropped all but one track.
4. **Pad every bot segment to its exact length.** The decoded audio is slightly
   shorter than the container says, and the shortfall accumulates (about a second
   over the hour). `apad=whole_dur=` then `atrim=end=` on each segment.
5. **Fill the gaps with the QuickTime audio**, left channel copied to both sides
   (`pan=stereo|c0=c0|c1=c0`), and join everything with 0.3 s crossfades.
6. **Check the result against the QuickTime audio** the same way as step 1. The
   QuickTime-sourced sections must come back at exactly zero offset; the bot
   sections within about 50 ms.

Work on an audio-only file first. A full-length `ffmpeg` pass that also copies 12 GB
of 240 fps video can deadlock in the filter graph; it is also much faster to
iterate on an `.m4a` and remux it onto the video at the end with `-c copy`.

## Checking sync without opening the 12 GB file

Cut three or four 40-second clips at 960 px wide with `-ss` before `-i` (seeking is
instant), one in each bot section and one across a join. Watch mouths and the
corner clock. Ten seconds of work per clip.

## Encoding for YouTube

Upload at native resolution, not 1080p. A 3024x1964 capture is classed above 1080p,
which gets YouTube's better VP9/AV1 ladder, and that helps the 1080p and 720p
renditions too, which matters for on-screen text. Do re-encode first though: the
QuickTime capture is 240 fps variable frame rate at 27 Mbps and YouTube caps at
60 fps.

```
ffmpeg -i screen-recording.mov -i episode-audio.m4a -map 0:v -map 1:a \
  -r 30 -c:v h264_videotoolbox -b:v 16M -pix_fmt yuv420p -c:a copy \
  -movflags +faststart -shortest episode.mp4
```

About 12 minutes for an hour of video on an M-series Mac, if nothing else is using
the encoder. Result: roughly 7 GB.

## The bumper

[`art/bumper.mp4`](art/bumper.mp4) is 4.6 s: the badge on a cream canvas, slow
push-in, fade in and out, with the super-short theme edit. It is encoded to the same
spec as the episode (3024x1964, 30 fps, H.264 High, AAC 48 kHz stereo), so it can be
prepended with a stream copy, no re-encode:

```
printf "file '%s'\nfile '%s'\n" "$PWD/art/bumper.mp4" "$PWD/episode.mp4" > /tmp/concat.txt
ffmpeg -f concat -safe 0 -i /tmp/concat.txt -c copy -movflags +faststart episode-with-bumper.mp4
```

Paths in the concat list resolve relative to the list file, so use absolute ones.
Any episode encoded with the settings above will match. If the spec changes,
regenerate the bumper to match rather than re-encoding the episode.

## Captions

Text and timing from whisper.cpp (`whisper-cli`, large-v3-turbo) on the final
episode audio. Speaker names from the bot's `speaker-events.jsonl`, which is Meet's
own speaking indicator, wall-clock stamped, so it maps onto the video timeline once
the offsets are known. For the gaps where the bot was out, a small classifier
trained on the QuickTime audio against the known-speaker sections fills in names at
about 83% per second (better per cue, since cues vote).

Whisper gotchas:

- Run it in five-minute chunks with `--prompt` and `--carry-initial-prompt`. On a
  single long pass it drifts into lowercase, unpunctuated, run-on segments after
  twenty minutes or so and never recovers, because its own output is its context.
- It writes `-` dialogue dashes when speakers change inside a segment. Split on them;
  they are a free speaker-change signal.
- The theme song comes out garbled. Find each play by correlating the song file
  against the episode audio, drop whisper's cues there, and drop in the real lyrics
  with per-line timing taken from a whisper run on the clean song file.

SRT hygiene YouTube cares about: no overlapping cues, lines under about 90
characters, nothing past the end of the video, UTF-8 without BOM.

**Uploading to YouTube Studio:** the top-level "Upload file" button on the Subtitles
page fails with "Upload captions failed. Retry upload." for any SRT, including a
three-cue ASCII test file. Click **Add language**, pick the language, and upload the
file from inside that dialog. That works first time.

Episode 1's captions: [`episodes/001-building-the-show.srt`](episodes/001-building-the-show.srt)
(timed for the version with the bumper).
