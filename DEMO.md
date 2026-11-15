# Demo: server-scoped `--downloadsDir`

This branch holds the recorded CLI run for `feat/download-behavior`.
The video is an animated replay of `commands.log`, not a mock.

## What was captured

Real built CLI, from the repo root:

```
node out/Default/gen/src/bin/chrome-devtools.js \
  --sessionId c8ae1115-0001-4000-8000-00000000d1d1 \
  start --headless --isolated --downloadsDir /tmp/cdm-downloads
```

A local page served `Content-Disposition: attachment; filename="file.txt"`.
The CLI then ran `new_page`, `take_snapshot`, and `click`.

## Evidence in `commands.log`

1. **Flag set.** `click` printed
   `Download completed: file.txt -> /tmp/cdm-downloads/file.txt (40 bytes)`.
   `ls`, `sha256sum`, and `cat` showed the file in `/tmp/cdm-downloads`.
2. **Flag unset.** The same click succeeded but did not write
   `/tmp/cdm-downloads/file.txt`.
3. **Non-directory path.** `start --downloadsDir /tmp/cdm-not-a-dir`
   exited 1 with
   `Invalid downloadsDir /tmp/cdm-not-a-dir. Expected a directory, found a file.`

## Video

`out.mp4` is a 24s 1920x1080 30fps H.264 replay of the exact log lines,
labeled **Animated replay of real captured command output**.
`preview.gif` is the 960x540 12fps GIF derived from that MP4.
See `logs/ffprobe.txt` for probe output.
