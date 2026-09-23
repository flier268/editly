---
name: editly
description: >
  Drive the editly CLI and write a valid JSON5 edit spec. Use when the user runs
  /editly, asks to cut or render a video with editly, or needs clips, layers,
  transitions, Ken Burns, audio tracks, or flags such as --out and --fast.
---

# editly CLI

Render with the `editly` binary from `@flier268/editly`. Write a JSON5 spec when the edit needs layers, timing, or audio. Use positional CLI args only for a quick sequence of videos, images, and `title:` cards.

Confirm `ffmpeg` and `ffprobe` are on `PATH` and `editly --help` runs. If the binary is missing: `npm i -g @flier268/editly` (Node.js 20+). Default output is `./editly-out.mp4`. Use `--fast` for the first preview, then render again without it.

```sh
editly my-spec.json5 --out output.mp4
```

A single `.json`, `.json5`, or `.js` argument is the spec (`--json PATH` is the same). CLI flags overwrite the same field on the parsed spec. `--transition-duration` is applied only when the number is not zero. From Node, `import editly from "@flier268/editly"` and `await editly(spec)`. `editly.renderSingleFrame` is also a named export. Call shape, timeline length, and output limits are in `references/spec.md`.

Flags: `--width`, `--height`, `--fps`, `--transition-name`, `--transition-duration`, `--clip-duration`, `--font-path`, `--audio-file-path`, `--loop-audio`, `--keep-source-audio`, `--output-volume`, `--allow-remote-requests`, `--fast`/`-f`, `--verbose`/`-v`, `--out`.

These have no CLI flag and must be written in the spec: `clipsAudioVolume`, `backgroundAudioVolume`, `audioNorm`, `audioTracks`, `globalLayers`, `customOutputArgs`. A Node spec may also set `logTimes`, `keepTmp`, `ffmpegPath`, `ffprobePath`, and `enableFfmpegLog` (`enableFfmpegLog` follows `verbose` when omitted). `ffmpegPath` and `ffprobePath` default to `ffmpeg` and `ffprobe`.

Remote `http`/`https` paths require `allowRemoteRequests: true` or `--allow-remote-requests`. JSON5 may use comments, trailing commas, and unquoted keys. It cannot contain functions. A `.js` path passed to the `editly` binary is still parsed as JSON5. Custom `canvas`, `fabric`, and `fabricImagePostProcessing` effects are a Node script that imports `@flier268/editly`; follow `references/custom-effects.md`. A `gl` layer with `fragmentPath` can stay in JSON5.

Before writing or editing a spec, read the reference that matches the fields you will set. Paths are next to this file:

- `references/spec.md` — top-level fields, `defaults`, `clips`
- `references/layers.md` — layer types and shared visual fields
- `references/audio-and-transitions.md` — transitions, clip audio, `audioTracks`
- `references/custom-effects.md` — custom GLSL, node-canvas, Fabric, and video frame post-processing

Copy field names and defaults from those files. Do not invent flags or layer fields.

If the render prints `Caught error` and exits non-zero, fix the reported field or missing file.
