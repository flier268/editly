# Edit spec

JSON5 object. `clips` is required. CLI `--out` overwrites `outPath`.

```json5
{
  outPath: "./editly-out.mp4", // .mp4, .mkv, .gif, or audio-only .mp3
  width: 640, // optional. resolution rules below
  height: 360, // optional. resolution rules below
  fps: 25, // optional. default: first video fps, else 25
  allowRemoteRequests: false,
  fast: false, // preview: low resolution and fps
  keepSourceAudio: false, // audio inside video layers. mixing rules in audio-and-transitions.md
  clipsAudioVolume: 1, // volume of ALL clip audio vs each audioTracks entry. number or "10dB"
  audioFilePath: "./music.mp3", // one bed for the whole timeline
  backgroundAudioVolume: 1, // volume of audioFilePath only. number or "10dB"
  loopAudio: false, // loop audioFilePath if it is shorter than the video
  outputVolume: 1, // final ffmpeg volume. number or "10dB"
  customOutputArgs: ["-vcodec", "libx264", "-crf", "18"], // replaces the default output args. default codec is h264
  audioNorm: { enable: false, gaussSize: 5, maxGain: 30 }, // see audio-and-transitions.md
  audioTracks: [], // see audio-and-transitions.md
  globalLayers: [], // visual layers drawn above every clip. no audio or detached-audio. fields in layers.md
  defaults: {
    duration: 4, // seconds, used when a clip omits duration and has no video layer
    transition: {
      duration: 0.5,
      name: "random",
      audioOutCurve: "tri",
      audioInCurve: "tri",
    }, // null disables transitions. names in audio-and-transitions.md
    layer: { fontPath: "./font.ttf" }, // copied onto every layer
    layerType: {
      "fill-color": { color: "#ff6666" }, // copied onto every layer of that type
    },
  },
  clips: [
    {
      duration: 4,
      transition: { name: "fade", duration: 0.5 }, // transition at the END of this clip. null disables it
      layers: [{ type: "fill-color", color: "#111111" }], // later layers are on top. types in layers.md
    },
  ],
}
```

Without `fast`, omitted `width`, `height`, and `fps` follow the first input video. With no video and no size, the frame is 640×640. A set `width` with a video and no `height` uses that video's aspect ratio; the calculated side is rounded to an even number. When both `width` and `height` are set, those numbers are used, then raised to at least 2. Every other clip is converted to that frame.

`fast` replaces that size with about 250 pixels on each side (aspect kept, sides even) and sets `fps` to 15. A `.gif` with no `width` uses width 320, and with no `fps` uses 10. `.gif` skips audio. A `.mp3` `outPath` writes audio only (`libmp3lame`, 192k, unless `customOutputArgs` is set) and requires `keepSourceAudio`, `audioTracks`, or `audioFilePath`.

Each clip needs `layers` (array). One object is accepted and treated as a one-element array. A clip with no `duration` uses the first `video` layer's length, otherwise `defaults.duration` (4).

Finished length is not the sum of clip durations. At each boundary the transition is clamped to the minimum of this clip's half, the next clip's half, and the requested transition duration. That clamped duration is subtracted from the outgoing clip. The last clip keeps its full duration.

`keepTmp: true` leaves `editly-tmp-<id>` next to `outPath`. It is removed after a successful render when omitted.

```js
import editly from "@flier268/editly";

await editly({ outPath: "./out.mp4", clips: [{ duration: 2, layers: [{ type: "fill-color" }] }] });

await editly.renderSingleFrame({
  outPath: "./frame.png",
  time: 1.5,
  clips: [{ duration: 4, layers: [{ type: "fill-color", color: "#111111" }] }],
});
```

`renderSingleFrame` writes a PNG to `outPath`. Omitted `width` and `height` are 800 and 600. `time` defaults to `0` and is matched against each clip's full `duration`, ignoring the transition overlap.

Title card then a photo:

```json5
{
  outPath: "./out.mp4",
  width: 1280,
  height: 720,
  fps: 30,
  defaults: { transition: { name: "fade", duration: 0.4 } },
  clips: [
    {
      duration: 3,
      layers: [
        {
          type: "title-background",
          text: "Hello",
          background: { type: "fill-color", color: "#111111" },
        },
      ],
    },
    {
      duration: 4,
      layers: [
        { type: "image", path: "./photo.jpg", zoomDirection: "in", zoomAmount: 0.1 },
        { type: "subtitle", text: "Caption" },
      ],
    },
  ],
}
```
