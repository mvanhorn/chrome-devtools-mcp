Title: feat: add a server-scoped downloads directory

## Summary

Adds `--downloadsDir` so Chrome saves downloads into a directory chosen at
server startup, without a save prompt, and reports each completed or canceled
download on the tool response that triggered it.

Closes #2397

## Design

Download destinations have to stay server-scoped. A tool argument that accepted
an arbitrary filesystem path would let a model write outside the workspace.
`--downloadsDir` is a startup / config-file option only. Tool schemas do not
take a download path.

When the option is set, every browser context this server manages is configured
with one CDP call: `Browser.setDownloadBehavior` with
`{behavior: 'allow', downloadPath, eventsEnabled: true, browserContextId}`.
That covers the default context, pages opened later, isolated contexts, and
browsers attached with `--browserUrl` or `--autoConnect`. The directory is
resolved once in `ServerConfig.create`. There is no Puppeteer
`downloadBehavior` launch/connect option and no
`BrowserContext.setDownloadBehavior` call.

The directory is created if it is missing. Startup fails if the path exists and
is not a directory. Omitting the option leaves Chrome's default prompt and
location unchanged.

## Behavior

- `click` and other tools that run through the shared browser-tool path wait
  briefly for in-flight downloads, then append a short line:
  `Download completed: <name> -> <path> (<n> bytes)` or
  `Download canceled: <name> (<n> bytes)`.
- Isolated contexts created after startup receive the same allow policy.
- Attach mode does not write Chrome profile preferences. On dispose, the
  collector sends `Browser.setDownloadBehavior {behavior: 'default',
  browserContextId}` for each context it configured, then detaches the CDP
  session, so the user's browser does not keep saving into the folder.

## Testing

- `npm run test` (spec reporter): 1959 tests, 1957 pass, 0 fail, 2 skipped
  (Windows-only daemon path tests on Linux), 69.1s.
- `npm run check-format`: pass.
- `npm run typecheck`: pass.
- `npm run gen`: regenerated `docs/configuration.md` for the updated option
  text.

Coverage includes: flag unset does not open a CDP session for downloads; flag
set applies the allow policy to the default and isolated contexts; a
non-directory path is refused at startup; dispose restores `{behavior:
'default'}` per applied context; a real-browser `Content-Disposition:
attachment` download lands in the configured directory and is reported on the
click response; canceled downloads format as canceled.

## Not covered

- Attach mode was not driven against a long-lived user Chrome profile. Reset
  is implemented as the CDP `default` restore above and covered by unit tests.
- There is no `list_downloads` tool. Completed and canceled downloads are
  reported on the triggering tool response.
- The recorded CLI capture covers completed downloads and the non-directory
  refusal. Canceled downloads and isolated-context policy application are
  covered by unit tests, not that capture.

## Demo

![demo](https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1115/download-behavior/preview.gif)

Full recording (MP4): https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1115/download-behavior/out.mp4
