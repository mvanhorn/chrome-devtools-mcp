# Code coverage demo

This branch holds recorded evidence for `feat/code-coverage`.

## What was captured

A local fixture page (`coverage.html` plus `app.js` / `app.css`) was served at
`http://127.0.0.1:8765`. The built CLI was started with
`--headless --isolated --categoryCoverage`.

Commands in `commands.log`:

1. `start` with `--categoryCoverage`
2. `new_page` to the fixture
3. `start_coverage 2 --reload true`
4. `take_snapshot` / `click` on the Go button
5. `stop_coverage` compact report (CSS unused 92.5%, JS unused 27.0%)
6. MCP `tools/list` without the flag: coverage tools absent (33 tools)
7. MCP `tools/list` with `--categoryCoverage`: `start_coverage`, `stop_coverage` (35 tools)
8. `stop_coverage` without a prior start: `No coverage session is running on page 2`

`out.mp4` and `preview.gif` are an animated replay of those exact log lines,
labeled "Animated replay of real captured command output".

## Media

- `out.mp4`: 1920x1080, 30 fps, 720 frames, 24.000 s, H.264 yuv420p
- `preview.gif`: 720x405, 12 fps, 288 frames, 24.000 s, 9.5 MB
