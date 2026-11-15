Title: feat: add JavaScript and CSS code coverage tools

## Summary

Adds opt-in JavaScript and CSS coverage tools so an agent can measure unused
bytes on the selected page. Coverage is a separate tool category, disabled by
default, behind `--categoryCoverage`.

Closes #731.

## Design

Coverage is not collected as part of a performance trace. The flow is:

1. `start_coverage` on the selected page (JS, CSS, or both; defaults both).
2. Reload or interact with the page. `resetOnNavigation` defaults to false so a
   reload is included. Pass `reload: true` to reload immediately after start.
3. `stop_coverage` stops collection and returns a compact unused-bytes report.

Starting twice without stopping is an error. Stopping when nothing is running
is an error. If coverage is running on another page, the error names that page
and how to target it. A failed start or reload stops any coverage that did
start so the page is not left collecting.

`--slim` and disabling the category stop running sessions.

## Output

Each resource lists type (JS or CSS), total bytes, used bytes, unused bytes,
and unused percent. Rows are sorted by unused bytes. Long URLs are truncated.
Inline and anonymous scripts are labeled `(inline/anonymous)`. The formatter
keeps the top 20 rows and a totals line; extra rows are counted as omitted.

## Testing

- `npm run gen` — pass
- `npm run check-format` — pass
- `npm run typecheck` — pass
- `npm run test` — 2010 tests, 2008 pass, 0 fail, 2 skipped

Coverage-specific coverage includes mock handler tests, `McpPage` start/stop
and restore-on-failure tests, formatter snapshots, category listing
(`--categoryCoverage` off by default / on when enabled), config release of
running sessions, and one real-browser test with used and unused JS and CSS.

## Not covered

- Block-level / raw script coverage
- Source-map mapping of unused ranges back to original files
- Collecting coverage during a performance trace (intentionally unsupported)

## Demo

![demo](https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1115/code-coverage/preview.gif)

<video src="https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1115/code-coverage/out.mp4" controls width="720"></video>
