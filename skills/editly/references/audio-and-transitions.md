# Transitions

Set on `defaults.transition` or on `clips[].transition`. `duration` default `0.5`. `name` default `random`. `audioOutCurve` and `audioInCurve` default `tri` (ffmpeg afade curves: `tri`, `qsin`, `hsin`, `esin`, `log`, `ipar`, `qua`, `cub`, `squ`, `cbr`, `par`, `exp`, `iqsin`, `ihsin`, `dese`, `desi`, `losi`, `nofade`).

`name` is any name from the gl-transitions gallery, or `directional-left`, `directional-right`, `directional-up`, `directional-down`, `random`, `dummy`. Do not invent a name. `directional-*` aliases set easing `easeOutExpo`. Other easings are `linear` and `easeInOutCubic` (`easing` field). `params` is an object of numbers, booleans, or number arrays passed to that gl-transition.

`random` picks a different transition per clip boundary. `dummy` is a cut.

# Audio

`mixVolume` is a number (default `1`) or an ffmpeg volume string such as `"10dB"`.

Clip audio (`video` and `audio` layers) is one mix. `audioTracks` and `audioFilePath` are a separate mix. `clipsAudioVolume` is the volume of the combined clip mix relative to each `audioTracks` entry. Top-level volume fields are listed in `spec.md`.

`audio` layer: `path` required, `cutFrom` default `0`, `cutTo` default the clip duration, `mixVolume`. The cut segment is time-stretched to the clip duration, limited to 0.5×–100×.

`audioTracks[]`: `path` required, `mixVolume` default `1`, `cutFrom` default `0`, `cutTo` optional, `start` default `0` (seconds into the finished video). These keep playing across clips.

`detached-audio` is an `audioTracks` entry placed on a clip. Same fields, but `start` is seconds from the start of that clip. It is not allowed in `globalLayers`.

`audioNorm.enable` runs dynaudnorm. `gaussSize` default `5`, `maxGain` default `30`.

Voice-over plus a music bed that starts at 2 seconds:

```json5
{
  outPath: "./out.mp4",
  keepSourceAudio: true,
  clipsAudioVolume: 1,
  audioTracks: [
    { path: "./voice.wav", start: 0, mixVolume: 1 },
    { path: "./music.mp3", start: 2, mixVolume: 0.2 },
  ],
  clips: [{ layers: [{ type: "video", path: "./clip.mp4", resizeMode: "cover" }] }],
}
```
