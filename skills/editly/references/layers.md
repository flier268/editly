# Layers

Layers in one clip stack in array order. The last layer is on top. Field names below are the only ones to write. `type` must be one of the sections below. Any other name throws `Invalid type`. `defineFrameSource` is not exported from `@flier268/editly`, so a script cannot register a new type.

Every layer accepts `start` and `stop` (seconds from the start of its clip). Omit them to show the layer for the whole clip.

`resizeMode` is `contain` (letterbox), `contain-blur` (default: blurred letterbox), `cover` (crop), or `stretch`.

`position` is one of `top`, `bottom`, `center`, `top-left`, `top-right`, `center-left`, `center-right`, `bottom-left`, `bottom-right`, or `{ x, y, originX, originY }`. `x`/`y` are 0–1 from the top-left of the frame. `originX` is `left`, `center`, or `right` (default `left`). `originY` is `top`, `center`, or `bottom` (default `top`).

Ken Burns, on `image`, `image-overlay`, and `title`: `zoomDirection` is `in`, `out`, `left`, `right`, `up`, `down`, or `null` to disable. `zoomAmount` defaults to `0.1`.

Colors are CSS colors (`#fff`, `#ffffff`, `rgba(0,0,0,0.3)`).

`mixVolume` is defined in `audio-and-transitions.md`.

## video

`path` required. `resizeMode`, `cutFrom` (default `0`), `cutTo` (default end of file), `width` (default `1`), `height` (default `1`), `position`, `mixVolume`. `left`, `top`, `originX`, `originY` still work and are deprecated; write `position` instead.

If the clip has `duration`, the `cutTo - cutFrom` segment is sped up or slowed down to fill that duration. Audio in the file is kept and mixed with other clip audio when `keepSourceAudio` is on. Animated GIFs belong here, not in `image-overlay`.

`fabricImagePostProcessing` mutates this layer's Fabric image before it is added to the clip canvas. The CLI cannot pass that function. Signature and call shape are in `custom-effects.md`.

## image

Full frame. `path` required. `resizeMode`. Ken Burns fields.

## image-overlay

`path` required. `position`, `width`, `height` (each 0–1 of the frame). Ken Burns fields.

## title

`text` required. `textColor` default `#ffffff`. `fontPath`. `fontSize` is pixels; omitted means 10% of the shorter frame side. `position`. Ken Burns fields. `style`: `word-by-word`, `fade-in`, or `letter-by-letter`. `animationDuration` (seconds) applies to `word-by-word` and `letter-by-letter`; omitted uses a duration derived from the text length. `outlineColor`, `outlineWidth` (default `0`), `outlineStyle` (`outline` default, `shadow`, or `glow`).

`fontFamily` is only valid when that family is already registered. If `fontPath` is also set, `fontFamily` is ignored.

## subtitle

Bottom bar. `text` required. `textColor` default `#ffffff`. `backgroundColor` default `rgba(0,0,0,0.3)`. `fontPath`. `fontSize` is pixels; omitted means 1/20 of the shorter frame side. `delay` default `0` and `speed` default `1` scale the fade/slide (progress is eased with easeOutExpo).

## title-background

Expands into a background layer plus a `title`. `text` required. Same text fields as `title` (`textColor`, `fontPath`, `fontSize`). `background` is a `radial-gradient`, `linear-gradient`, or `fill-color` object. Omitted `background` picks one of those three at random.

## news-title

`text` required. `textColor` default `#ffffff`. `backgroundColor` default `#d02a42`. `fontPath`. `position`. `delay` default `0`. `speed` default `1`. Font size is fixed at 5% of the shorter frame side.

## slide-in-text

`text` required. `textColor` default `#ffffff`. `fontPath`. `position`. `fontSize` is a fraction of frame width, default `0.05`. `charSpacing` is a fraction of frame width, default `0.1`. The old field `color` still overrides `textColor` and prints a deprecation warning.

## fill-color and pause

`color`, or a random color when omitted. `pause` is rendered as `fill-color`.

## radial-gradient and linear-gradient

`colors`: exactly two colors. Omitted means two random colors.

## rainbow-colors

No fields. Renders the built-in rainbow shader.

## gl

`fragmentPath` (`.frag`) or inline `fragmentSrc`. `vertexPath` and `vertexSrc` are optional. `speed` defaults to `1`. Shader inputs and a minimal fragment shader are in `custom-effects.md`. This layer is valid in a JSON5 file passed to the CLI.

## canvas and fabric

Not valid in JSON5 or in a file the CLI parses. Run them from a Node script. `func` and `onRender` contracts are in `custom-effects.md`.

## editly-banner

No required fields. Expands to a random `linear-gradient` plus a title `"Made with\nEDITLY\nmifi.no"`. `fontPath` is passed through to that title.
